# Small Safety Improvements in the C++ 26 Core Language

September 22, 2025/in [C++26](https://www.modernescpp.com/index.php/category/blog/c26-blog/)/by [Rainer Grimm](https://www.modernescpp.com/index.php/author/admin/)

Safety is an important concern in C++26. Contracts are probably the most important feature for safety. But C++26 has much more to offer.

<img src="https://www.modernescpp.com/wp-content/uploads/2025/08/Time26Safety-1-1030x534.png" alt="img" style="zoom:50%;" />

Today, I would like to introduce three small but important improvements in C++26 that solve typical safety issues in C++.

## Disallow binding a returned reference to a temporary

A safety issue can be challenging to find. This code snippet is from the proposal [P2748R5](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2748r5.html), which I refer to throughout this section. I turned it into a minimal executable program.

```c++
// bindReferenceToTemporary.cpp

#include <iostream>
#include <string>
#include <string_view>

const std::string_view& getString() {
    static std::string s = "Hallo Welt!";
    return s;
}

int main() {
    std::cout << getString() << '\n';
}   
```

The GCC compiler already provides a meaningful error message:

<img src="https://www.modernescpp.com/wp-content/uploads/2025/08/bindReferenceToTemporary-1030x249.png" alt="img" style="zoom:50%;" />

Need another example? The following code snippet has a bug.

```c#
struct X {
    const std::map<std::string, int> d_map;
    const std::pair<std::string, int>& d_first;

    X(const std::map<std::string, int>& map)
        : d_map(map), d_first(*d_map.begin()) {}
};
```

Unfortunately, the programmer overlooked the fact that the first element of the pair, known as the key, is constant. This creates a temporary variable. As a result, `d_first `is no longer valid.

This brings us to the next safety improvement.

## Erroneous Behaviour for Uninitialized Reads

Now I will refer to proposal [p2795r5](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2795r5.html).

First of all, which objects does this apply to? These are all objects with automatic storage duration and temporaries. The special behaviour is that these objects are initialized to an arbitrary value. This means that the program has undefined behavior.

This, of course, leaves one question to be answered. What does automatic storage duration mean? The following variables have automatic storage duration :

- Variables that have a block scope and have not been explicitly declared as `static`, `thread_local`, or `extern`. The storage of these variables only remains valid until the block is completed.
- Variables that belong to a parameter scope, such as a function. These are automatically destroyed when the parameter scope is dismantled.
  Two examples from the proposal should clarify these words:

```c
extern void f(int);

int main() {
  int x;     // default-initialized, value of x is indeterminate
  f(x);      // glvalue-to-prvalue conversion has undefined behaviour
}
void f(T&);
void g(bool);

void h() {
  T* p;    // uninitialized, has erroneous value (e.g. null)
  bool b;  // uninitialized, erroneous value may not be valid bool

  f(*p);   // dereferencing has undefined behaviour
  g(b);    // undefined behaviour if b is invalid
}
```

To get to the point: the proposal turns uninitialized read operations, which represented undefined behavior with C++23, into erroneous programs with C++26.

Of course, the complete initialization of automatic variables can be a relatively expensive operation. Therefore, there is an opt-out mechanism.

### The `[[indeterminate]]` Attribute

The  `[[indeterminate]]` attribute represents the opt-out mechanism. This feature is only intended for experts. The attribute allows variables with automatic storage that have not been initialized to be read without the program being erroneous. The following simplified example is from the proposal:

```c
int x [[indeterminate]];
std::cin >> x;

[[indeterminate]] int a, b[10], c[10][10];
compute_values(&a, b, c, 10);

// This class benefits from avoiding determinate-storage initialization guards.
struct SelfStorage {
  std::byte data_[512];
  void f();   // uses data_ as scratch space
};

SelfStorage s [[indeterminate]];   // documentation suggested this

void g([[indeterminate]] SelfStorage s = SelfStorage());   // same; unusual, but plausible
```

The last safety feature is about incomplete types.

## Deleting a Pointer to an Incomplete Type should be ill-formed

An incomplete type is a data type of which only the declaration exists, but no definition. Having a pointer or a reference to an incomplete type is perfectly OK. However, operations that require the size, layout, or member functions of this data type are erroneous.

The new feature is a little more specific: Deleting a pointer to an incomplete class type is ill-formed unless that class type has a trivial destructor and no class-specific deallocation function when completed. This means that the compiler created the destructor of the class, and `operator delete` was not overloaded in the class.

The proposal provides a nice example to illustrate the subtle difference between a trivial destructor and a non-trivial destructor:

- Trivial destructor

```c++
// trivialDestructor.cpp
    
namespace xyz {
  struct Widget; // forward decl
  Widget *new_widget();
} // namespace xyz

int main() {
  xyz::Widget *p = xyz::new_widget();
  delete p;
}

namespace xyz {
struct Widget {
  const char *d_name;
  int         d_data;
  // (implicit) trivial destructor
  // This is the only difference.
}; 

Widget *new_widget() { 
    return new Widget(); 
}
} // namespace xyz
```

- Nontrivial destructor

```c++
// nontrivialDestructor.cpp

namespace xyz {
  struct Widget; // forward decl
  Widget *new_widget();
} // namespace xyz

int main() {
  xyz::Widget *p = xyz::new_widget();
  delete p;
}
namespace xyz {
struct Widget {
  const char *d_name;
  int         d_data;
  ~Widget() {} // nontrivial dtor
  // This is the only difference.
};

Widget *new_widget() { 
    return new Widget(); 
}
} // namespace xyz
```

The second example,` nontrivialDestructor.cpp`, has a nontrivial destructor. In line with the proposal, compilation aborts with an error message:

<img src="https://www.modernescpp.com/wp-content/uploads/2025/08/trivialDestructor-1030x241.png" alt="img" style="zoom:50%;" />

