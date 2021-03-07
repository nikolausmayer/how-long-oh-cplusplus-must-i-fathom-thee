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


# Lambdas, or what ever happened to Perl being the least readable thing ever
*Sigh.* Lambdas.
```
int main() { [](){}(); }
```
Some utter degenerate went full "bruh I'm a *hard simp* for *punctuation marks* OOH YES HARDER BRACKETS-せんぱい UWU"? 

How else do you explain this? **What were they thinking?!**

And no it doesn't stop there, why would it. There's `[=](){}` and `[](){}` and `[]{}()` (hahah of *course* those are not the same) and naturally then there's good ol' `[&]()->void{}()` dog bless its soul and oh right we can't ever forget *this* motherlover right here because *yes OF COURSE we needed lambdas AND TEMPLATES EVERYTHING NEEDS TEMPLATES IS THIS CPLUSPLUS OR THE WEENY HUT*
```
[]<class T>(T frick){}
```
and ohoho now don't you worry precious darling cause daddy got you a REAL present 
```
auto f = []<typename ...Ts>(Ts&& ...ts) {
  return foo(std::forward<Ts>(ts)...);
};
```
WHY YES I ALWAYS WANTED VARIADIC ANONYMOUS FUNCTIONS but good lord I never aw heck no what a terrible day to have eyes


Ah pardon me sire forgive this unworthy worm's transgression, for I did indeed forget about the unholy darkness that is RECURSIVE LAMBDAS because oh Satan take me now
```
#include <functional>

int main() {
  std::function<int(int)> fuctorial;
  fuctorial = [&fuctorial](int x) -> int {
    return (x <= 1 ? 1 : x * fuctorial(x-1));
  };
  return fuctorial(10);
}
```
Yes. Yes, we must tell the function about itself so it can call itself because of course the function would not know about itself how could it it's not like every other thing in C++ knows about itself without needing to be explicitly told about itself ooh lordy why do it be like that

Of course bless your innocent heart if you ask your self "why declare the `std::function` before defining it?". By all means, try writing it as
```
std::function<int(int)> fuctorial = [&fuctorial] ...
```
or (yes I know these are not the same)
```
auto fuctorial = [&fuctorial] ...
```
try it! But maybe keep a punching bag and/or frustration-proof SO nearby.

It was a mistake to ever go beyond Church literals.


# Designated Array Initializers ("DAI")
You ever wanted to create an array of zeroes? Wait, let me reformulate that—you ever wanted to create an array of 27 zeroes in the **least sensible syntax imaginable?**
```
int array[27] = {};
```
Hahah, yes, of course I want to assign the *empty scope* or a *Python dict* or the f\*cking *frog mouth operator* to my array of 27 integer numbers, thank you C++ that's *EXACTLY* what I wanted. UGHH.

But wait, there's more! Remember how numbers are kinda the same? Like, a zero and a one are kinda both numbers? Right? So of course they work the same way? Right?? Well apparently C++ DIDN'T GET THAT MEMO BECAUSE FFFFFUUUUUUU
```
int array[27] = {1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1
```
UUUUUUUUUUUU
```
1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1};
```
UUUUUGGIN HECK WHY! WHY IS IT DIFFERENT! Why is it so *fundamentally not the same* in *every way* that numbers are *supposed to be fundamentally the same?!*

Ahah, but this isn't even the worst we can do. No, *this* is:
```
int array[27] = {[0] = 1, [1] = 1, [2] = 1, [3] = 1, ...
```
And no, specifying the elements out-of-order does *not* work: GCC throws a temper tantrum because apparently `{[0] = 1}` is "trivial" while `{[26] = 1}` is somehow a "non-trivial designated initializer" (because who knows, maybe the compiler doesn't know how to *sort things*? When it usually just skirts any sort of responsibility and just evaluates things in whatever order it pleases? When it's typically fancy enough to just magically *infer* cognitive-dissonance-level complexities of indirection into all sorts of *guaranteed elisions*? But suddenly it's *to good* so sort a bunch of *numbers*? ***Sigh***). 

Well, you might ask, isn't that completely pointless then? Yes. Yes it is.

—

... turns out, in good ol' **C**, *that thing works*
```
int array[] = {[26] = 1, [4] = 1};
```
and *so does this*:
```
int array[] = {[0 ... 26] = 1};
```
However, it is somehow **NOT** standard C++, AFAICT. GCC 10.2 does not like it, in any case. Clang, may the force be with it, **works**.

—

Oh right, and if you ever want to have an array of GODDAMN RANDOM BULLSH—
```
int array[27];
```
yeah have fun debugging *THAT* in prod, suckers.

(They're not called *DAI*, by the way. I am not responsible for damages resulting from unlawful use of this term.)

—

Just for completeness' sake: do not do this. Use a *nice* way, one that won't make your reviewer want to choke you. Try to keep the WTFs/min to *ever so slightly above minimum*.
```
#include <algorithm>

int array[27];
std::fill(std::begin(array), std::end(array), 1);
```
or in prehistory C++ (yes I like to treat my `sizeof` like a function, *as it deserves*. Barbarian.)
```
int array[27];
std::fill(array, array + sizeof(array)/sizeof(array[0]), 1);
```
or, if you can use at least C++17,
```
for (auto i = 0; i < std::size(array); i++)
  array[i] = 1;
```
or at least this ugly sumbish
```
for (auto i = 0; i < sizeof(array)/sizeof(array[0]); i++)
  array[i] = 1;
```
ooooor if you *really* want to make your designated rubber ducky consider a career change and/or violent homicide
```
#include <cwchar>
std::wmemset((wchar_t*)array, 1, 27);
```
