# acenturyandabit's dev blog

Welcome to my dev blog! I'm acenturyandabit, a professional software developer with experience in C++, Python, Typescript, and making wonderful things come to life.

I write about my work and my opinions on software development; and I read and anthologize others' works. Come read my stuff! If you'd like to stay up to date, please send an email to [acenturyandabit.gmail.com](mailto:acenturyandabit.gmail.com) with your desired subscription frequency (every post; monthly; annually) and I will mail you when a new post comes out.

## Index

<details>
<summary>Development day-to-day</summary>

- [Multithreaded Development: Increase your productivity when you have background jobs](./multithreaded-development.md)
- [Assistance Repositories](./assistance-repositories.md)

</details>
<details>
<summary>Git tricks series</summary>

- [Git tricks 1: .gitignore a whole folder with no side effects](./git-tricks-1-gitignore.md)
- [Git tricks 2: Using git worktree to speed up multi-issue development](./git-tricks-2-git-worktree.md)

</details>
<details>
<summary>Programming languages</summary>

<div style="padding-left: 10px">
<details>
<summary>C++</summary>

- FluentC++: [Strong types for strong interfaces](https://www.fluentcpp.com/2016/12/08/strong-types-for-strong-interfaces/) (and really, the rest of the blog)
- [C++: Overoptimizing the builder pattern with template metaprogramming](./cpp-builder-pattern.md)
- Mircea Baja: [Using std::tie to quickly write comparison operators for user-defined data structures](https://bajamircea.github.io/coding/cpp/2017/03/10/std-tie.html)

</details>
<details>
<summary>Javascript</summary>

- [Javascript: Benchmarking KVstore pagination algorithms](./js-kv-pagination-benchmark.md)

</details>
<details>
<summary>Python</summary>

- [Python Quirks](./python-quirks.md)

</details>
</div></details>
<details>
<summary>Algorithms</summary>

- Marc Brooker: [Adding randomness to load balancing makes it better](https://brooker.co.za/blog/2012/01/17/two-random.html)

</details>

<details>
<summary>Thinking like a C-suite decision maker</summary>

- Marc Brooker: [Questions to ask when starting out projects](https://brooker.co.za/blog/2024/03/04/mousetrap.html)

</details>

<details>
<summary>Collections</summary>

- [Let's Build AI](https://letsbuild.ai/)

</details>


<br>
I've got a few articles in the works, but I figured this project needs a README, so my first post is going to be about just that :)

## Kudos

Thanks to our staff eng at [Company 1] for sharing their knowledge about C++ during code reviews.

# Give a dev a README 🐟, or teach a dev to README 🎣?

A README is the landing page of a project. Making a great landing page is typically in the domain of marketing teams, but if you are leading the development of a project (whether it's a large project used by thousands, or something small to help you land your first job), then you may be the best developer-facing marketing team your project will get.

> A good README is a landing page that concisely gives a value proposition and a call to action.

A good landing page has two main functions: presenting the value proposition of the product, and providing a call to action that visitors can jump on. A good landing page also needs to deal with the realities of its readership: it is competing for a limited amount of attention, so it should be clear and concise.

READMEs can have a lot of different readers: developers, business users, technical users, and investors. If you are catering to multiple kinds of readers, try to have a table of contents, like [ZOD's readme](https://github.com/colinhacks/zod?tab=readme-ov-file).

## READMEs (like other software) require maintenance

In my experience as a programmer, I've often been pointed at repositories with an inviting README.md file, only to find that the README either:

- Is just a blank file, or contains a single line with a heading of the name of the repository
- Hasn't been updated since the project took a major turn 3 years ago, and now describes something completely different
- Is a 30KB file which is the dumping ground for _all_ of the project's documentation, including developer docs, user guides, design docs and issue docs;
- Is a README from a boilerplate with a whole lot of information about the boilerplate... but nothing about the project itself

This happens because even though a README is a landing page, a README is also code; and like any other code, READMEs naturally decay in value as the scope increases or changes. Documentation is important, but a README isn't the only form of documentation. (I shall to write about this later!)

Since effort is a limited resource, one ought to allocate effort to the README proportional to the amount of utility the README provides. That is to say, if there are no new developers encountering the project, the README may not need to be updated in a long time.

### Unit test your READMEs

So if READMEs are software, can we unit test a README? In fact we can, and we can put it in our CI / git hooks; and I would encourage it! If you have instructions in your README, consider writing them as bash scripts with comments, and then running your README as part of your test procedure. Remember there are tools like docker that allow you to mock an entire fresh machine in CI.

## Making a boilerplate? Keep your READMEs extra lean

Boilerplates should be concise and fit for purpose; and arming folks with the skills to write a README is much more effective than providing a README template. What a README boilerplate _shouldn't_ do is to extol the virtues of the boilerplate itself more than a single link.

One of github's most prolific boilerplates is Create React App. Its README is 3.28KB over 70 lines; the total code in the boilerplate excluding the README is around 4.5 KB; making the README almost half the size of the project. (Prior to 2018 it was even larger - a whopping 126KB). There are 247 public repositories with one of the title lines of CRA's README: "This project was bootstrapped with Create React App"; of which 13 of them contain the README mostly unmodified.  

CRA isn't the only culprit here - I have seen internal company boilerplates which also provide incredibly verbose READMEs. And I get the urge to extol the virtues of the boilerplate and give usage instructions - a lot of work goes into making a good boilerplate - but the moment the boilerplate has been instantiated, the project has a life of its own. It doesn't need a fat README to get it started - that belongs in the boilerplate cutter's README only.

Really the CRA README should be blank; with a single title and "This project was bootstrapped with CRA [link to docs]", and "Use npm run to start". When devs see the README file, they should think: "Hey, this boilerplate has given me some free real estate, time to write my own README!" rather than "I better not touch that README, it looks very official, maybe there's something I need there". This is the best a boilerplate README can do; especially since the boilerplate can't know in advance how it is going to be used.

----
That's all for this time folks! If you enjoyed this, please star this repository and consider subscribing by sending an email to [acenturyandabit.gmail.com](mailto:acenturyandabit.gmail.com) with your desired subscription frequency. See you next time!
