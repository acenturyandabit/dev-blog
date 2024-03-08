# Benchmarking key-value pagination algorithms in javascript

I am making a to-do list app which stores entries in a Key-Value store. The KVstore
has so many entries that sending the entire KVstore to the frontend takes longer
than the connection timeout on my phone, but I want to send the entire KVstore across.

So, I need to split the KVstore into pages. The KVstore is a javascript
`Record<string, object>` with many different keys. How can we split this in a
performant way?

Let's use <https://jsperf.app/> to try two algorithms.

<details>
    <summary>Javascript profiling tools</summary>

I also tried <https://jsbench.me/> but this site gave me very high variance
between runs. I'm not sure that's it's fault though, because I'm fairly sure these
sites run within your browser, which means if you're multitasking (say, writing
a blog entry) while you're running the code, that will skew the results.

</details>

## The setup

We create a KVstore with 100,000 items, and we aim to split it into pages of
10,000 items.

```js
const large_kvstore = Array(100000).reduce((dict,_,idx)=>{
    dict[Math.random()*Math.random()]={value: idx}; return dict;
},{});
const page_size = 10000;
```

## First approach: Using Javascript `Object`/`Array` methods

```js
const entries = Object.entries(large_kvstore)
let page = 0;
const sub_entries = [];
while (page * page_size < entries.length) {
 sub_entries.push(entries
        .slice(page*page_size, (page+1)*page_size)
        .reduce((dict,[key,value])=>{dict[key]=value; return dict},{}));
 page++;
}
```

## Second approach: Using `for-in` loop

```js
const sub_entries = [{}];
let page = 0;
let counter = 0;
for (let key in large_kvstore){
 sub_entries[page][key] = large_kvstore[key];
 counter++;
 if (counter == page_size){
  counter = 0;
  sub_entries.push({});
  page++;
 }
}
```

## Results and discussion

Both approaches gave 148-149 million ops per second. I guess the chrome JS engine
is pretty well optimized for these kinds of loops...
