# Rust调用C库

调用自定义 C 库假设有一个 C 库 mylib.c：

```c
#include <stdio.h>
void say_hello(const char* name) {
    printf("Hello, %s!\n", name);
}
```

编译为静态库：

```bash
gcc -c mylib.c -o mylib.o -fPIE
ar rcs libmylib.a mylib.o 
# 打包为静态库

# 可执行文件的 .text 段（代码段）包含 say_hello 的机器码。
# 使用 nm 或 objdump 检查可执行文件，可以看到 say_hello 的符号和代码：bash

nm target/debug/my_project | grep say_hello

# 输出类似：

0000000000401000 T say_hello

# 表示 say_hello 的机器码已嵌入


```

Rust 代码调用：

```rust
use std::ffi::CString;

extern "C" {
    fn say_hello(name: *const c_char);
}

fn main() {
    let name = CString::new("Rust").expect("CString::new failed");
    unsafe {
        say_hello(name.as_ptr());
    }
}
```

在 Cargo.toml 中链接 C 库：

toml

```toml
[dependencies]
libc = "0.2"

[build-dependencies]
cc = "1.0"

[profile.release]
lto = true
```

创建 build.rs 链接静态库：

rust

```rust
fn main() {
    println!("cargo:rustc-link-lib=static=mylib");
    println!("cargo:rustc-link-search=native=/lib");
}
```



或者；

```Rust
use std::ffi::CString;
use libc::c_char;

// Tell Rust to look for libraries in the "lib" directory
#[link(name = "mylib", kind = "dylib")]
extern "C" {
    fn say_hello(name: *const c_char);
}

fn main() {
    let name = CString::new("Rust").expect("CString::new failed");
    unsafe {
        say_hello(name.as_ptr());
    }
}
```

```bash
# BASH
LD_LIBRARY_PATH=./lib cargo run
```

- Make sure your `libmylib.so` (Linux shared library) is in the [lib](vscode-file://vscode-app/c:/Program Files/VSCode/resources/app/out/vs/code/electron-browser/workbench/workbench.html) directory.
- If your library is static (`libmylib.a`), change `kind = "dylib"` to `kind = "static"`.
- 