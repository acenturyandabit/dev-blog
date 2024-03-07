# C++: Overoptimizing the builder pattern with template metaprogramming

In C++, the builder pattern can help you construct objects that cannot be easily
default-constructed and have lots of arguments with defaults during testing. We
can use type traits and tuples to reduce repetition in the builder pattern...
but just because we can, doesn't mean we should.

## Builder patterns 101: Soup editon

The language agnostic builder pattern has been covered before
[quite a](https://refactoring.guru/design-patterns/builder/cpp/example)
[few times](https://www.geeksforgeeks.org/builder-pattern-c-design-patterns/),
[including in C++](https://medium.com/@antwang/builder-pattern-in-c-the-right-way-e943abbe0d2d);
and even though we're going beyond that today, let's start by looking at a toy
example: we're going to make some soup.

```c++
#include <string>
#include <iostream>

class Soup{
public:
    // Soup cannot be default-constructed, otherwise it would taste bad.
    Soup(int waterVolume, std::string base, float mushroomCount) :
      waterVolume_(waterVolume), base_(base), mushroomCount_(mushroomCount) {}

    void enjoy(){
        std::cout << "Wow, I love " << waterVolume_
            << " litres of " << base_ 
            << " soup with " << mushroomCount_
            << " mushrooms!" << std::endl;
    }

    // We want these to be our default values
    static constexpr int DEFAULT_WATER_VOLUME = 10;
    static constexpr std::string_view DEFAULT_BASE = "tomato";
    static constexpr float DEFAULT_MUSHROOM_COUNT = 100;

private:
    int waterVolume_;
    std::string base_;
    float mushroomCount_;
};

Soup soup{10, "miso", 100}; // ok
// Soup soup{"miso"}; // Not ok, we need to provide all three arguments
```

So here is some soup. The soup has three arguments, and you must specify all
three arguments in order to construct the soup. Even though we have some default
values, we can't pick and chose between them easily: we could try and specify
them in order in the ctor as default valued arguments, but then we can only omit
arguments starting from the right; alternatively, we could provide a default
constructor and setter methods, but we might want some of our members to be
read-only, in which case setters would not be allowed.

Fortunately, we can fix these issues with the builder pattern!

```c++
struct SoupBuilder {
    int waterVolume {Soup::DEFAULT_WATER_VOLUME};
    std::string base {Soup::DEFAULT_BASE};
    float mushroomCount {Soup::DEFAULT_MUSHROOM_COUNT};

    Soup build(){
        return Soup(waterVolume, base, mushroomCount);
    }
};

Soup soup = SoupBuilder{}.build(); // Default soup

SoupBuilder misoSoupBuilder{};
misoSoupBuilder.base = "miso";
misoSoupBuilder.mushroomCount = 1000;
Soup misoSoup = misoSoupBuilder.build(); // partially default soup

int main() {
  misoSoup.enjoy();
  return 0;
}
```

<details>

<summary>Wrapped modified-default builders</summary>

Suppose in one part of your code you often make miso soup; and in another part
you often make cream-of-chicken soup. You can try and semi-specialize your builders:

```c++
// WRONG! DO NOT COPY
SoupBuilder misoSoupBuilder = SoupBuilder{}.setBase("miso");
SoupBuilder creamSoupBuilder = SoupBuilder{}.setBase("cream-of-chicken");

misoSoupBuilder.mushroomCount = 2000;
Soup soup1 = misoSoupBuilder.build();

misoSoupBuilder.water = 1000;
Soup soup2 = misoSoupBuilder.build();
```

The problem here is that soup2 will have 2000 mushrooms, even though we may have
intended to have the original default of 100, because we've reused an instance of
SoupBuilder. Instead, we need to make a SoupBuilder factory:

```c++
SoupBuilder misoSoupBuilder() { return SoupBuilder{}.setBase("miso"); };
```

and everything works perfectly fine.

</details>

<br>

Great! But we had to mention `misoSoupBuilder` in our code three times. A cool
thing about the builder pattern is that you can create setter methods that
return the builder itself, which means you can chain your builder setter methods:

```c++
struct SoupBuilder{
    SoupBuilder &setWaterVolume(int waterVolume) {
        waterVolume_ = waterVolume;
        return *this;
    }

    SoupBuilder &setBase(std::string base) {
        waterVolume_ = waterVolume;
        return *this;
    }

    SoupBuilder &setMushrooms(float mushroomCount) {
        mushroomCount_ = mushroomCount;
        return *this;
    }

    Soup build(){
        return Soup(waterVolume, base, mushroomCount);
    }

private:
    int waterVolume_{Soup::DEFAULT_WATER_VOLUME};
    std::string base_{Soup::DEFAULT_BASE};
    float mushroomCount_{Soup::DEFAULT_MUSHROOM_COUNT};
};

Soup soup = SoupBuilder{}.build(); // Default soup

SoupBuilder{};
Soup misoSoup = SoupBuilder{}
    .setBase("miso")
    .setMushrooms(1000)
    .build();
```

Yay! We don't have to repeat the name of the builder anymore! However, we pay a
price: each time we add a property we need to define a new setter method: three
whole lines! Wouldn't it be nice if we didn't have to do that?

## Cooking with C++17 tuples and type traits

> A good senior engineer seeing this in a code review will *probably* tell you off
> if you actually did this, because it doesn't conform to what people expect to see
> when they open up a class definition. There's a more developer-friendly way to
> avoid repetition write builder classes which doesn't involve a ridiculous amount
> of metaprogramming, which I will write about this in another blog in future.

That being said, let's try to boil our code down even further with some templates.

We can create a `set()` method that will take an argument, check its type, then
based on that type, determine the correct member to set. Here's how it works:

```c++
#include <type_traits>

// static_assert false requires a template type otherwise the code will 
// always fail to compile
template<typename T>
struct always_false : std::false_type
{ }; 

struct SoupBuilder{

    template <typename ArgT>
    SoupBuilder &set(ArgT valueToSet) {
        if constexpr (std::is_same_v<ArgT, int>){
            waterVolume_ = valueToSet;
        } else if constexpr (std::is_same_v<ArgT, std::string>){
            base_ = valueToSet;
        } else if constexpr (std::is_same_v<ArgT, float>){
            mushroomCount_ = valueToSet;
        } else {
            static_assert(always_false<ValueToSetT>::value,
                      "Could not match parameter in SoupBuilder!");
        }
        return *this;
    }

    Soup build(){
        return Soup(waterVolume_, base_, mushroomCount_);
    }

private:
    int waterVolume_{Soup::DEFAULT_WATER_VOLUME};
    std::string base_{Soup::DEFAULT_BASE};
    float mushroomCount_{Soup::DEFAULT_MUSHROOM_COUNT};
};

Soup misoSoup = SoupBuilder{}
    .set(std::string{"miso"}) // base
    .set(1000.0) // mushrooms
    .set(static_cast<double>(300)) // gives a compile-time error
    .build();
```

Nice. Now we only need two lines per new property. But we have a problem: What if
we have two int-members? Or even worse, what if we try to `set(100)` mushrooms,
but end up setting water instead?

For this, we can use strong types
(Check out [FluentCPP - an awesome C++ blog!](https://www.fluentcpp.com/2016/12/08/strong-types-for-strong-interfaces/))
to define exactly which property we're setting, and as a bonus, get stronger interfaces.

<details>
<summary>A caveat about the compile time error</summary>

The compile time error produced by using a wrong type in our builder is
`Static assertion failed due to requirement 'always_false<double>::value'`
`Could not match parameter in SoupBuilder!`; but it appears *at the static_assert*,
not at the call site of the bad set() function. Even though we can chase the error's
definition trace down to the call site, and we get the 'double' type in our assert
message, this is obviously not ideal.
</details>

```c++

template <typename BaseT, typename TagT>
struct StrongType{
    // In real code we would use a better StrongType implementation
    BaseT value;
};

// In real code we might use these types to define our Soup, or even globally
// across the codebase.
using Water = StrongType<int, struct WaterTag>;
using Mushrooms = StrongType<float, struct MushroomTag>;

struct SoupBuilder{
    template <typename ArgT>
    SoupBuilder &set(ArgT valueToSet) {
        if constexpr (std::is_same_v<ArgT, Water>){
            waterVolume_ = valueToSet;
        } // ... etc 
        return *this;
    }

    Soup build(){
        return Soup(waterVolume_.value, base_, mushroomCount_.value);
    }

private:
    // We don't _have_ to tag all our types, if they're already unique in the 
    // struct.
    Water waterVolume_{Soup::DEFAULT_WATER_VOLUME};
    std::string base_{Soup::DEFAULT_BASE};
    Mushrooms mushroomCount_{Soup::DEFAULT_MUSHROOM_COUNT};
};

Soup misoSoup = SoupBuilder{}
    .set("miso")
    .set(Mushrooms{1000})
    .build();

```

Now we see that for the types we've decided to make into strong types, the name
of the parameter appears at the call site, which means we can clearly see what
is going on and avoid implicit type conversions. Hooray!

While I'm a pretty big fan of strong types, this isn't a strong type post -
it's a repetition minimisation post; and we are still repeating a bunch of
`if constexpr` for each parameter. Let's see if tuples will help us here...

```c++
#include <tuple>

// A functor class that takes a tuple of references, then recursively tries to
// set a value on the tuple for a matching type. 
class TrySetInTuple{
  template <typename TupleT, typename ValueToSetT, int IDX = 0>
  void recursiveTrySet(TupleT &tuple, ValueToSetT &value) {
    auto &prop = std::get<IDX>(tuple);
    if constexpr (std::is_same_v<std::decay_t<decltype(prop)>, ValueToSetT>) {
      prop = value;
    } else if constexpr (IDX < std::tuple_size_v<TupleT> - 1) {
      recursiveTrySet<TupleT, ValueToSetT, IDX + 1>(tuple, value);
    } else {
        static_assert(always_false<ValueToSetT>::value,
                      "Could not match parameter in tuple!");
    }
  };

public: 
  template <typename TupleT, typename ValueToSetT>
  void operator()(TupleT &tuple, ValueToSetT &value){
    recursiveTrySet(tuple, value);
  }
};

struct SoupBuilder{
  template <typename ArgT> SoupBuilder &set(ArgT valueToSet) {
    // tie() creates a tuple of references.
    auto props = std::tie(
        waterVolume_,
        base_,
        mushroomCount_
    );
    TrySetInTuple{}(props, valueToSet);
    return *this;
  }

  Soup build(){
    return Soup(waterVolume.value, base, mushroomCount.value);
  }

private:
    Water waterVolume_{Soup::DEFAULT_WATER_VOLUME};
    std::string base_{Soup::DEFAULT_BASE};
    Mushrooms mushroomCount_{Soup::DEFAULT_MUSHROOM_COUNT};
};
```

Fantastic! Now we can just add our parameters to the `std::tie()` and we
automatically get a setter!

<details>
<summary>The compile time error is getting more abstract...</summary>

Now that the static assert is not in our concrete SoupBuilder, if we try and define
multiple setters using our TrySetInTuple, we may get even more confused.
</details>

<br>

But can we do *better*? What if we created a meta builder class that allows us to
specify all the defaults in one go in the constructor, so we don't even have to write
the member list?

First, let's quickly change our Soup class so that we define the ctor in terms of
the strong types (which _is_ good practice); and also add a DEFAULTS tuple for
what we want the default values of our ctor to be:

```c++
class Soup {
public:
  Soup(Water waterVolume, std::string_view base, Mushrooms mushroomCount)
      : waterVolume_(waterVolume.value), base_(base),
        mushroomCount_(mushroomCount.value) {}
  ...

  // We want these to be our default values
  static constexpr auto DEFAULTS =
      std::make_tuple(Water{10}, std::string_view{"miso"}, Mushrooms{100});
private:
  ...
};
```

Now, for the grand finale: Here's a MetaBuilder class, where we can pass the Soup
and get a builder for the soup, without having to write any extra builder code.

```c++
template <typename BaseT> class MetaBuilder {
public:
  MetaBuilder() : props_(BaseT::DEFAULTS) {}

  template <typename ArgT> MetaBuilder &set(ArgT valueToSet) {
    // props is a tuple of values; we turn it into a tuple of references
    auto referenceProps = MakeReferenceTuple{}(props_);
    TrySetInTuple{}(referenceProps, valueToSet);
    return static_cast<MetaBuilder &>(*this);
  }

  auto props() { return props_; }

  BaseT build() { return std::make_from_tuple<BaseT>(props()); }

private:
  // Value type to store the values of the props.
  // Use std::decay_t to make it non-const.
  std::decay_t<decltype(BaseT::DEFAULTS)> props_;

  template <typename T> struct always_false : std::false_type {};

  class TrySetInTuple {
    template <typename TupleT, typename ValueToSetT, int IDX = 0>
    void recursiveTrySet(TupleT &tuple, ValueToSetT &value) {
      auto &prop = std::get<IDX>(tuple);
      if constexpr (std::is_same_v<std::decay_t<decltype(prop)>, ValueToSetT>) {
        prop = value;
      } else if constexpr (IDX < std::tuple_size_v<TupleT> - 1) {
        recursiveTrySet<TupleT, ValueToSetT, IDX + 1>(tuple, value);
      } else {
        static_assert(always_false<ValueToSetT>::value,
                      "Could not match parameter in builder!");
      }
    };

  public:
    template <typename TupleT, typename ValueToSetT>
    void operator()(TupleT &tuple, ValueToSetT &value) {
      recursiveTrySet(tuple, value);
    }
  };

  class MakeReferenceTuple {
    template <typename TupleT, int N_EXPANDED_PARAMS = 0,
              typename... ExpandedParams>
    auto recursiveMakeTuple(TupleT &tuple, ExpandedParams &...references) {
      if constexpr (N_EXPANDED_PARAMS == std::tuple_size_v<TupleT>) {
        return std::tie(references...);
      } else if constexpr (N_EXPANDED_PARAMS < std::tuple_size_v<TupleT>) {
        return recursiveMakeTuple<
            TupleT, N_EXPANDED_PARAMS + 1,
            std::tuple_element_t<N_EXPANDED_PARAMS, TupleT>>(
            tuple, std::get<N_EXPANDED_PARAMS>(tuple), references...);
      } else {
        static_assert(always_false<TupleT>::value); // Invalid index (should be
                                                    // impossible)
      }
    };

  public:
    template <typename TupleT> auto operator()(TupleT &tuple) {
      return recursiveMakeTuple(tuple);
    }
  };
};

// A super clean definition of the builder class.
using SoupBuilder = MetaBuilder<Soup>;
```

Finally, we've used 67 lines of (re)cursed metaprogramming to avoid writing 12
lines of SoupBuilder code!

> If you've read through this blog and you effortlessly understand everything that
> is going on here, then congratulations - you must have had a decent amount of
> experience with `std::tuple`, template classes, references, and C++
> metaprogramming :)
