

## **每个初学者都应该知道的 10 个 Python 技巧（附代码示例）**

*Python 是一门优美且对初学者友好的语言，但它也隐藏着许多精妙之处，可以让你的代码更简洁、更快速、更 Python 化。在本文中，我们将介绍 10 个简单却强大的 Python 技巧，每个技巧都通过真实的代码示例，以初学者友好的方式进行讲解。*

## 1. 不使用临时变量来交换变量

而不是这样：

```
a = 10
b = 20

temp = a
a = b
b = temp
```



您可以执行以下操作：

```
a, b = b, a
print(a, b)  # Output: 20 10
```



Python 允许您使用元组解包在一行中交换变量。

## 2. 使用列表推导

而不是：

```
squares = []
for i in range(10):
    squares.append(i * i)
```



尝试一下：

```
squares = [i * i for i in range(10)]
print(squares)
```



更清洁、更快速！

## 3.合并字典（Python 3.9+）

您通常可能会像这样合并两个字典：

```
dict1 = {'a': 1}
dict2 = {'b': 2}
dict1.update(dict2)
```



现在你可以直接写：

```
dict1 = {'a': 1}
dict2 = {'b': 2}
merged = dict1 | dict2
print(merged)  # Output: {'a': 1, 'b': 2}
```



## 4. 使用 enumerate() 代替 Range + Index

不好的方法：

```
fruits = ['apple', 'banana', 'mango']
for i in range(len(fruits)):
    print(i, fruits[i])
```



更好的：

```
fruits = ['apple', 'banana', 'mango']
for i, fruit in enumerate(fruits):
    print(i, fruit)
```



更具可读性和 Pythonic！

## 5. 使用 zip() 迭代多个列表

而不是：

```
names = ['Alice', 'Bob']
ages = [25, 30]

for i in range(len(names)):
    print(names[i], ages[i])
```



使用：

```
names = ['Alice', 'Bob']
ages = [25, 30]

for name, age in zip(names, ages):
    print(name, age)
```



## 6. 使用星号（*）解包

您可以像这样解包值：

```
a, *b, c = [1, 2, 3, 4, 5]
print(a)  # 1
print(b)  # [2, 3, 4]
print(c)  # 5
```



对于灵活的参数处理或忽略中间值很有用。

## 7. 使用 _ 作为一次性变量

如果您不关心循环中的值：

```
for _ in range(5):
    print("Hello")
```



下划线 _ 表示“我不关心这个值。”

## 8. 设置独特元素

无需手动删除重复项：

```
nums = [1, 2, 2, 3, 3, 3]
unique = list(set(nums))
print(unique)  # Output: [1, 2, 3]
```



设置自动删除重复项！

## 9. 使用 get() 获取字典值

避免这种情况：

```
data = {'name': 'Alice'}
print(data['age'])  # KeyError!
```



使用这个：

```
print(data.get('age', 'Not Found'))  # Output: Not Found
```



更安全的访问密钥的方式。

## 10. 使用 any() 和 all()

检查列表中的条件：

```
nums = [2, 4, 6, 8]
print(all(n % 2 == 0 for n in nums))  # True (all even)
print(any(n > 5 for n in nums))       # True (some > 5)
```



## 最后的想法

这些 Python 小技巧可以提升你的代码可读性、效率和风格。坚持实验和构建项目——这是掌握 Python 的最佳方法！

这些技巧中，哪一个对你来说是新的？请在评论区告诉我。🚀