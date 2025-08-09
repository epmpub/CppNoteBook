# New C++ features in GCC 15

April 24, 2025

 Table of contents:

The next major version of the[ GNU Compiler Collection](https://gcc.gnu.org/) (GCC), 15.1, is expected to be released in April or May 2025. Like every major GCC release, this version will bring many [additions, improvements, bug fixes, and new features](https://gcc.gnu.org/gcc-15/changes.html). GCC 15 is already the system compiler in [Fedora 42](https://fedoraproject.org/wiki/Changes/GNUToolchainF42). [Red Hat Enterprise Linux](https://developers.redhat.com/products/rhel/overview) (RHEL) users will get GCC 15 in the Red Hat GCC Toolset. It's also possible to try GCC 15 on [Compiler Explorer](https://compiler-explorer.com/) and similar pages.

Like my previous article, [New C++ features in GCC 14](https://developers.redhat.com/articles/2024/05/15/new-c-features-gcc-14), this article describes only new features implemented in the C++ front end; it does not discuss developments in the C++ language itself.

The default dialect in GCC 15 is still `-std=gnu++17`. You can use the `-std=c++23` or `-std=gnu++23` command-line options to enable C++23 features, and similarly for C++26 and others. 



Warning

Note that C++20, C++23, and C++26 features are **still experimental** in GCC 15. (GCC 16 plans to switch the default to C++20.)



## C++26 features

C++26 features in GCC 15 include pack indexing, attributes for structured bindings, enhanced support for functions whose definition consists of `=delete`, and more.

### Pack indexing

C++11 introduced [variadic templates](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2007/n2242.pdf) which allow the programmer to write templates that accept any number of template arguments. A pack can represent a series of types or values. For example, to print out arbitrary arguments, one could write:

```cpp
template<typename T, typename... Types>
void print (T t, Types... args)
{
  std::cout << t << '\n';
  if constexpr (sizeof...(args) > 0)
    print (args...);
}

int main ()
{
  print ('a', "foo", 42);
}
```

Copy snippet

However, it was not possible to index an element of a pack, unless the programmer resorted to using various recursive tricks which are generally slow to compile. With this C++26 feature, to index a pack one can write `pack...[N]` (where `N` has to be a constant expression). A pack index then behaves exactly as if the resulting expression was used. An empty pack cannot be indexed. The following program will print `a`:

```cpp
template<typename... Types>
void print (Types... args)
{
  std::cout << args...[0] << '\n';
}

int main ()
{
  print ('a', "foo", 42);
}
```

Copy snippet

### Attributes for structured bindings

This proposal allows you to add an attribute that appertains to each of the introduced structured bindings, as in the following example:

```cpp
struct S { int a, b; };

void
g (S& s)
{
  auto [ a, b [[gnu::deprecated]] ] = s;
}
```

Copy snippet

### =delete with a reason

C++11 provided support for [deleted functions](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2007/n2346.htm); that is, functions whose definition consists of `=delete`. Deleted functions participate in overload resolution, but calling them is an error. This replaced the old mechanism of declaring special member functions as `private`. 

In C++26, it is possible to provide a [message](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2573r2.html) explaining why the function is marked as deleted: `=delete(“reason”)`. The following program:

```cpp
void oldfn(char *) = delete("unsafe, use newfn");
void newfn(char *);

void g ()
{
  oldfn ("bagel");
}
```

Copy snippet

will cause the compiler to emit:

```cpp
q.C: In function ‘void g()’:
q.C:7:9: error: use of deleted function ‘void oldfn(char*)’: unsafe, use newfn
7 |   oldfn ("bagel");
  |   ~~~~~~^~~~~~~~~
q.C:1:6: note: declared here
1 | void oldfn(char *) = delete("unsafe, use newfn");
  |      ^~~~~
```

Copy snippet

### Variadic friends

The following [feature](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2893r3.html) makes it possible to use a `friend` declaration with a parameter pack:

```cpp
template<class... Ts>
class Foo {
  friend Ts...;
};
```

Copy snippet

An example where this feature can be used in practice is the `Passkey` idiom:

```cpp
template<typename... Ts>
class Passkey {
  friend Ts...;
  Passkey() {}
};

class A;
class B;

struct Widget {
  // Only callable from A and B.
  void secret (Passkey<A, B>);
};

class A {
  void doit (Widget& w) {
    w.secret ({}); // OK
  }
};

class B {
  void doit (Widget& w) {
    w.secret ({}); // OK
  }
};

class D {
  void doit (Widget& w) {
    w.secret ({}); // won't compile!
  }
};
```

Copy snippet

### constexpr placement new

C++20 added [support](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p0784r7.html) for using `new` in a `constexpr` context:

```cpp
constexpr void use (int *) { }

constexpr int
foo ()
{
  auto *p = new int[]{1, 2, 3};
  use (p);
  delete[] p;
  return 1;
}

int main ()
{
  constexpr auto r = foo ();
}
```

Copy snippet

GCC implemented the proposal in [GCC 10](https://developers.redhat.com/blog/2020/09/24/new-c-features-in-gcc-10#). However, `constexpr placement new` was not yet possible. C++26 rectifies that situation, and it allows the programmer to write code like:

```cpp
#include <memory>

constexpr int foo ()
{
  std::allocator<int> alloc;
  auto p = alloc.allocate (16);
  new (p) int(42);
  alloc.deallocate (p, 16);
  return 1;
}

int main ()
{
  constexpr int r = foo ();
}
```

Copy snippet

### Structured binding declaration as a condition

The [structured bindings](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0217r3.html) feature was added in C++17, and GCC has supported it for a long time. GCC 15 implements [P0963R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p0963r3.html), which allows structured bindings declaration in `if`/`while`/`for`/`switch` conditions; previously, this wasn’t possible. 

If a structured binding is used in a condition context, its decision variable is the underlying artificial variable created by the compiler. This variable must be convertible to `bool` (except when used in a `switch` statement). For example:

```cpp
struct S {
  int a, b;
  explicit operator bool () const noexcept { return a != b; }
};
void use (int, int);

void g (S s)
{
  if (auto [ a, b ] = s)
    use (a, b);
}
```

Copy snippet

In the preceding example, `use` will be called when `a` and `b`, decomposed from `s`, are not equal. The artificial variable used as the decision variable has a unique name and its type here is `S`.

### Deleting a pointer to an incomplete type

C++26 [made it clear](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p3144r2.pdf) that `delete` and `delete[]` on a pointer to an incomplete class type is invalid. Previously it invoked undefined behavior unless the class had a trivial destructor and no custom deallocator. Consequently, GCC 15 will emit an error in C++26 mode, or a warning in older modes for this example:

```cpp
struct S;

void foo (S *p)
{
  delete p;
}
```

Copy snippet

### The Oxford variadic comma

This [paper](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p3176r0.html) deprecates the use of a variadic ellipsis without a preceding comma in C++26. That means that GCC 15, when compiling the following test case from the proposal, will emit three warnings in C++26 mode:

```cpp
void d_depr(auto......); // deprecated
void d_okay(auto..., ...);  // OK

void g_depr(int...);     // deprecated
void g_okay(int, ...);   // OK

void h_depr(int x...);   // deprecated
void h_okay(int x, ...); // OK
```

Copy snippet

Users can enable the warning in older modes by specifying `-Wdeprecated-variadic-comma-omission`.

### Remove deprecated array comparison

This [paper](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2865r5.pdf) makes the following program comparing two arrays ill-formed:

```cpp
int arr1[5];
int arr2[5];
bool same = arr1 == arr2;
```

Copy snippet

The comparison was deprecated in C++20, but GCC 14 emitted a warning only when `-Wall` was specified. GCC 15 emits an error in C++26, and a warning in older modes even without `-Wall`.

### #embed

This [paper](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p1967r14.html), first proposed for C23, has made its way into C++26. It gives the programmer a new directive to include binary data into the program. 

A dedicated article discusses this feature in detail: [How to implement C23 #embed in GCC 15](https://developers.redhat.com/articles/2025/01/30/how-implement-c23-embed-gcc-15#)



## Defect report resolutions

A number of defect reports were resolved in GCC 15. A few examples follow. The overall status can be viewed [here](https://gcc.gnu.org/projects/cxx-dr-status.html).

### Redeclaration of **using-declarations**

[DR 36](https://cplusplus.github.io/CWG/issues/36.html) clarifies that it’s valid to redeclare an entity via a `using-declaration` if the redeclaration happens outside a class scope. As a result, the following program compiles with GCC 15. GCC 14 would report a “redeclaration” error.

```cpp
enum class E { Smashing, Pumpkins };
void foo() {
  using E::Smashing;
  using E::Smashing;
  using E::Pumpkins;
}
```

Copy snippet

### Trivial fixes

GCC 15 fixes [DR 1363](https://cplusplus.github.io/CWG/issues/1363.html) and [DR 1496](https://cplusplus.github.io/CWG/issues/1496.html). Certain classes that were previously considered trivial are not trivial anymore. The details can be found in this [commit message](https://gcc.gnu.org/git/?p=gcc.git;a=commitdiff;h=9690fb3a43e5cf26a5fb853283d4200df312a640).

### Bit-fields and narrowing conversions

The resolution of [DR 2627](https://cplusplus.github.io/CWG/issues/2627.html) means that GCC 15 no longer emits the `narrowing conversion` warning here:

```cpp
#include <compare>

struct C {
  long long i : 8;
};

void f() {
  C x{1}, y{2};
  x.i <=> y.i; // OK
}
```

Copy snippet

### Overloaded functions and constraints

[DR 2918](https://cplusplus.github.io/CWG/issues/2918.html) deals with deduction from an overload set when multiple candidates succeed and have the same type; the most constrained function is selected. The following test case shows this new behavior:

```cpp
template<bool B> struct X {
  static void f(short) requires B; // #1
  static void f(short);            // #2
};
void test() {
  auto x = &X<true>::f;     // OK, deduces void(*)(short), selects #1
  auto y = &X<false>::f;    // OK, deduces void(*)(short), selects #2
}
```

Copy snippet



## Additional updates

This section describes other enhancements in GCC 15.

### Fix for range-based for loops

[This paper](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2718r0.html) fixes a long-standing problem with range-based for loops, which caused much grief in practice. The issue was that the lifetime of the temporaries used in the initializer of a range-based for loops weren’t extended, causing undefined behavior. In C++23, the problem was corrected and now the lifetime of the temporaries is extended to cover the whole loop.

The effect is that the following program will print in C++20:

```cpp
~T()
loop
```

Copy snippet

But in C++23, it will print:

```cpp
loop
~T()
```

Copy snippet

It is possible to control this behavior with the `-frange-for-ext-temps` and `-fno-range-for-ext-temps` options.

```cpp
struct T {
  int arr[1];
  ~T() { std::printf ("~T()\n"); }
  const int *begin () const { return &arr[0]; }
  const int *end () const { return &arr[1]; }
};
const T& f(const T& t) { return t; }
T g() { return T{42}; }

int main ()
{
  for (auto e : f (g ()))
    std::printf ("loop\n");
}
```

Copy snippet

As an aside, this proposal added the fourth case where lifetime extension takes place. [DR 2867](https://cplusplus.github.io/CWG/issues/2867.html), also implemented in GCC 15, added the fifth case, which deals with lifetime extension in structured bindings.

### Qualified lookup failures diagnosed early

GCC 15 diagnoses certain invalid qualified lookups earlier, while parsing the template. When the scope of a qualified name is the current instantiation, and qualified lookup finds nothing at template definition time, then we know that qualified lookup will find nothing at instantiation time either (unless the current instantiation has dependent bases). 

Therefore, such qualified name lookup failures can be diagnosed ahead of time. This is allowed by the C++ standard—[temp.res.general]/6 says: “The validity of a templated entity may be checked prior to any instantiation.” Consequently, the following program is rejected by GCC 15:  

```cpp
template<typename T>
struct S {
  void foo() {
    int i = this->nothere;
  };
};
```

Copy snippet

But this one isn’t:

```cpp
template<typename T>
struct S : T {
  void foo() {
    int i = this->nothere;
  };
};
```

Copy snippet

because `nothere` could be found in the dependent base.

### Concepts TS removed

The support for Concepts TS was removed and `-fconcepts-ts` has no effect anymore. Programs using the Concepts TS features need to convert their code to C++20 Concepts. For example:

```cpp
template<typename T>
concept C = true;

C{T} void foo (T); // write template<C T> void foo (T);
```

Copy snippet

### -fassume-sane-operators-new-delete

GCC 15 gained the `-fassume-sane-operators-new-delete` option which can be used to adjust the behavior regarding optimizations around calls to replaceable global operators new and delete. The [manual](https://gcc.gnu.org/onlinedocs/gcc/C_002b_002b-Dialect-Options.html#index-fassume-sane-operators-new-delete) provides more information about this option.

### C++ modules

GCC 15 greatly improved the modules code. For instance, `module std` is now supported (even in C++20 mode).

### Compile-time speed improvements

Code that uses a lot of template specializations should compile faster with GCC 15 due to the improvements in the hashing of template specializations.

### flag_enum

GCC 15 has a new attribute called `flag_enum` (see the [manual](https://gcc.gnu.org/onlinedocs/gcc/Common-Type-Attributes.html#index-flag_005fenum-type-attribute) for more information). It can be used to suppress a `-Wswitch` warning emitted when enumerators are used in bitwise operations. For example, without the attribute the following test case:

```cpp
enum [[gnu::flag_enum]] E { cheer = 1, blue = 2 };
void f (enum E e) {
  switch (e) {
  case blue|cheer:
  default:;
  }
}
```

Copy snippet

would result in a `case value ‘3’ not in enumerated type ‘E’` warning.

### New prvalue optimization

GCC 15 tries harder to constant evaluate class prvalues used as function arguments. Previously, there was a discrepancy between the behavior of:

```cpp
  Widget w{"Shepp"};
  do_something (w);
```

Copy snippet

and:

```cpp
do_something (Widget{“Shepp”});
```

Copy snippet

As a result of this optimization, GCC is able to compile code that it previously wasn’t.

### C++11 attributes in C++98

GCC 15 allows C++11 attributes to be used even in C++98, which previous versions of GCC sometimes refused to compile. For example, the following code compiles even in C++98 mode:

```cpp
struct [[gnu::packed]] S { int sardines; };
```

Copy snippet

With `-Wpedantic` the compiler warns when it encounters a C++11 attribute in C++98 mode. 



## New and improved warnings

GCC's set of warning options have been enhanced in GCC 15.

### -Wtemplate-body

Errors in uninstantiated templates are IFNDR (ill-formed, no diagnostic required), but GCC progressively diagnoses more and more errors in template definitions ahead of time to catch errors sooner. For example:

```cpp
template<typename>
void foo ()
{
  const int n = 42;
  ++n; // error, no valid instantiation exists
}
```

Copy snippet

For various reasons, the more aggressive behavior can be undesirable so GCC 15 added the `-Wno-template-body` [option](https://gcc.gnu.org/onlinedocs/gcc/C_002b_002b-Dialect-Options.html#index-Wtemplate-body), which can be used to disable the template errors.

### -Wself-move

The[ -Wself-move](https://gcc.gnu.org/onlinedocs/gcc/Warning-Options.html#index-Wself-move) warning now warns even in a `member-initializer-list`:

```cpp
#include <utility>

struct B {
int i_;
B(int) : i_(std::move(i_)) { } // warning
};
```

Copy snippet

### -Wdefaulted-function-deleted

In GCC 15 we fixed our handling of `[dcl.fct.def.default]`. For example, the following test used to be rejected with an error:

```cpp
struct C {
  C(const C&&) = default;
};
```

Copy snippet

But instead, the move constructor should be marked as deleted. The reason is that the function’s type doesn’t match the type of an implicit move constructor. GCC 15 accepts the code, but warns about it by default with this [new warning](https://gcc.gnu.org/onlinedocs/gcc/C_002b_002b-Dialect-Options.html#index-Wdefaulted-function-deleted).

### -Wheader-guard

This [new warning](https://gcc.gnu.org/onlinedocs/gcc/Warning-Options.html#index-Wheader-guard) catches typos in header file guarding macros. For instance, including the following `.h` file:

```cpp
#ifndef STDIO_H
#define STDOI_H
extern void elmo ();
extern void hope ();
#endif
```

Copy snippet

will cause a warning to be emitted (if `-Wall` was specified): `did you mean ‘STDIO_H’?`

### -Wdangling-reference

This [warning](https://gcc.gnu.org/onlinedocs/gcc/C_002b_002b-Dialect-Options.html#index-Wdangling-reference) has been improved in GCC 15. For instance, it no longer warns for empty classes.



## Acknowledgments

As usual, I'd like to thank my coworkers at Red Hat who made the GNU C++ compiler so much better, notably Jason Merrill, Jakub Jelinek, Patrick Palka, and Jonathan Wakely. We would also like to thank Nathaniel Shead for his work on C++ modules.

*Last updated: April 25, 2025*