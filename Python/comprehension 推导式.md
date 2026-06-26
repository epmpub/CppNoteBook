前言
Python有一个相当特殊也相当强大的语法叫列表（list）的“推导式”（comprehension），相信大家也都听说过。

```python
[a for a in range(5)]
[0, 1, 2, 3, 4]

python
运行
1
2
以前python的list comprehension被翻译成“列表解析”或者“列表解释”，自从官方的中文文档出来后，我们发现在官方comprehension翻译成“推导式”，个人认为这个翻译更准确。
```

一句话概括，comprehension的本质就是python里面创建列表的一个语法：

下面是最简单的一个comprehension，它生成一个包含0 - 4 这5个数字的list

```python
>>> my_list = [a for a in range(5)]
>>> my_list
>>> [0, 1, 2, 3, 4]
>>> 
>>> python
>>> 运行
>>> 1
>>> 2
>>> 3
>>> 如果想对 0 - 4的数字乘以2，可以如下：

>>> my_list = [a * 2 for a in range(5)]
>>> my_list
>>> [0, 2, 4, 6, 8]
>>> 
>>> python
>>> 运行
>>> 1
>>> 2
>>> 3


```

>>> 语法
>>>
>>> 我们看看compresion的语法如下：
>>> new_list = [expresion_for_member for member in iterable if condition]
>
>

1.expresion_for_member是表达式。

2. member是可迭代的对象或值。
3. iterable是一个list，set，序列生成器或可以一次返回其元素的任何其他对象。

这个例子中 ：
a * 2 是 expresion 表达式，
a就是member
range(5) 就是iterable可循环的对象

条件过滤
如果想对iterable进行过滤，可以加上后面的if condition，比如：

```python
my_list = [a * 2 for a in range(5) if a > 0]
my_list
[2, 4, 6, 8]

python
运行
1
2
3
上面的代码过滤了iterable，只让a > 0的值参与列表的构建。
```

被循环的对象
iterable除了是list，还可以是set，tuple等：

```python
>>> org_list = ['a', 'b', 'c']
>>> my_list = [a for a in org_list]
>>> my_list
>>> ['a', 'b', 'c']
>>> 
>>> python
>>> 运行
>>> 1
>>> 2
>>> 3
>>> 4
>>> 或者复杂一点：

>>> my_list = [a for a in org_list[:1]]
>>> my_list
>>> ['a']
>>> 
>>> python
>>> 运行
>>> 1
>>> 2
>>> 3
>>> 用comprehension构建其它对象
>>> 除了构建list列表，python还可以用comprehension推导式创建set（集合）或者dict（字典），用法和list基本一样。

>>> my_set = {a * 2 for a in range(5) if a > 0}
>>> my_set
>>> {8, 2, 4, 6}
>>> 
>>> python
>>> 运行
>>> 1
>>> 2
>>> 3
>>> 构建dict则需要指定一个key和一个value，这两个都是表达式，和我们平常定义dict一样，key和value中间用冒号“:”区分开。

>>> my_dict = {'key{}'.format(a) : a * 2 + 3 for a in range(5) if a > 0}
>>> my_dict
>>> {'key1': 5, 'key2': 7, 'key3': 9, 'key4': 11}
>>> 
>>> python
>>> 运行
>>> 1
>>> 2
>>> 3
>>> comprehension还可以构建tuple



>>> my_tuple = tuple(a for a in [1,2,3])
>>> my_tuple
>>> (1, 2, 3)
>>> 
>>> python
>>> 运行
>>> 1
>>> 2
>>> 3
>>> 和list，set，dict的构建不同，构建tuple需要用tuple()函数将generator expression转成一个真正的tuple


```

