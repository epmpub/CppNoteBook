

# std::quick_exit , std::at_quick_exit , std::_Exit

退出进程而不调用析构函数并使用 std::quick_exit 和 std::_Exit 引发 SIGABRT 信号。

The *std::_Exit* function can be used to exit a process without invoking destructors and without raising the *SIGABRT* signal.

The standard also offers *std::quit_exit*, which additionally calls the functions registered through *std::at_quick_exit*, after which it calls *std::_Exit*.

```C++
#include <iostream>
#include <string>
#include <cstdlib>

struct User {
    std::string label;
    ~User() { std::cerr << "Destructing " << label << "\n";  }
};

User global("global object");

int main() {
    // Destructors of static, dynamic and thread
    // local objects are not called.
    User local("local object");

    // Functions registered using at_quick_exit 
    // will be executed in reverse order.
    std::at_quick_exit([]{ std::cerr << "at exit [1]\n"; });
    std::at_quick_exit([]{ std::cerr << "at exit [2]\n"; });

    // Only functions registered using at_quick_exit will be called.
    std::quick_exit(EXIT_SUCCESS);
    // calls std::_Exit(EXIT_SUCCESS); internally
}
```



