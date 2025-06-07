# asyncio.ensure_future() 和 asyncio.gather()的区别

以下是对 loop.run_until_complete(asyncio.gather(foo(), bar())) 和 asyncio.ensure_future(foo()); asyncio.ensure_future(bar()); loop.run_forever() 两段代码在 Python asyncio 框架中区别的详细分析，基于您提供的上下文（foo 和 bar 是包含无限循环的异步协程）。

代码背景

假设您提供的 foo 和 bar 协程定义如下（基于之前的上下文）：

python

```python
async def foo():
    while True:
        print("foo")
        await asyncio.sleep(1)

async def bar():
    while True:
        print("bar")
        await asyncio.sleep(1)
```

这两个协程都包含无限 while True 循环，每秒分别打印 "foo" 和 "bar"，并通过 await asyncio.sleep(1) 暂停 1 秒。

分析两段代码的区别

1. 第一种方式：loop.run_until_complete(asyncio.gather(foo(), bar()))

**代码行为**：

- **asyncio.gather(foo(), bar())**：
  - gather 将多个协程（foo() 和 bar()）打包为一个单一的 Future 对象，表示这些协程的并发执行。
  - 它会等待所有指定的协程**完成**（即返回结果或抛出异常）后才返回。
  - 在这个场景中，foo 和 bar 都包含无限循环（while True），因此它们**永远不会完成**，除非被外部取消或抛出异常。
- **loop.run_until_complete(...)**：
  - 运行事件循环，直到指定的 Future（这里是 gather 创建的 Future）完成。
  - 因为 foo 和 bar 永远不完成，run_until_complete 会**无限阻塞**，事件循环会一直运行 foo 和 bar，每秒交替打印 "foo" 和 "bar"。
  - 后续代码（如果有）无法执行，因为 run_until_complete 不会返回。

**效果**：

- foo 和 bar 并发运行，每秒打印 "foo" 和 "bar"。
- 程序卡在 run_until_complete，直到外部中断（例如 Ctrl+C）。
- 适用于希望等待任务完成的情况，但不适合无限运行的协程，因为它会导致阻塞。

**局限性**：

- 由于 foo 和 bar 无限循环，run_until_complete 永远不会结束，后续代码不可达。
- 如果需要执行其他操作或调度更多任务，这种方式不灵活。
- 第二种方式：asyncio.ensure_future(foo()); asyncio.ensure_future(bar()); loop.run_forever()

**代码行为**：

- **asyncio.ensure_future(foo())**：
  - 将 foo() 协程包装为一个 Task（Task 是 Future 的子类），并将其调度到事件循环中运行。
  - 任务会立即开始执行（在事件循环运行时），但 ensure_future 本身不会等待任务完成。
- **asyncio.ensure_future(bar())**：
  - 类似地，将 bar() 调度为另一个独立的任务。
  - foo 和 bar 任务会并发运行。
- **loop.run_forever()**：
  - 启动事件循环并无限运行，直到显式停止（例如通过 loop.stop() 或 KeyboardInterrupt）。
  - 事件循环会持续处理所有调度任务（包括 foo 和 bar），每秒交替打印 "foo" 和 "bar"。
  - 这种方式允许任务无限运行，适合无限循环的协程。

**效果**：

- foo 和 bar 并发运行，每秒打印 "foo" 和 "bar"。
- 程序会无限运行，直到外部中断（例如 Ctrl+C）或显式调用 loop.stop()。
- 适用于需要无限运行多个并发任务的场景。

**优势**：

- 不等待任务完成，适合无限运行的协程。
- 可以在调度任务后继续添加更多任务（例如后续调用 ensure_future）。
- 更灵活，事件循环保持运行，允许动态管理任务。

**注意事项**：

- 需要显式处理循环停止和清理（例如在 try/finally 块中关闭循环）。
- 如果不处理中断，程序可能不会优雅退出。

详细对比

