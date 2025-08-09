# std::copy_if、std::remove_copy_if

如果我们需要有选择地将元素从一个范围复制到另一个范围，标准提供了`std::copy_if`和`std::remove_copy_if`算法。

两种算法都接受谓词，分别复制谓词返回*真*和*假的*元素。

```c++
# include  <vector>
 # include  <algorithm>
 # include  <iterator>

 std::vector< int > data{ 1 , 2 , 3 , 4 , 5 , 6 , 7 , 8 }; 

std::vector< int > dst1; 
std:: copy_if (data.begin ( ), data.end ( ), // 所有元素
    std:: back_inserter (dst1), // 将元素 push_back 放入 dst1
     []( int v) { return v % 2 == 0 ; }); // 条件
// dst1 == {2, 4, 6, 8}

 std::vector< int > dst2; 
std:: remove_copy_if ( data.begin (), data.end ( ), // 所有元素
    std:: back_inserter (dst2), // 将元素 push_back 放入 dst2
     []( int v) { return v % 2 == 0 ; }); // 否定条件
// dst2 == {1, 3, 5, 7}
```

[在 Compiler Explorer 中打开示例。](https://compiler-explorer.com/z/bYnhf54G5)