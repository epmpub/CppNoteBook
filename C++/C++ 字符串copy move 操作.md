# 字符串copy move 操作



```C++
// securityStr.cpp : This file contains the 'main' function. Program execution begins and ends there.
//

#include <iostream>
#include <cstring>
#include <string>
#include <algorithm>
#include <utility>

int main()
{
    const char* str = "Hello World!";
    //size_t LEN = strlen(str) + 1;
    //char *dest = new char[13];
    std::string dest;

    //memcpy_s(dest, LEN, str, LEN);
    //memmove_s(dest, LEN, str, LEN);
    std::copy(str, str + 13, std::back_inserter(dest));
    //std::move(str, str + LEN, dest);
    //strcpy_s(dest, LEN, str);
    printf_s("%s", dest.c_str());

}

```

