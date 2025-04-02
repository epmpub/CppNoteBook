# user-defined literals

C++11 引入了用户定义文字，允许库以自然且可读的方式公开强类型单元值类型。

请注意，用户定义文字的唯一允许基类型是 unsigned long long int、long double、字符类型和字符串文字类型。

```c++
#include <print>

 namespace unit { 
    struct  m { 
        unsigned  long  long  int value; 
        friend  auto  operator <=>( const m&, const m&) = default ; 
    }; 
    struct  km { 
        unsigned  long  long  int value; 
        operator  m ()  const  { return {value* 1000 }; } 
        friend  auto  operator <=>( const km&, const km&) = default ; 
    }; 
} 

constexpr unit::km operator "" _km( unsigned  long  long  int v) { 
 return {v}; 
} 
constexpr unit::m operator "" _m( unsigned  long  long  int v) { 
 return {v}; 
} 

int  main ()  { 
    if ( 100 _m < 1 _km) { 
        std:: println ( "100m 小于 1km" ); 
    } 
    if ( 1001 _m > 1 _km) { 
        std:: println ( "1001m 大于 1km" ); 
    } 
}
```

[在 Compiler Explorer 中打开示例。](https://compiler-explorer.com/z/f8TvhcPbe)