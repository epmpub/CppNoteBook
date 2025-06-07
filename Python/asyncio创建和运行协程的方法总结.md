# asyncio创建和运行协程的方法总结

在 Python 的 asyncio 模块中，除了 ensure_future、gather 和 create_task 可以用来创建和运行协程（coroutine）外，还有一些其他方法和工具可以用来处理协程的创建和运行。以下是一些常见的方法和工具：

1. **asyncio.run**

- **作用**: 用于在程序的入口点运行一个协程，并等待其完成。适合作为异步程序的主入口。

- **特点**: 

  - 它会创建一个新的事件循环（event loop），运行协程，然后在协程完成后关闭事件循环。
  - 只能在程序的顶层调用，不能在已经运行的事件循环中调用。

- **示例**:

  python

  ```python
  import asyncio
  
  async def say_hello():
      print("Hello")
      await asyncio.sleep(1)
      print("World")
  
  asyncio.run(say_hello())
  ```

- **loop.run_until_complete**

- **作用**: 通过事件循环运行一个协程或 Future 对象，直到它完成。

- **特点**:

  - 需要手动获取或创建事件循环（asyncio.get_event_loop() 或 asyncio.new_event_loop()）。
  - 适合在需要显式管理事件循环的情况下使用。

- **示例**:

  python

  ```python
  import asyncio
  
  async def say_hello():
      print("Hello")
      await asyncio.sleep(1)
      print("World")
  
  loop = asyncio.get_event_loop()
  loop.run_until_complete(say_hello())
  ```

- **asyncio.create_task (补充说明)**

- 虽然你已经提到 create_task，但值得强调它直接将协程包装为 Task 并在当前事件循环中调度运行。

- **特点**: 

  - 返回一个 Task 对象，可以用来跟踪任务的状态或取消任务。
  - 需要在已有的事件循环中调用。

- **示例**:

  python

  ```python
  import asyncio
  
  async def say_hello():
      print("Hello")
      await asyncio.sleep(1)
      print("World")
  
  async def main():
      task = asyncio.create_task(say_hello())
      await task
  
  asyncio.run(main())
  ```

- **asyncio.ensure_future (补充说明)**

- 同样是你提到的方法，它可以将协程、Future 或其他可等待对象（awaitable）转换为 Future 对象。

- **特点**:

  - 如果传入的是协程，会自动将其包装为 Task。
  - 适合在需要兼容不同类型可等待对象时使用。

- **示例**:

  python

  ```python
  import asyncio
  
  async def say_hello():
      print("Hello")
      await asyncio.sleep(1)
      print("World")
  
  async def main():
      future = asyncio.ensure_future(say_hello())
      await future
  
  asyncio.run(main())
  ```

- **asyncio.gather (补充说明)**

- 你提到的 gather 用于并发运行多个协程，并收集它们的返回值。

- **特点**:

  - 可以控制是否在某个协程失败时取消其他协程（通过 return_exceptions 参数）。
  - 返回一个包含所有协程结果的列表。

- **示例**:

  python

  ```python
  import asyncio
  
  async def task1():
      await asyncio.sleep(1)
      return "Task 1"
  
  async def task2():
      await asyncio.sleep(2)
      return "Task 2"
  
  async def main():
      results = await asyncio.gather(task1(), task2())
      print(results)
  
  asyncio.run(main())
  ```

- **loop.create_task**

- **作用**: 直接通过事件循环对象（loop）创建任务。

- **特点**:

  - 与 asyncio.create_task 类似，但需要显式指定事件循环。
  - 适合在手动管理事件循环时使用。

- **示例**:

  python

  ```python
  import asyncio
  
  async def say_hello():
      print("Hello")
      await asyncio.sleep(1)
      print("World")
  
  loop = asyncio.get_event_loop()
  task = loop.create_task(say_hello())
  loop.run_until_complete(task)
  ```

- **loop.run_in_executor**

- **作用**: 在线程池或进程池中运行同步（阻塞）函数，并将其包装为协程。

- **特点**:

  - 适合将阻塞的 I/O 操作（如文件读写、网络请求）或 CPU 密集型任务移到单独的线程或进程中。
  - 返回一个 Future 对象。

- **示例**:

  python

  ```python
  import asyncio
  
  def blocking_task():
      import time
      time.sleep(1)
      return "Done"
  
  async def main():
      loop = asyncio.get_event_loop()
      result = await loop.run_in_executor(None, blocking_task)
      print(result)
  
  asyncio.run(main())
  ```

