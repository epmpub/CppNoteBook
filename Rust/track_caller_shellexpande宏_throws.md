# track_caller

`#[track_caller]` 是 Rust 的一个属性，用于追踪函数的调用者位置。

作用

- 让 `std::panic::Location::caller()` 返回调用这个函数的位置，而不是函数内部的位置
- 用于调试，看到是哪个文件、哪一行代码调用了这个函数

例子

```
#[track_caller]
fn my_function() {
    let location = std::panic::Location::caller();
    println!("被调用于：{}:{}", location.file(), location.line());
}

fn main() {
    my_function(); // 会显示这一行的位置
}
```

通过添加 `#[track_caller]` ，我们能看到：

- PacketReceiver 调用 PacketDeserializer → 显示 packet_receiver.rs:某行
- 其他地方 直接调用 PacketDeserializer → 显示真正的调用位置

这样我们就能找到是谁绕过了 PacketReceiver，直接调用了 PacketDeserializer

# shellexpand

`shellexpand::full` 是 Rust 中 shellexpand crate 的一个函数，用于展开 shell 路径表达式。

作用：

- 展开 ~ (用户主目录)
- 展开环境变量 (如 USER)
- 处理相对路径和绝对路径

示例：

```
use shellexpand;

fn main() {
    // 展开用户主目录
    let expanded = shellexpand::full("~/Documents/file.txt").unwrap();
    // 结果："/Users/username/Documents/file.txt"

    // 展开环境变量
    let expanded = shellexpand::full("$HOME/config").unwrap();
    // 结果："/Users/username/config"

    //   你的代码中没有使用这个函数，如果需要处理用户输入的路径（支持 ~ 和环境变量），可以这样添加：

    let expanded_path = shellexpand::full(filepath).expect("路径展开失败");
    let file = File::open(expanded_path.as_ref()).expect("无法打开文件");
}
```

# #[throws(Error)]

`#[throws(Error)]` 是 Rust 中的一个属性宏，用于简化错误处理。

作用：

- 自动为函数添加 Result 返回类型
- 允许使用 ? 操作符传播错误
- 简化函数签名，无需手动写 -> Result<_, _>

## 对比

传统写法：

```
fn read_file(path: &str) -> Result<String, std::io::Error> {
    let mut file = File::open(path)?;
    let mut contents = String::new();
    file.read_to_string(&mut contents)?;
    Ok(contents)
}
```

使用 #[throws]

```
#[throws(std::io::Error)]
fn read_file(path: &str) -> String {
    let mut file = File::open(path)?;
    let mut contents = String::new();
    file.read_to_string(&mut contents)?;
    contents // 自动包装为 Ok(contents)
}
```

注意：

- 需要 fehler crate 或类似库
- 函数最后的表达式自动包装为 Ok(value)
- 仍可使用 ? 进行错误传播

这个属性让错误处理代码更简洁易读。

```rust
// #[track_caller]
// fn my_function() {
//     let location = std::panic::Location::caller();
//     println!("被调用于：{}:{}", location.file(), location.line());
// }

// fn main() {
//     my_function(); // 会显示这一行的位置
//     println!("next line");
//     my_function(); // 会显示这一行的位置

// }

use std::fs::File;
use shellexpand;
use std::io::{self, BufRead};
use std::io::Read;
use fehler::throws; 

#[throws(std::io::Error)]
fn read_file(path: &str) -> String {
    let mut file = File::open(path)?;
    let mut contents = String::new();
    file.read_to_string(&mut contents)?;
    contents // 自动包装为 Ok(contents)
}


fn main() {
    // 展开用户主目录
    // let expanded = shellexpand::full("~/Documents/file.txt").unwrap();
    // 结果："/Users/username/Documents/file.txt"

    // 展开环境变量
    // let expanded = shellexpand::full("$HOME/config").unwrap();
    // 结果："/Users/username/config"

    // 你的代码中没有使用这个函数，如果需要处理用户输入的路径（支持 ~ 和环境变量），可以这样添加：

    // let filepath = "~/Documents/file.txt";

    let expanded_path = shellexpand::full("~/Documents/file.txt").expect("路径展开失败");
    let file = File::open(expanded_path.as_ref()).expect("无法打开文件");

    let reader = io::BufReader::new(file);
    for line in reader.lines() {
        println!("{}", line.expect("读取行失败"));
    }

    match read_file(expanded_path.as_ref()) {
        Ok(contents) => println!("文件内容: {}", contents),
        Err(e) => eprintln!("错误: {}", e),
    }
}

```

