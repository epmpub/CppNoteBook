# Once 和 OnceLock

在 Rust 中，std::sync::Once 和 std::sync::OnceLock 是标准库提供的两种用于一次性初始化的同步原语，适用于需要在多线程环境中安全地初始化共享资源的场景。它们功能上类似，但设计和使用方式有显著差异。以下是两者的详细对比，以及它们在功能和适用场景上的区别。1. 核心区别

| 特性       | std::sync::Once                | std::sync::OnceLock                            |
| ---------- | ------------------------------ | ---------------------------------------------- |
| 引入版本   | Rust 1.0（旧版同步原语）       | Rust 1.70（2023年2月，较新）                   |
| 初始化方式 | 通过回调函数（call_once）      | 直接存储值（get_or_init 或 set）               |
| 返回值     | 无返回值，仅执行初始化         | 返回引用（&T），便于访问初始化后的值           |
| 状态检查   | is_completed 检查是否初始化    | get 检查是否初始化并获取引用                   |
| 灵活性     | 较为受限，适合简单初始化       | 更灵活，支持初始化后直接访问值                 |
| 错误处理   | 支持 call_once 中的 panic 捕获 | 支持 get_or_init 中的 panic，set 可返回 Result |
| 性能       | 轻量，适合简单场景             | 略高开销，但更方便使用                         |
| 典型场景   | 全局单例初始化                 | 延迟初始化并需要频繁访问值的场景               |

2. 详细说明std::sync::Once

- 定义：Once 是一个低级同步原语，用于确保某个初始化操作在多线程环境中只执行一次。初始化通过 call_once 方法提供一个闭包来执行。
- 特点：
  - 初始化逻辑通过闭包提供，call_once 确保闭包只执行一次。
  - 其他线程调用 call_once 时会阻塞，直到初始化完成。
  - 不直接存储初始化的值，调用者需自行管理结果（通常通过外部静态变量）。
  - 如果初始化闭包发生 panic，Once 会被标记为“中毒”（poisoned），后续调用会失败。
- 适用场景：
  - 简单的全局初始化，例如设置全局配置或日志系统。
  - 不需要频繁访问初始化结果的场景。
- 局限性：
  - 不提供对初始化值的直接访问，需配合其他机制（如静态变量）存储结果。
  - API 较为低级，使用不够直观。

示例：

rust



```rust
use std::sync::Once;

static mut GLOBAL_CONFIG: Option<&str> = None;
static INIT: Once = Once::new();

fn get_config() -> &'static str {
    INIT.call_once(|| {
        // 模拟复杂初始化
        unsafe {
            GLOBAL_CONFIG = Some("Initialized Config");
        }
    });
    unsafe { GLOBAL_CONFIG.unwrap() }
}

fn main() {
    let threads: Vec<_> = (0..3)
        .map(|i| std::thread::spawn(move || {
            println!("Thread {}: Config = {}", i, get_config());
        }))
        .collect();

    for t in threads {
        t.join().unwrap();
    }
}
```

- 输出（可能顺序不同）：

  

  

  ```text
  Thread 0: Config = Initialized Config
  Thread 1: Config = Initialized Config
  Thread 2: Config = Initialized Config
  ```

- 说明：

  - Once 确保初始化只发生一次。
  - GLOBAL_CONFIG 存储初始化结果，需使用 unsafe 管理。

std::sync::OnceLock

- 定义：OnceLock 是 Rust 1.70 引入的更现代的同步原语，用于线程安全的延迟初始化。它直接存储初始化后的值，并提供对值的引用访问。
- 特点：
  - 通过 get_or_init（接受闭包）或 set（直接设置值）进行初始化。
  - 初始化后可以通过 get 获取值的引用（&T），无需额外存储。
  - 支持 into_inner 提取内部值（销毁 OnceLock）。
  - 如果初始化闭包 panic，OnceLock 不会被标记为中毒，后续可以重试初始化。
  - 更符合 Rust 的现代设计，API 更直观。