| 方面           | loop.run_until_complete(asyncio.gather(foo(), bar())) | asyncio.ensure_future(foo()); asyncio.ensure_future(bar()); loop.run_forever() |
| -------------- | ----------------------------------------------------- | ------------------------------------------------------------ |
| 执行方式       | 等待 foo 和 bar 完成（无限阻塞）                      | 调度任务后无限运行循环，不等待任务完成                       |
| 适合场景       | 等待有限任务完成                                      | 运行无限循环任务或需要持续运行的并发任务                     |
| 后续代码可达性 | 不可达（因无限循环阻塞）                              | 可达（ensure_future 不阻塞，可在调度后执行其他代码）         |
| 任务完成处理   | 等待所有任务完成                                      | 不关心任务是否完成，循环持续运行                             |
| 中断处理       | 需外部中断（如 Ctrl+C）                               | 需外部中断或显式停止循环（如 loop.stop()）                   |
| 灵活性         | 较低（阻塞在 gather）                                 | 较高（可动态调度任务，循环持续运行）                         |
| 行为           | 打印 "foo" 和 "bar" 直到中断                          | 打印 "foo" 和 "bar" 直到中断                                 |

示例：正确使用第二种方式

为了确保第二种方式优雅处理中断和清理，推荐以下代码：

python

```python
import asyncio

async def foo():
    while True:
        print("foo")
        await asyncio.sleep(1)

async def bar():
    while True:
        print("bar")
        await asyncio.sleep(1)

loop = asyncio.get_event_loop()
try:
    asyncio.ensure_future(foo())
    asyncio.ensure_future(bar())
    loop.run_forever()
except KeyboardInterrupt:
    print("Interrupted, stopping loop...")
    loop.stop()
    loop.run_until_complete(loop.shutdown_asyncgens())  # 清理异步生成器（如果有）
finally:
    loop.close()
```

**说明**：

- 使用 try/except 捕获 KeyboardInterrupt（Ctrl+C）。
- 在中断时调用 loop.stop() 停止循环。
- 使用 loop.run_until_complete(loop.shutdown_asyncgens()) 清理异步生成器（Python 3.6+）。
- 在 finally 中关闭循环以释放资源。

实际场景中的选择

- **使用 loop.run_until_complete(asyncio.gather(...))**：
  - 适合任务有明确结束条件（例如处理有限数据集、完成特定计算）。
  - 不适合无限循环任务，因为会导致阻塞。
- **使用 asyncio.ensure_future(...); loop.run_forever()**：
  - 适合需要无限运行的并发任务（例如服务器、监控任务、周期性任务）。
  - 更适合动态任务调度或长期运行的程序。

回答您的问题

**loop.run_until_complete(asyncio.gather(foo(), bar())) 和 asyncio.ensure_future(foo()); asyncio.ensure_future(bar()); loop.run_forever() 的区别**：

- **核心区别**：
  - 第一种方式通过 gather 并发运行 foo 和 bar，但等待它们完成。由于 foo 和 bar 是无限循环，程序会无限阻塞在 run_until_complete，后续代码不可达。
  - 第二种方式通过 ensure_future 调度 foo 和 bar 作为独立任务，并使用 run_forever 保持事件循环运行，不等待任务完成，适合无限运行的协程。
- **行为**：
  - 两者都会并发运行 foo 和 bar，每秒打印 "foo" 和 "bar"，直到中断。
  - 第一种方式因阻塞而无法执行后续代码，第二种方式更灵活，允许在调度任务后继续操作。
- **适用性**：
  - 第一种方式适合有限任务，第二种方式适合无限运行任务或动态任务管理。

**建议**： 对于 foo 和 bar 这样的无限循环协程，推荐使用第二种方式（ensure_future + run_forever），并添加适当的异常处理和清理代码（如上例所示），以确保程序优雅运行和退出。

如果您有进一步的问题或需要更具体的代码示例，请告诉我！





## 改为create_task()

```C++
import asyncio

async def foo():
    while True:
        print("foo")
        await asyncio.sleep(1)

async def bar():
    while True:
        print("bar")
        await asyncio.sleep(1)


loop = asyncio.get_event_loop()
try:
    # loop.run_until_complete(asyncio.gather(foo(), bar()))
    
    # asyncio.ensure_future(foo())
    # asyncio.ensure_future(bar())
    
    loop.create_task(foo())
    loop.create_task(bar())

    loop.run_forever()
    
except KeyboardInterrupt:
    pass
finally:
    loop.close()
    
```





