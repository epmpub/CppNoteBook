# std::initializer_list。

std *::initializer_list* (C++11) 是围绕 const 数组的简单代理对象，当大括号初始化列表如下时会自动构造该对象：

- 用作函数参数
- 使用适当的构造函数/赋值运算符来初始化或分配给对象
- 绑定到汽车

请注意，我们不能从 const 数组移动，这会影响性能。

```
# include  <initializer_list>
# include  <vector>

 struct  X { 
    X (std::initializer_list< int >) {} 
    X& operator =(std::initializer_list< int >){ return * this ; } 
}; 

void  function (std::initializer_list< int >)  {} 

struct  Data {}; 


// 构造一个对象
X x{ 1 , 2 , 3 , 4 , 5 }; 

// 分配给一个对象
x = { 1 , 2 , 3 , 4 , 5 }; 

// 明确接受 initializer_list 的
函数 function ({ 1 , 2 , 3 , 4 , 5 }); 

// 绑定到自动
auto y = { 1 , 2 , 3 , 4 , 5 }; 
// decltype(y) == std::initializer_list<int> 

// 通过复制初始化初始化
Data a, b; 
std::vector<Data> z{a, b}; 
// 4x 复制
// 2x 创建 std::initializer_list 
// 2x 创建 std::vector 

// C++17 保证复制省略
std::vector<Data> w{Data{}, Data{}}; 
// 从 pr 值的复制初始化变为直接初始化
// 2x 复制创建 std::vector
```

[在 Compiler Explorer 中打开示例。](https://compiler-explorer.com/z/WjzETocx7)