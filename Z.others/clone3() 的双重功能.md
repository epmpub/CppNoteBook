### clone3() 的双重功能

是的，你说得完全正确！`clone3()` 系统调用既可以创建线程，也可以创建进程，这完全取决于传递给它的 `flags` 参数。

### 创建线程时的 flags

从你的 strace 输出中可以看到创建线程时使用的 flags：

```
flags=CLONE_VM|CLONE_FS|CLONE_FILES|CLONE_SIGHAND|CLONE_THREAD|CLONE_SYSVSEM|CLONE_SETTLS|CLONE_PARENT_SETTID|CLONE_CHILD_CLEARTID
```

这些 flags 的含义：

- `CLONE_VM` - 共享内存空间
- `CLONE_FS` - 共享文件系统信息
- `CLONE_FILES` - 共享文件描述符表
- `CLONE_SIGHAND` - 共享信号处理
- `CLONE_THREAD` - 创建线程而非进程
- `CLONE_SYSVSEM` - 共享 System V 信号量
- `CLONE_SETTLS` - 设置线程本地存储

### 创建进程时的 flags

如果要创建进程，flags 会很不同，比如：

```
flags=CLONE_CHILD_CLEARTID|CLONE_CHILD_SETTID
```

或者根本不使用共享内存的 flags。

## Linux 内核的统一模型

Linux 内核实际上把**线程和进程都视为"任务"（task）**：

- **进程** = 独立内存空间的任务
- **线程** = 共享内存空间的任务

`clone3()` 是这个统一模型的体现，通过不同的 flags 组合：

- 创建完全独立的进程（类似 `fork()`）
- 创建共享资源的线程（类似 `pthread_create()`）
- 创建各种中间状态的任务

## 与其他系统调用的关系

```
fork() ≈ clone3() with minimal flags
pthread_create() ≈ clone3() with thread flags
vfork() ≈ clone3() with CLONE_VFORK
```

所以 `clone3()` 是 Linux 内核提供的最灵活、最底层的任务创建接口，上层的各种创建函数本质上都是对它的封装。