您希望将代码中的 asyncio.ensure_future(foo()) 和 asyncio.ensure_future(bar()) 替换为 loop.create_task(foo()) 和 loop.create_task(bar())，并对比 loop.run_until_complete(asyncio.gather(foo(), bar())) 与使用 create_task 和 loop.run_forever() 的方式的区别。以下是改写后的代码和详细分析。

改写后的代码

将原始代码中的 asyncio.ensure_future 替换为 loop.create_task，并提供适合无限循环协程的正确实现：

python

```python
import asyncio

async def foo():
    while True:
        print("foo")
        await asyncio.sleep(1)

async def bar():
    while True:
        print("bar")
        await asyncio.sleep(1)

loop = asyncio.get_event_loop()
try:
    loop.create_task(foo())  # 替换 ensure_future
    loop.create_task(bar())  # 替换 ensure_future
    loop.run_forever()
except KeyboardInterrupt:
    print("Interrupted, stopping loop...")
    loop.stop()
    loop.run_until_complete(loop.shutdown_asyncgens())  # 清理异步生成器
finally:
    loop.close()
```

loop.create_task 与 asyncio.ensure_future 的区别

- **loop.create_task(coro)**：
  - 直接在指定的事件循环（loop）上创建并调度一个 Task 来运行协程 coro。
  - 仅接受协程对象，且任务绑定到指定的循环。
  - 返回一个 Task 对象。
  - 更明确地与特定事件循环关联，适合需要控制事件循环的场景。
- **asyncio.ensure_future(coro_or_future, \*, loop=None)**：
  - 将协程或 Future 对象转换为 Task 或 Future，并调度其运行。
  - 如果传入的是协程，会自动包装为 Task；如果是 Future，则直接返回。
  - 如果未指定 loop 参数（默认 None），它会尝试获取当前上下文的事件循环（通过 asyncio.get_event_loop()）。
  - 更通用，适合不需要显式指定事件循环的场景。
- **在您的代码中**：
  - loop.create_task(foo()) 和 asyncio.ensure_future(foo()) 在功能上等价，因为 foo() 是协程，且您已经显式定义了 loop。
  - 使用 loop.create_task 更直接，明确绑定到 loop，代码更清晰。
  - 性能差异微乎其微，但在需要确保任务绑定到特定循环时，create_task 是首选。

对比两种方式

1. 第一种方式：loop.run_until_complete(asyncio.gather(foo(), bar()))

**代码**：

python

```python
loop.run_until_complete(asyncio.gather(foo(), bar()))
```

**行为**：

- asyncio.gather(foo(), bar()) 将 foo() 和 bar() 打包为一个 Future，并发运行两者，并等待它们全部完成。
- loop.run_until_complete 运行事件循环，直到 gather 的 Future 完成。
- 由于 foo 和 bar 包含无限 while True 循环，它们永不完成，因此 run_until_complete 会**无限阻塞**，程序卡在此处。
- 每秒交替打印 "foo" 和 "bar"，直到外部中断（例如 Ctrl+C）。
- 后续代码（如果有）不可达。

**适用场景**：

- 适合等待有限任务完成（例如处理固定数量的数据）。
- 不适合无限循环协程，因为会导致程序阻塞。

**局限性**：

- 因无限循环而无法执行后续代码。
- 不灵活，无法动态添加任务。
- 第二种方式（改写后）：loop.create_task(foo()); loop.create_task(bar()); loop.run_forever()

**代码**：

python

```python
loop.create_task(foo())
loop

.create_task(bar())
loop.run_forever()
```

**行为**：

- loop.create_task(foo()) 和 loop.create_task(bar()) 将 foo 和 bar 协程调度为独立的任务，立即在事件循环中开始运行（当循环运行时）。
- loop.run_forever() 无限运行事件循环，持续处理所有调度任务。
- foo 和 bar 并发运行，每秒交替打印 "foo" 和 "bar"，直到外部中断（例如 Ctrl+C）或显式停止循环。
- 可以在调度任务后继续添加其他任务，事件循环保持运行。

