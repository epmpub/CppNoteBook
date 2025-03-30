**C++ CRTP（奇异递归模板模式）详解**



**CRTP（Curiously Recurring Template Pattern）** 是一种通过模板继承实现的**静态多态**技术，其核心思想是**基类将派生类作为模板参数**，从而在编译时实现多态行为。以下是 CRTP 的详细解析：

---

### **一、CRTP 的核心机制**
#### **1. 基本结构**
```cpp
template <typename Derived>
class Base {
public:
    void interface() {
        // 将 this 转换为派生类指针，调用派生类方法
        static_cast<Derived*>(this)->implementation();
    }
};

class Derived : public Base<Derived> {
public:
    void implementation() {
        std::cout << "Derived::implementation()" << std::endl;
    }
};

int main() {
    Derived d;
    d.interface(); // 输出：Derived::implementation()
}
```
- **关键点**：派生类 `Derived` 继承自以自身为模板参数的基类 `Base<Derived>`。
- **静态多态**：基类通过 `static_cast<Derived*>(this)` 直接访问派生类方法，无需虚函数。

#### **2. 工作原理**
- **编译时绑定**：基类模板在实例化时已知派生类类型，方法调用在编译期确定。
- **零运行时开销**：无虚函数表（vtable）查找，性能等同于直接调用。

---

### **二、CRTP 的典型应用场景**
#### **1. 静态多态（Static Polymorphism）**
- **替代虚函数**：在不需要运行时多态的场景，通过 CRTP 消除虚函数开销。
- **示例**：策略模式（Policy-Based Design）。
  ```cpp
  template <typename Derived>
  struct Serializer {
      std::string serialize() const {
          return static_cast<const Derived*>(this)->serialize_impl();
      }
  };
  
  class JSONSerializer : public Serializer<JSONSerializer> {
  public:
      std::string serialize_impl() const { return "{ \"data\": 42 }"; }
  };
  
  class XMLSerializer : public Serializer<XMLSerializer> {
  public:
      std::string serialize_impl() const { return "<data>42</data>"; }
  };
  ```

#### **2. 代码复用（Mixin 模式）**
- **组合功能**：通过模板继承为派生类添加通用功能。
- **示例**：添加统计功能。
  ```cpp
  template <typename Derived>
  class Counter {
  public:
      static int get_count() { return count; }
  protected:
      Counter() { ++count; }
      ~Counter() { --count; }
  private:
      static inline int count = 0;
  };
  
  class MyClass : public Counter<MyClass> {};
  
  int main() {
      MyClass a, b;
      std::cout << MyClass::get_count(); // 输出：2
  }
  ```

#### **3. 接口强制（Interface Enforcement）**
- **编译时检查**：要求派生类实现特定方法。
- **示例**：强制实现 `clone` 方法。
  ```cpp
  template <typename Derived>
  class Cloneable {
  public:
      Derived* clone() const {
          return new Derived(*static_cast<const Derived*>(this));
      }
  };
  
  class Shape : public Cloneable<Shape> {
      // 必须定义拷贝构造函数
  };
  ```

---

### **三、CRTP 的优缺点**
| **优点**                    | **缺点**                     |
| --------------------------- | ---------------------------- |
| 无运行时开销（无虚函数）    | 代码可读性降低               |
| 编译时多态，性能更优        | 模板错误信息复杂             |
| 支持值语义（无需指针/引用） | 派生类只能继承一个 CRTP 基类 |
| 灵活扩展功能（Mixin）       | 基类与派生类耦合紧密         |

---

### **四、CRTP 与虚函数的对比**
| **特性**     | **CRTP**                   | **虚函数**                   |
| ------------ | -------------------------- | ---------------------------- |
| **多态类型** | 静态多态（编译时）         | 动态多态（运行时）           |
| **性能**     | 更高（无虚表查找）         | 较低（虚表查找开销）         |
| **内存开销** | 无虚表指针                 | 每个对象包含虚表指针         |
| **扩展性**   | 通过模板组合功能           | 通过继承层次结构扩展         |
| **适用场景** | 编译时确定类型，高性能需求 | 运行时类型不确定，需动态绑定 |

---

### **五、实际案例：数学库向量优化**
```cpp
template <typename Derived>
class VectorBase {
public:
    Derived operator+(const Derived& other) const {
        Derived result;
        for (size_t i = 0; i < static_cast<const Derived*>(this)->size(); ++i) {
            result[i] = (*static_cast<const Derived*>(this))[i] + other[i];
        }
        return result;
    }
};

class Vec3f : public VectorBase<Vec3f> {
public:
    float x, y, z;
    size_t size() const { return 3; }
    float operator[](size_t i) const {
        return (&x)[i];
    }
    float& operator[](size_t i) {
        return (&x)[i];
    }
};

int main() {
    Vec3f a{1, 2, 3}, b{4, 5, 6};
    Vec3f c = a + b; // c.x=5, c.y=7, c.z=9
}
```

---

### **六、注意事项**
1. **避免切割（Slicing）**：始终通过派生类对象操作 CRTP 基类。
2. **模板参数校验**：可使用 `static_assert` 确保派生类符合预期。
3. **生命周期管理**：CRTP 基类不管理派生类对象的生命周期。

---

**总结**：CRTP 是一种强大的编译时多态技术，适用于高性能、代码复用和接口强制等场景。合理使用可提升代码效率，但需权衡其带来的模板复杂性和耦合度。