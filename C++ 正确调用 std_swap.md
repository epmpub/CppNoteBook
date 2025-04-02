# 正确调用 std::swap

*当将std::swap*与用户定义类型一起使用时，我们需要小心参数相关查找的后果。

*对std::swap 的*限定调用将始终调用默认实现。

非限定调用可以使用 ADL 找到用户定义的实现，但看不到默认实现。

因此，为了正确调用*std::swap*，我们必须首先将默认实现拉入本地范围，然后再进行无限定调用。

*或者，使用 C++20，对std::ranges::swap*的合格调用将始终执行正确的操作。

```cpp
# include  <algorithm>

 namespace MyNamespace { 
struct  MyClass { 
    // 使用内联友元函数实现自定义交换。
    friend  void  swap (MyClass&, MyClass&)  {} 
}; 

struct  MyOtherClass {}; 
} 

// 请注意，在全局范围内名为 swap 的非函数声明
// 将在非限定查找期间被发现
// ，并会阻止 ADL。
// constexpr auto swap = "Hello!";

MyNamespace::MyClass a, b; 
MyNamespace::MyOtherClass x, y; 

// 完全限定调用，将始终调用 std::swap
 std:: swap (a,b); // 调用 std::swap
 std:: swap (x,y); // 调用 std::swap 

// 没有适合 MyOtherClass 的交换。
swap (a,b); // 调用 MyNamespace::swap 
// swap(x,y); // 无法编译

// 将 std::swap 作为默认值拉入本地范围：
 { 
using std::swap; 
swap (a,b); // 调用 MyNamespace::swap 
swap (x,y); // 调用 std::swap
 } 

// C++20 std::ranges::swap 将执行正确的操作：
 std::ranges:: swap (a,b); // 调用 MyNamespace::swap
 std::ranges:: swap (x,y); // 默认交换
```

[在 Compiler Explorer 中打开示例。](https://compiler-explorer.com/z/4nY1sK3Kn)