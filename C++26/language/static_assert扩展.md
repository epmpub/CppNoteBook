

C++26 static_assert & delete



```c++
#include <string_view>
#include <typeinfo>
constexpr std::string_view get_error_message()
{
    return "The required condition was not met in this specific context.";
}


template <typename T>
constexpr std::string_view generate_type_size_error()
{
    return  std::string_view("is too large. Maximum allowed size is 8 bytes.");
}


template <typename T>
void process_data(T value) {
    // 假设我们有一个编译时函数可以根据类型 T 生成描述字符串
    // 这在以前是无法直接放入 static_assert 的
    static_assert(sizeof(T) <= 8, generate_type_size_error<T>()); 
}

struct Large {
    char msg[8];
};

template <>
void process_data<int>(int) = delete("dis allow to call");

int main(){
    static_assert(sizeof(int) == 4, "Integer size must be 4 bytes"); 
    static_assert(sizeof(int) == 4, get_error_message());

    // process_data(42);  //can't call ,because process_data<int>(int) has been deleted.
    process_data(3.14);
    process_data(Large{});
    return 0;
}

```