- **asyncio.wait**

- **作用**: 并发运行一组可等待对象（协程、任务或 Future），并等待它们完成。

- **特点**:

  - 提供更灵活的控制，可以设置超时或等待条件（如 FIRST_COMPLETED）。
  - 返回两个集合：已完成的任务和未完成的任务。

- **示例**:

  python

  ```python
  import asyncio
  
  async def task1():
      await asyncio.sleep(1)
      return "Task 1"
  
  async def task2():
      await asyncio.sleep(2)
      return "Task 2"
  
  async def main():
      tasks = [task1(), task2()]
      done, pending = await asyncio.wait(tasks, return_when=asyncio.FIRST_COMPLETED)
      for task in done:
          print(task.result())
  
  asyncio.run(main())
  ```

- **loop.call_soon 和 loop.call_later**

- **作用**:

  - call_soon: 立即在事件循环的下一次迭代中调用一个同步回调函数。
  - call_later: 在指定延迟后调用同步回调函数。

- **特点**:

  - 虽然它们主要用于同步函数，但可以通过包装协程来间接运行协程。
  - 适合需要延迟执行或调度简单任务的场景。

- **示例**:

  python

  ```python
  import asyncio
  
  async def say_hello():
      print("Hello")
      await asyncio.sleep(1)
      print("World")
  
  def schedule_coroutine(loop, coro):
      task = loop.create_task(coro)
      return task
  
  loop = asyncio.get_event_loop()
  loop.call_soon(schedule_coroutine, loop, say_hello())
  loop.run_until_complete(loop.shutdown_asyncgens())
  ```

- **asyncio.as_completed**

- **作用**: 以完成顺序迭代一组可等待对象。

- **特点**:

  - 适合需要按完成顺序处理协程结果的场景。
  - 返回一个迭代器，产生每个协程的结果。

- **示例**:

  python

  ```python
  import asyncio
  
  async def task1():
      await asyncio.sleep(1)
      return "Task 1"
  
  async def task2():
      await asyncio.sleep(2)
      return "Task 2"
  
  async def main():
      for coro in asyncio.as_completed([task1(), task2()]):
          result = await coro
          print(result)
  
  asyncio.run(main())
  ```

- **asyncio.to_thread** (Python 3.9+)

- **作用**: 在单独的线程中运行同步函数，并将其包装为协程。

- **特点**:

  - 简化了 loop.run_in_executor 的使用，专门用于线程执行。
  - 适合运行阻塞的同步代码。

- **示例**:

  python

  ```python
  import asyncio
  
  def blocking_task():
      import time
      time.sleep(1)
      return "Done"
  
  async def main():
      result = await asyncio.to_thread(blocking_task)
      print(result)
  
  asyncio.run(main())
  ```

- **loop.create_future**

- **作用**: 创建一个 Future 对象，可以手动设置其结果或异常。

- **特点**:

  - 通常与手动调度结合使用，适合需要自定义 Future 行为的场景。
  - 可以通过事件循环调度协程来间接运行。

- **示例**:

  python

  ```python
  import asyncio
  
  async def say_hello():
      print("Hello")
      await asyncio.sleep(1)
      return "World"
  
  loop = asyncio.get_event_loop()
  future = loop.create_future()
  
  async def resolve_future():
      result = await say_hello()
      future.set_result(result)
  
  loop.create_task(resolve_future())
  loop.run_until_complete(future)
  print(future.result())
  ```

总结

除了 ensure_future、gather 和 create_task，以下是其他创建或运行协程的主要方法：

- **顶层运行**: asyncio.run
- **事件循环管理**: loop.run_until_complete, loop.create_task
- **并发控制**: asyncio.wait, asyncio.as_completed
- **同步代码集成**: loop.run_in_executor, asyncio.to_thread
- **延迟调度**: loop.call_soon, loop.call_later
- **自定义 Future**: loop.create_future

根据具体场景（如是否需要并发、是否需要手动管理事件循环、是否涉及阻塞代码等），你可以选择合适的方法来创建和运行协程。如果需要更复杂的异步逻辑，可以组合使用这些方法。

如果你有具体场景或代码示例需要进一步分析，可以提供更多细节，我可以帮你优化或解释！