- 适用场景：
  - 需要延迟初始化并频繁访问结果的场景。
  - 希望避免 unsafe 或复杂状态管理的场景。
- 局限性：
  - 相比 Once，有轻微内存和性能开销（因存储值）。
  - Rust 1.70 之前的版本不可用。

示例：

rust



```rust
use std::sync::OnceLock;

static CONFIG: OnceLock<&str> = OnceLock::new();

fn get_config() -> &'static str {
    CONFIG.get_or_init(|| {
        // 模拟复杂初始化
        "Initialized Config"
    })
}

fn main() {
    let threads: Vec<_> = (0..3)
        .map(|i| std::thread::spawn(move || {
            println!("Thread {}: Config = {}", i, get_config());
        }))
        .collect();

    for t in threads {
        t.join().unwrap();
    }
}
```

- 输出（可能顺序不同）：

  

  

  ```text
  Thread 0: Config = Initialized Config
  Thread 1: Config = Initialized Config
  Thread 2: Config = Initialized Config
  ```

- 说明：

  - OnceLock 直接存储初始化值，get_or_init 返回 &str。
  - 无需 unsafe，API 更简洁安全。

- 与 C++ 的对比如果你熟悉 C++，可以类比：

- std::sync::Once 类似于 C++11 的 std::call_once 结合 std::once_flag，用于确保初始化只执行一次，但需要外部存储结果。

  cpp

  

  ```cpp
  #include <mutex>
  #include <iostream>
  #include <thread>
  
  std::once_flag flag;
  std::string config;
  
  void init_config() {
      std::call_once(flag, []() {
          config = "Initialized Config";
      });
  }
  
  int main() {
      std::thread t1([] { init_config(); std::cout << config << std::endl; });
      std::thread t2([] { init_config(); std::cout << config << std::endl; });
      t1.join();
      t2.join();
      return 0;
  }
  ```

- std::sync::OnceLock 没有直接的 C++ 标准库等价物，但可以通过自定义类（结合 std::mutex 和 std::optional）模拟，或者使用第三方库（如 absl::once_flag 或 Boost）。OnceLock 的设计更接近现代 C++ 的封装理念。

- 性能对比

- Once：
  - 更轻量，仅管理初始化状态，不存储值。
  - 适合简单初始化，性能开销极低。
- OnceLock：
  - 存储初始化值，增加少量内存开销。
  - 提供更便捷的 API（如 get 返回引用），适合频繁访问值的场景。
- 如何选择

- 用 Once：
  - 需要最低开销的简单初始化。
  - 不介意使用 unsafe 或外部变量存储结果。
  - 在旧版本 Rust（< 1.70）中工作。
- 用 OnceLock：
  - 需要直接访问初始化后的值（通过 &T）。
  - 希望更现代、简洁的 API，避免 unsafe。
  - 在 Rust 1.70 或更高版本中工作。
  - 适用于延迟初始化并频繁访问的场景。
- 注意事项

- Once：
  - 如果初始化闭包 panic，后续调用会失败（中毒）。
  - 需要手动管理初始化结果的存储，常用 static mut（需 unsafe）。
- OnceLock：
  - 初始化闭包 panic 不会阻止后续重试，set 方法返回 Result 可处理错误。
  - 线程安全，适合静态变量或多线程共享。
  - 不支持动态释放（除非用 into_inner 销毁）。
- 线程安全：两者都线程安全，适合多线程环境。
- Rust 版本：OnceLock 要求 Rust 1.70+，检查项目兼容性。
- 总结

- Once：低级、轻量，适合简单一次性初始化，需外部存储结果，API 较老旧。
- OnceLock：现代、便捷，直接存储值并返回引用，适合频繁访问和更安全的使用场景。
- 功能差异：Once 仅保证初始化执行一次，OnceLock 还管理初始化结果，API 更符合现代 Rust 设计。

如果你有具体场景（例如初始化复杂数据结构或与 C++ 交互），可以提供更多细节，我可以帮你优化代码或进一步对比！