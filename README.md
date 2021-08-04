# Oh, C++
Why must you be like that.

---

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

Meet the ~most valuable player~ ~minimum viable product~ **Most Vexing Parse**. The final boss of syntax bullshit, or at least I hecking hope it is because *goshdarnit C++ can you please just NOT?* The MVP will make you pine for the sweet release of multi-page Boost template GCC compile error torture. A parse so vexing that two out of the three words its name consists of require looking them up in a dictionary. It's **recursively stupefying**.

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


# Fantastic Types and How to Know Them

## Eowyn... my body is mangled

Ok. 

So one of *the* things about C++ is that it is *typed*. As in, *statically typed* (an `int` is born an `int` and will die an `int`), and generally considered pretty *strongly typed* as well. You got an `int`, that's an `int` alright and no two ways about it. Sure you can *cast* that `int` into whatever you want, but if you wanna go stupid with that, you gotta be *explicitly stupid*. The Compiler® won't just let you assign a `std::string` to an `int` all willy-nilly.

So, in light of all that... why does C++ have to be such an **asshat** about its types?

Consider this function definition:
```
int a_function(int a_param, float another_param) {
  return 0;
}
```
When we compile this (and only this snippet) into an object file using `g++ -c code.cpp` and check the result's contents using `nm code.o`, we of course get a very clear and readable representation of our very simple function's signature:
```
0000000000000000 T _Z10a_functionif
```
...

ahh, f\*ck.

No, *of course we don't*. This is C++. Are you not entertained?

—

**Name mangling** is a big thing in C++, and just like wearing pants, nobody explicitly says you have to do it, but everybody does it anyway. Name mangling means that you take something understandable and turn it into a *horrible mess* so that the *code-illiterate plebs* never be tempted to encroach unto the sunlit holy mesa that is The Land Of **Us The Coders**. (In a transparently futile attempt to hide their crimes behind pretty make-up, some people say "name decoration".)

