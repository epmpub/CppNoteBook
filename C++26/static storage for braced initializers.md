## static storage for braced initializers

在之前的 C++ 标准中，当您使用花括号初始化列表（braced-init-list，例如 `{1, 2, 3}`）来初始化诸如 `std::initializer_list` 之类的对象时，编译器通常需要在程序运行到该代码处时，在**栈（stack）**上创建一个临时数组并进行初始化。

这一特性的引入，允许编译器在特定条件下（通常是列表内容为常量时），将这些初始化数据的底层数组存放在**静态存储区**（Static Storage，如只读数据段）中。

2. 主要优势：性能优化

通过将数据从“栈临时存储”改为“静态存储”，该特性带来了显著的性能提升：

**减少重复开销**：如果一个函数被多次调用，且内部包含固定的初始化列表，编译器不再需要每次调用时都在栈上重新分配空间和初始化数组。**减小栈空间使用**：对于大型的常量数据列表，这能有效防止栈溢出并降低内存压力。

```c++
#include <print>

void do_something(int x){
    std::print("output is {}\n",x);
}

void process_constants() {
    for (int x : {-1,0,1}) { 
        // 这里的列表现在享有静态存储
        do_something(x);
    }
}

int main(){
    process_constants();
}
```

