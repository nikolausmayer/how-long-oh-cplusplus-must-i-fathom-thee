# Oh, C++
Why must you be like that.

```
#include <numeric>
#include <iostream>
#include <vector>

template <typename... Args>
auto sum(const Args... args) -> decltype((args + ... + 0)) {
  const std::vector<decltype((args + ... + 0))> values = {args...};
  return std::accumulate(values.begin(), values.end(), decltype((args + ... + 0)){0});
}

int main() {
  std::cout << sum(1, 2, 3.5) << std::endl;
}
```

### +++EXISTENTIAL CRISIS AVERTED+++
With C++17, this works *as it should*:
```
#include <iostream>

template <typename... Args>
auto sum(const Args... args) {
  return (args + ... + 0);
}

int main() {
  std::cout << sum(1, 2, 3.5) << std::endl;
}
```


# The MVP

```
struct Index {
  int i = 4;
};
struct Something {
  Something(Index i) : m_index(i) { }
  void incrementIndex() { m_index.i++; }
  Index m_index;
};

int main() {
  Something my_something(Index());
  my_something.incrementIndex();
}
```
This makes GCC one angry boi:
```
In function ‘int main()’:
error: request for member ‘incrementIndex’ in ‘my_something’, which is of non-class type ‘Something(Index (*)())’
  my_something.incrementIndex();
               ^~~~~~~~~~~~~~
```
### Coder goes 👁️👄wat👁️

Meet ~the most valuable player~ **Most Vexing Parse**. The final boss of syntax bullshit, or at least I hecking hope it is because *goshdarnit C++ can you please just NOT?* The MVP will make you pine for the sweet release of multi-page Boost template GCC compile error torture.

So what in the sweet Bejeebus is happening? Well, `Something my_something(Index())` can unfortunately be read in two different ways:
1. *"Construct a Something called »my_something« out of a freshly created Index".* This is what the coder wants because the coder is a sane person who isn't mentally stuck in 1970. Nuh-uh. No such luck.
2. *"Declare a function called »my_something« which takes [a function that produces an Index], and produces a Something".* Because `Index()` can be a declaration of a parameter-less `Index`-returning function; think like how in `int main()`, the `main` is just the *name* for an `int()` and in a declaration you don't need the names of parameters. No sane person wants this. But **t h e s t a n d a r d** wants this because the standard is a braindead moron who likes to eat glue, and because the standard wants this, the compiler friggin *does it YES SIR MISTER PRESIDENT SIR*.

So because 2. is what happens, OF COURSE the compiler bottoms out in the next line `my_something.incrementIndex();`! Who has ever heard of a function having a callable method? Nobody has, that's who. F*CK

### +++A Solution or whatever+++
With the *uniform initialization* syntax introduced with C++11 (because C++ apparently didn't have **enough braces** or whatever), that deeply offensive line can instead be written as
```
Something my_something(Index{});
```
or
```
Something my_something{Index()};
```
or yes daddy more braces
```
Something my_something{Index{}};
```
which finally makes the compiler realize that for heaven's sake buddy I guess the square peg just don't go in the round hole, and all is fine and it compiles and there was peace throughout the land.
