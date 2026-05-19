C++ 的 RTTI（Run-Time Type Information）核心解决的是：

> “在运行时识别对象的真实动态类型”。

它主要用于：

# 多态（polymorphism）场景下的类型识别与安全转换。

------

# 一、先理解问题来源

C++ 有：

# 继承 + 虚函数

例如：

```cpp
struct Animal
{
    virtual ~Animal() = default;
};

struct Dog : Animal
{
    void bark() {}
};

struct Cat : Animal
{
    void meow() {}
};
```

然后：

```cpp
Animal* p = new Dog;
```

问题来了：

# `p` 的静态类型是 `Animal*`

# 但真实对象是 `Dog`

那么：

```text
运行时怎么知道真实类型？
```

------

# 二、RTTI 解决的核心问题

RTTI 提供：

# “运行时动态类型信息”

即：

```text
这个对象真正是什么类型？
```

------

# 三、RTTI 的两个核心能力

------

# 1. `dynamic_cast`

安全向下转型。

------

例如1：

```
class Animal { public: virtual void speak() = 0; };
class Dog : public Animal { public: void bark() { } };
class Cat : public Animal { };

Animal* a = new Dog();

Dog* d = dynamic_cast<Dog*>(a);
if (d) {
    d->bark(); // 安全！只有真的是 Dog 才会执行
}

```



例如2：

```cpp
Animal* p = new Dog;

Dog* d = dynamic_cast<Dog*>(p);
```

如果：

```text
p 真的是 Dog
```

则：

```text
d != nullptr
```

否则：

```text
nullptr
```

------

# 2. `typeid`

获取运行时类型。

例如：

```cpp
typeid(*p).name()
```

可能得到：

```text
Dog
```

------

# 四、为什么需要 RTTI？

因为：

# 多态隐藏了真实类型。

例如：

```cpp
std::vector<Animal*>
```

里面：

```text
Dog
Cat
Bird
```

混在一起。

程序运行时：

可能需要：

- 判断真实类型
- 做特定处理
- 安全转换

------

# 五、没有 RTTI 会怎样？

你只能：

# 手动维护类型标签。

例如：

```cpp
enum Type
{
    DOG,
    CAT
};
```

然后：

```cpp
if (obj->type == DOG)
```

问题：

- 容易出错
- 不可扩展
- inheritance 不自然
- 破坏封装

------

RTTI：

# 编译器自动维护动态类型信息。

------

# 六、RTTI 的底层原理

这是关键。

只要类里有：

```cpp
virtual function
```

编译器通常会生成：

------

# vtable（虚函数表）

和：

# typeinfo（类型信息）

------

对象布局类似：

```text
object
 ├── vptr
      ↓
    vtable
      ├── virtual funcs
      └── RTTI/typeinfo
```

------

因此：

```cpp
dynamic_cast
typeid
```

都能：

# 通过 vtable 找到真实类型。

------

# 七、RTTI 最经典用途

------

# 1. 多态容器

例如：

```cpp
std::vector<Base*>
```

------

# 2. 插件系统

运行时判断：

```text
具体插件类型
```

------

# 3. GUI Framework

例如：

```text
Widget*
```

实际：

- Button
- Slider
- Label

------

# 4. Game Engine

例如：

```text
Entity*
```

实际：

- Player
- Enemy
- NPC

------

# 八、`dynamic_cast` 真正解决了什么？

重点：

# “安全 downcast”

普通 cast：

```cpp
Dog* d = (Dog*)p;
```

危险：

如果：

```text
p 实际是 Cat
```

则：

# UB（未定义行为）

------

而：

```cpp
dynamic_cast
```

会：

# 运行时检查类型。

------

# 九、为什么 RTTI 只对多态类有效？

因为：

# RTTI 信息放在 vtable 里。

没有：

```cpp
virtual
```

就：

# 没有 vtable。

因此：

```cpp
dynamic_cast
```

无法工作。

------

# 十、RTTI 的问题

虽然有用。

但很多大型 C++ 项目：

# 禁用 RTTI。

例如：

- Unreal Engine
- LLVM（部分）
- Chromium（部分）

------

原因：

1. runtime overhead

需要：

- typeinfo
- vtable metadata

2. binary size 增大

3. dynamic_cast 较慢

需要：

runtime hierarchy traversal。

设计味道（code smell）

很多人认为：

```text
频繁 dynamic_cast
```

意味着：

OOP 设计不好。

# 十一、现代 C++ 更推荐什么？

很多时候：

virtual dispatch 本身就够。

例如：

不要：

```cpp
if(dynamic_cast<Dog*>)
```

而是：

```cpp
virtual speak()
```

利用：

多态 dispatch。



十二、现代替代方案

1. virtual function

最经典。

------

# 2. visitor pattern

避免：

```text
type switch
```

------

# 3. std::variant

现代类型安全 union。

------

# 4. ECS/data-oriented

减少 inheritance。

------

# 十三、RTTI 和 Reflection 完全不同

很多人混淆。

RTTI

解决：

“运行时真实类型是什么？”

例如：

```text
Dog or Cat?
```

------

# Reflection

解决：

“类型内部结构是什么？”

例如：

```text
有哪些字段？
字段名字？
字段类型？
```

------

RTTI：

非常有限。

Reflection：

强大得多。

------

# 十四、RTTI 的真正本质

一句话：

> “让多态对象在运行时保留真实类型信息。”

从而支持：

- 安全 downcast
- 动态类型识别

------

# 十五、一个经典现实

```cpp
Base* p = factory();
```

编译期：

# 不知道真实类型。

只有运行时：

```text
plugin loaded
network packet parsed
scene object created
```

才知道。

RTTI：

就是：

# 解决这个动态性。

------

# 十六、一句话总结

C++ RTTI 解决的是：

> “多态对象在运行时如何识别真实类型”的问题。

它通过：

- `dynamic_cast`
- `typeid`
- vtable/typeinfo

提供：

# 运行时动态类型识别（Run-Time Type Information）。

------

# 核心能力

| RTTI 功能          | 作用               |
| ------------------ | ------------------ |
| `dynamic_cast`     | 安全向下转型       |
| `typeid`           | 获取动态类型       |
| typeinfo           | 类型元信息         |
| vtable integration | 支持运行时类型查询 |

------

可以把 RTTI 理解成：

```text
“多态对象的运行时身份证系统”
```