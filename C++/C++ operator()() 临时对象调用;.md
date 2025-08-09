# operator()() 临时对象调用

```C++
#include <iostream>

struct foo {

   void operator()()  { // Corrected the operator() method
       std::cout << "operate()" << std::endl;
   }
};

int main() {

   foo{}();// Corrected the call to the operate() method

   return 0;
}
```