Because The Compiler® giveth the gifts of namespaces and overloaded functions, The Compiler® taketh away the ability to *just look at a thing and see what it is*. Instead, we must chant an arcane incantation:
```
$ nm --demangle code.o
0000000000000000 T a_function(int, float)
```
(You know it's arcane because it has *two letters* and thus it was made back when men of all shapes and genders had mighty beards and mighty opinions. ed, vi, nm, bc.)

A few things worthy of note:
- Mangling is not standardized, and consequently *everyone does it differently and nothing works*. Compile with MSVC and link to GCC? Hahah, I think not. Just because you cannot *see* the walls all the time does not mean that your garden is without bounds.
- The *return* type is not part of the function name. This does not mean that The Compiler® does not *know* the return type (how *dare* you question its methods), GCC just doesn't use the function name to keep its tab on it. Other compilers (with lowercase "c" you heathen) do it differently. (Tidbit to impress your next interviewer: does GCC do it differently with *templates*?)


## Actually *Getting* to a Type

Because we as humans just *had* to go ahead and breathe lightning into sand, we today have such problems as "my microwave keeps crashing my WiFi" and "my Twitter-trained AI is an asshole" and of course "help please how do i print type in C++".

Well, it's GCC. It knows its own types. I mean—*how could it not*?

```
#include <typeinfo>
#include <iostream>

int a_function(int a_param, float another_param) {
  return 0;
}
```
Yes.
```
int main() {
  std::cout << typeid(a_function).name() << std::endl;
}
```
Yes. YES!
```
FiifE
```
nnNNOWHATTHEFUUUU̷̧̬͍̭̫̕U̴͍̗͒Ư̶͉̈́̈́Ù̸̧̯̱̠̞̿̀Ú̴̢͖͍̝̜̇U̵̧̱̜͌͂̿?!!!

—

Ok. Ok calm down.

—

Breathe in. Breathe out. Breathe in. Breathe oooouuuuuuu~
```
#include <cxxabi.h>

...

const char* demangle(const char* mangled_name) {
  int err;
  const char* res = abi::__cxa_demangle(mangled_name, 0, 0, &err);
  return res;
}

template <typename T>
std::string printable_type(T&& t) {
  return demangle(typeid(t).name());
}

int main() {
  std::cout << printable_type(a_function) << std::endl;
}
```
The great gate of charity is wide open,
```
int (int, float)
```
with no obstacles before it.

## Lambdas, again, or When The Compiler Says "SMOKE YOUUU"

So lambdas are multidimensionally awesome. On the offensively ungrokable plane of functoresk nightmares on which anonymous function objects coexist together with function pointers and `std::function` polymorphic function wrappers, lambdas are proof that you *can* indeed put lipstick on a pig.

A lambda is also... *kind of*... just [a `struct` by any other name](https://cppinsights.io/lnk?code=I2luY2x1ZGUgPGNzdGRpbz4KI2luY2x1ZGUgPGlvc3RyZWFtPgojaW5jbHVkZSA8dHlwZWluZm8+CgppbnQgbWFpbigpCnsKICAgIGludCBqID0gMDsKICAgIGF1dG8gbCA9IFsmXShpbnQgaSkgewogICAgICByZXR1cm4gaStqOwogICAgfTsKfQ==&insightsOptions=cpp2a&std=cpp2a&rev=1.0). Except... **sigh**. Except this is C++, so we absolutely *positively* can not ever have nice things.
```
main.cpp
---
int main() {
  int j = 0;
  auto l = [&](int i) {
    return i+j;
  };
  std::cout << printable_type(l) << std::endl;
}
---
$ g++-10 -std=c++20 main.cpp
main::{lambda(int)#1}
```
Yeah. That's... not a real type. Like, it *is*, obviously, but holy cow would the compiler ever slap you around a bit with a large trout should you ever feel so cheeky as to endavour *actually writing such a type yourself*. You *can't*.  (See above for how to `printable_type`.) It's an *unknowable type* and C++ has come full circlejerk: a type so strict, not even the coder is allowed to know it. It is for your own safety, citizen. Now pick up that trashcan.

So, `struct`s with unknowable types. There are two entirely horrifying consequences arising from this.

### *You can inherit from lambdas*.
Yes, from an anonymous function object's class. But not really, or kinda yes really but only **some** lambdas. Let's... just look at one example:
```
auto k = [](int i) {
  return i;
};

struct DL : decltype(k) {
  DL() { }

  int operator()(int i) {
    return decltype(k)::operator()(i);
  }
} dl;
```
Do you already hate it? Good, so do I. By god do I hate this. It's fucking *AWESOME*. But unfortunately this particular house of cards breaks down in an entirely hurricaneorific fashion as soon as your lambda turns into a **closure**:
```
auto k = [&capture](int i) {
  return i + capture;
};

struct DL : decltype(k) {
  DL() { }

  int operator()(int i) {
    return decltype(k)::operator()(i);
  }
} dl;
---
error: a lambda closure type has a deleted default constructor
```
(A *"closure"* is ... let's keep it simple and say that a "function" cannot know anything except for its *parameters*, and a "closure" *can*. So a `[&]`-capture makes your lambda a closure and technically not a "function". Kinda. Computer science is funky business.)

### *GCC lets you fuck with captures*
Because a lambda is a struct and `[&]`-captured variables have to be represented somehow.
```
int j = 0;
auto l = [&](int i) {
  return i+j;
};

int j2 = 15;
l.__j = j2;
```
GCC *will let you do that* and *it works* and shoot yes you can *abuse it in so many glorious ways*. Unfortunately, like `__int128_t` this is a *vendor-specific* thing (which is formalspeak for "your compiler might hate this and enthusiastically refuse to compile it") and it does not work in Clang because Clang makes the captures `private` (it also does not add the leading underscores but who cares at this point, damn loser compiler).


## Detective ABI and the Case of the Silent Schism

Alright, young ones, gather 'round. Time for a history lesson!

Ok, so to you youngsters this might be a boomer topic, but: there was a time when C++11 was like Python3. Brand-spanking new. Shiny. You knew it was the future, but of course 99.9% of your codebase were OLD CRUST (still are, I bet? *keheheeheh*) and their maintainers (if by some goddamn *miracle* they were actually still around) refused to recompile their whole garbage heap because even they were scared shitless of their own ghosts, and what's RAII anyway, Unit tests? I've never heard of 'em, and now you were stuck with prehistoric code *and compiled binaries* and this meant that **you would just not get to use C++11, suck it up**.

I am talking about ABIs, of course.
```
lib.cpp
---
#include <string>

void make_string(std::string) {
}
```
This function would, in *Þe Olden Days*, make GCC (something like version 4.8 or something, honestly this oldtimer don't remember that good) produce the name
```
$ g++-4 lib.cpp
$ nm --demangle lib.o
0000000000000000 T make_string(std::string)
```
So far, so nice.

But *god forbid* you tried linking this object into something compiled with the *new* GCC **5**!
```
main.cpp
---
#include <string>

void make_string(std::string);

int main() {
  make_string("ok boomer");
}
---
$ g++-5 lib.o main.cpp
/tmp/bFLkbfli.o: In function `main':
whot.cpp:(.text+0x43): undefined reference to `make_string(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >)'
collect2: error: ld returned 1 exit status
```
Now you've done it, good job. You have torn space-time and summoned an Old God. Slow clap.

What happened? How did we go from `std::string` to `make_string(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >)`? And WHY?!

Ok, so first — `make_string(std::string)` isn't actually the full name of that symbol. `nm --demangle` is just kinda smart and tries to be helpful, good girl that she is. But not this time, Lassie, this time we actually want the raw fish:
```
$ nm lib.o | c++filt
0000000000000000 T make_string(std::basic_string<char, std::char_traits<char>, std::allocator<char> >)
```
Euggh. Yeah, that... looks righter. More like the chaos you are used to, not like the polished veneer that `nm --demangle` tried to soothe you with. Now we can actually try to compare those two idiots:
```
Old and busted: make_string(std::basic_string<char, std::char_traits<char>, std::allocator<char> >)
New hotness:    make_string(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >)
```
...see that? Some sneaky fucker snuck a fucky extra `::__cxx11` in there! Between C++03 and C++11, the ABI (*“Application Binary Interface”*, kinda like an API for the compiler/linker, if you will) *changed* in subtle but *fundamentally incompatible* ways (questions? [answers](https://www.youtube.com/watch?v=7RoTDjLLXJQ)). Just like Python3 and poorly written Python2 libraries, or horses and no carriages, you **just can't have both**.

And that, my young friends, was when you said Fuck it and went home for the day. Because here be dragons. *This line* meant that *someone* would either have to recompile the whole goddamn codebase ... or you poor sod would just. not. get. to use fancy new C++ features. Guess which one usually happened.

—

Ah but the Old One she is merciful. When she grants you her unholy audience one moonless night, her dark whispers inside your head take on the shapes of terrible forbidden runes, impossible to utter with human flesh and voice but with the power to alter reality: `-D_GLIBCXX_USE_CXX11_ABI=0`.
```
$ g++-5 -D_GLIBCXX_USE_CXX11_ABI=0 lib.o main.cpp
```
You sit in silence as your code compiles. No dogs are howling as the linker goes to work on its grim task. No portent of evil rends the night with blood-curdling screeches. You know you have done a despicable thing, an *evil* thing. Like a zombie, your executable will blend in with the other programs, relying on the fact that nobody will ever bother to check its symbols to discover that **it isn't fully formed, it's just a human-shaped blob of flesh, imitating a human mind with dark purpose**. But your middle manager will not have to chew you out on monday for not having passed those mindless integration tests, and that is good. Saving your own hide is nothing but human. You pave your road with another good intention.

—

The thunder is close now, already less of a low rumble and more deafening bone-dry cracks. It is blackest night and the howling storm is throwing down with mindless fury wave after wave of sleety rain onto the weary tin roof. No animal with four legs or fewer is seeking shelter under your porch; tonight the weather is the lesser evil to face. Inside, under the unnervingly metastable 6500K glare of a naked Singaporean 4 Watt LED driven too hot by at least a full sigma, you are declaring a nontrivial **function pointer**. 

```
void (*(*f[n])())();
```

You know, I have a theory. That back in the 1970's, some poor soul was about to just not have it. V'ger about to launch? Unravel the mysteries of ~harmony and focus~ the universe? Fuck that. Mr X about to make some new mysteries right here down on this earth goddamnit. Give people power they should never have. An n-element array of pointers to functions which return pointers to functions that themselves return void. Move over, linear-A. A new boss is in town.

Really though, function pointers are just too cheap to rap on. Not even gonna go there. Forget they exist. *Please.* Be happy with lambdas and `std::function`. No man needs more in life or death. We slapped `++` on that `C` for a reason, and that reason was mercy.

—

## The Chests Unopened
Behold, the ones still slumbering in the void.
- pointers to member functions, the level of indirection you always knew you didn't want
- `typename template`, the emperor of torment
- the story of `dynamic_cast` and the struct that would not stay virtual

May the lock be strong and the watch ever faithful.
