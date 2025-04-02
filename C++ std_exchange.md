# std::exchange

C++14 *std::exchange*是一个简单的实用程序，它将第一个参数设置为提供的值并返回原始值。

注意: **std::swap 和 std::exchange的区别**

虽然很简单，但这种行为大大简化了移动语义和其他用例的典型实现，否则这些用例必须依赖临时变量。

```c++
 # include  <utility>
 # include  <generator>
 # include  <iostream>

 // std::exchange 作为移动语义的辅助程序
struct  EraseOnMove { 
    EraseOnMove ( int value) : value_ (value) {} 
    EraseOnMove (EraseOnMove&& other) 
        : value_ (std:: exchange (other.value_, 0 )) {} 
    EraseOnMove& operator =(EraseOnMove&& other) { 
        // 对于 this == &other 是安全的
        value_ = std:: exchange (other.value_, 0 ); 
        /* 注意：以下对于 this == &other 
            value_ = std::move(other.value_) 不安全；
            other.value_ = 0; 
        */ 
        return * this ; 
    } 
    int value_; 
}; 

EraseOnMove a{ 10 }; 
EraseOnMove b (std::move(a)) ; 
// a == {0}, b == {10} 


// 使用 std::exchange 的斐波那契数列
std::generator< int > fibonacci ( size_t cnt)  { 
    int a = 0 ; 
    int b = 1 ; 
    while (cnt > 0 ) { 
        co_yield  std::exchange (a, std::exchange(b, a+b)) ; 
        /* 与以下相同：
        int b_old = b; 
        b = a + b; 
        int a_old = a; 
        a = b_old; 
        co_yield a_old; 
        */
         --cnt; 
    } 
} 

// std::exchange 作为分隔输出的辅助程序
std::string delim = "" ; 
for ( int f : fibonacci ( 10 )) 
    std::cout << std:: exchange (delim, ", " ) << f; 
std::cout << "\n" ; 
// 打印：0、1、1、2、3、5、8、13、21、34
```

[在 Compiler Explorer 中打开示例。](https://compiler-explorer.com/z/M5sMGMfnn)