**适用场景**：

- 适合无限运行的并发任务（例如服务器、周期性任务）。
- 适合需要动态调度任务或长期运行的程序。

**优势**：

- 不等待任务完成，适合无限循环协程。
- 更灵活，可在运行时添加新任务。
- 事件循环持续运行，适合实时任务管理。

**注意事项**：

- 需要显式处理中断和清理（如停止循环、关闭循环）。
- 推荐使用 try/except 和 finally 确保优雅退出（见改写代码）。

详细对比

| 方面           | loop.run_until_complete(asyncio.gather(foo(), bar())) | loop.create_task(foo()); loop.create_task(bar()); loop.run_forever() |
| -------------- | ----------------------------------------------------- | ------------------------------------------------------------ |
| 执行方式       | 等待 foo 和 bar 完成（无限阻塞）                      | 调度任务后无限运行循环，不等待任务完成                       |
| 任务调度       | 通过 gather 统一调度                                  | 通过 create_task 独立调度                                    |
| 适合场景       | 有限任务完成                                          | 无限循环任务或持续运行的并发任务                             |
| 后续代码可达性 | 不可达（阻塞在 gather）                               | 可达（create_task 不阻塞，可继续执行代码）                   |
| 任务完成处理   | 等待所有任务完成                                      | 不关心任务是否完成，循环持续运行                             |
| 中断处理       | 需外部中断（如 Ctrl+C）                               | 需外部中断或显式停止循环（如 loop.stop()）                   |
| 灵活性         | 较低（阻塞在 gather）                                 | 较高（可动态调度任务，循环持续运行）                         |
| 行为           | 打印 "foo" 和 "bar" 直到中断                          | 打印 "foo" 和 "bar" 直到中断                                 |

为什么选择改写后的方式

- **loop.run_until_complete(asyncio.gather(foo(), bar()))** 不适合 foo 和 bar 这样的无限循环协程，因为它会无限阻塞，导致程序无法执行后续逻辑。
- **loop.create_task(foo()); loop.create_task(bar()); loop.run_forever()** 更适合无限运行的任务，因为它不等待任务完成，允许事件循环持续运行，并支持动态任务管理。
- 使用 loop.create_task 替代 asyncio.ensure_future 使代码更明确，任务直接绑定到指定的事件循环，符合现代 asyncio 编程习惯。

正确实现的完整代码

改写后的代码已包含中断处理和清理，确保优雅运行和退出：

python

```python
import asyncio

async def foo():
    while True:
        print("foo")
        await asyncio.sleep(1)

async def bar():
    while True:
        print("bar")
        await asyncio.sleep(1)

loop = asyncio.get_event_loop()
try:
    loop.create_task(foo())
    loop.create_task(bar())
    loop.run_forever()
except KeyboardInterrupt:
    print("Interrupted, stopping loop...")
    loop.stop()
    loop.run_until_complete(loop.shutdown_asyncgens())
finally:
    loop.close()
```

**运行结果**：

- 每秒交替打印 "foo" 和 "bar"。
- 按 Ctrl+C 中断时，打印 "Interrupted, stopping loop..."，停止循环并清理。
- 事件循环正确关闭，程序优雅退出。

最终答案

**loop.run_until_complete(asyncio.gather(foo(), bar())) 与 loop.create_task(foo()); loop.create_task(bar()); loop.run_forever() 的区别**：

- **第一种方式**：通过 gather 并发运行 foo 和 bar，但等待它们完成。由于 foo 和 bar 是无限循环，程序无限阻塞在 run_until_complete，后续代码不可达。适合有限任务，不适合无限循环。
- **第二种方式**：通过 loop.create_task 调度 foo 和 bar 作为独立任务，使用 run_forever 保持事件循环运行，不等待任务完成。适合无限循环协程，灵活性更高，可动态管理任务。
- **改写说明**：将 asyncio.ensure_future 替换为 loop.create_task，功能等价但更明确，任务直接绑定到指定循环。推荐第二种方式（改写后代码）用于无限运行的协程，并搭配中断处理和循环清理。

如果您有其他问题或需要进一步优化代码，请告诉我！