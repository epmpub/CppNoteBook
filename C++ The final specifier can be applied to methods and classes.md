#  *final* specifier 

The *final* specifier can be applied to **methods and classes.**

When a method is marked as *final*, it cannot be overridden in derived classes. 

When a class is marked as *final*, it cannot be derived from.

Note that *final* is not a reserved keyword, meaning that you can create variables and types with the name *final*. Although, in the case of types, that may lead to some confusion.

```c++
struct A final {};

// Would not compile, A is marked as final.
struct B : A {};

struct X {
    virtual void method() = 0;
};
struct Y : X {
    void method() override final {} 
};

// Would not compile, B::method is marked as final.
struct Z : Y {
    void method() override {}
};

// final is not a reserved keyword
struct final final {}; // struct final, which cannot be derived from

int main() {
    final f; // OK
}
```

[Open the example in Compiler Explorer.](https://compiler-explorer.com/z/3afa95Tf6)