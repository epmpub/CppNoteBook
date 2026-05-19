## Rust 支持的 OOP 特性

**1. 封装 ✅ 完全支持**

```rust
pub struct BankAccount {
    balance: f64,  // 私有字段
}

impl BankAccount {
    pub fn new(initial_balance: f64) -> Self {
        BankAccount { balance: initial_balance }
    }
    
    pub fn deposit(&mut self, amount: f64) {  // 公共方法
        if amount > 0.0 {
            self.balance += amount;
        }
    }
    
    pub fn get_balance(&self) -> f64 {  // 受控访问
        self.balance
    }
    
    fn validate_transaction(&self) -> bool {  // 私有方法
        self.balance >= 0.0
    }
}
```

**2. 多态 ✅ 通过 trait 对象实现**

```rust
trait Animal {
    fn make_sound(&self);
    fn move_around(&self);
}

struct Dog { name: String }
struct Cat { name: String }

impl Animal for Dog {
    fn make_sound(&self) { println!("{} barks!", self.name); }
    fn move_around(&self) { println!("{} runs!", self.name); }
}

impl Animal for Cat {
    fn make_sound(&self) { println!("{} meows!", self.name); }
    fn move_around(&self) { println!("{} prowls!", self.name); }
}

// 运行时多态
fn animal_behavior(animals: Vec<Box<dyn Animal>>) {
    for animal in animals {
        animal.make_sound();  // 动态分发
        animal.move_around();
    }
}
```

## Rust 不直接支持的传统 OOP 特性

**1. 类继承 ❌ 不支持**

```rust
// 这在 Rust 中不存在
// class Vehicle { ... }
// class Car extends Vehicle { ... }

// 但可以通过组合实现类似效果
struct Vehicle {
    speed: f32,
    fuel: f32,
}

struct Car {
    vehicle_parts: Vehicle,  // 组合而非继承
    doors: u8,
}

impl Car {
    fn accelerate(&mut self) {
        self.vehicle_parts.speed += 10.0;
    }
}
```

**2. 方法重写 ❌ 传统意义上不支持**

```rust
// 不能像传统 OOP 那样重写父类方法
// 但可以通过 trait 的默认实现来模拟

trait Drawable {
    fn draw(&self) {
        println!("Drawing default shape");  // 默认实现
    }
    
    fn area(&self) -> f64;  // 必须实现
}

struct Circle { radius: f64 }

impl Drawable for Circle {
    fn draw(&self) {  // "重写"默认实现
        println!("Drawing a circle with radius {}", self.radius);
    }
    
    fn area(&self) -> f64 {
        3.14159 * self.radius * self.radius
    }
}
```

## Rust 的独特 OOP 方式

**1. Trait 系统 - 比继承更强大**

```rust
// 多个 trait 可以同时实现
trait Flyable { fn fly(&self); }
trait Swimmable { fn swim(&self); }

struct Duck;

impl Flyable for Duck {
    fn fly(&self) { println!("Duck flies!"); }
}

impl Swimmable for Duck {
    fn swim(&self) { println!("Duck swims!"); }
}

// 泛型约束支持多重"继承"
fn multi_talented<T: Flyable + Swimmable>(creature: T) {
    creature.fly();
    creature.swim();
}
```

**2. 枚举 + 模式匹配 - 代数数据类型**

```rust
enum Shape {
    Circle { radius: f64 },
    Rectangle { width: f64, height: f64 },
    Triangle { base: f64, height: f64 },
}

impl Shape {
    fn area(&self) -> f64 {
        match self {
            Shape::Circle { radius } => 3.14159 * radius * radius,
            Shape::Rectangle { width, height } => width * height,
            Shape::Triangle { base, height } => 0.5 * base * height,
        }
    }
}

// 这种方式比继承更类型安全
```

**3. 所有权驱动的设计**

```rust
struct Database {
    connection: String,
}

struct UserRepository {
    db: Database,  // 拥有所有权
}

struct OrderRepository {
    // 不能同时拥有同一个 Database
    // 必须通过借用或 Rc/Arc 共享
}

// 这强制了更好的架构设计
```

## 总结

Rust **不能完全支持传统面向对象思想**，但提供了：

**支持的方面：**

- 封装（完全支持）
- 多态（通过 trait 对象）
- 抽象（通过 trait）
- 模块化设计

**不支持/替代的方面：**

- 类继承 → 组合 + trait
- 方法重写 → trait 默认实现
- 动态类型 → 静态类型系统
- 垃圾回收 → 所有权系统

**Rust 的哲学是：** 组合优于继承，显式优于隐式，编译时安全优于运行时检查。这使得 Rust 程序更安全、更高效，但需要开发者转变传统的 OOP 思维方式。

如果你习惯了传统 OOP，可能需要一段时间适应 Rust 的方式，但这种方式往往能写出更健壮的代码。