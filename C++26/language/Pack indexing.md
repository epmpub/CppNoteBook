

Pack indexing

```cpp
#include <tuple>
#include <string>
#include <iostream>

std::tuple<int, std::string, int> t{ 100, "hello", 200 };

template <typename T>
void process(const T& tuple_obj) {
    // C++26 语法：引入名为 args 的绑定包
    auto [...args, rest] = tuple_obj; 
    
    // rest 成功绑定了最后一个元素 200
    std::cout << rest << std::endl; 
    
    //print args
    ((std::cout << args << " "), ...);
    std::cout << "\n"; 

}


int main() {
    process(t);
}
```

