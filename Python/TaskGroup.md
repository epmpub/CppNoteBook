# TaskGroup



![image-20250518085210252](C:\Users\sheng\AppData\Roaming\Typora\typora-user-images\image-20250518085210252.png)



```python
async def main():
    async with asyncio.TaskGroup() as tg:
        task1 = tg.create_task(some_coro(...))
        task2 = tg.create_task(another_coro(...))
    print(f"Both tasks have completed now: {task1.result()}, {task2.result()}")
            
```

