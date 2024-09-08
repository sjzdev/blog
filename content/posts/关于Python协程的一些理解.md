---
blog: true
tags: 
categories: 
collections: 
title: 关于Python协程的一些理解
date: 2024-08-27 21:05:56
update: 2024-09-08 13:11:21
---

事件循环可以简单粗暴的理解成如下代码展示的那样，不断地从一个任务列表中获取任务，然后判断任务是否完成，如果完成就从任务列表中移除，如果未完成则继续运行。

```python
# 任务列表  
task_list = []  

# 任务列表不为空，则一直运行循环  
while len(task_list) > 0:  

    # 遍历任务列表  
    for task in task_list:  
    
        # 任务已完成，则从任务列表中移除  
        if task.completed():  
            task_list.remove(task)  
            
        # 任务未完成，则运行任务  
        else:  
            task.run()
```

所以一个线程同一时刻只能有一个事件循环在运行，同时一个事件循环中同一时刻也只会有一个任务在运行。

下面的代码是一个简单的小实验：

```python
import threading
import asyncio

# 异步函数 task_a，展示如何使用不同的异步任务和线程池执行任务
async def task_a():
    # 获取当前线程信息，并打印
    task_a_thread = threading.current_thread()
    print("task_a thread is:", task_a_thread)
    # 获取当前异步任务信息，并打印
    print("task_a task is:", asyncio.current_task())
    # 使用异步任务方式执行 task_b
    await asyncio.create_task(task_b())
    # 使用线程池方式执行 task_c
    await asyncio.to_thread(task_c)

# 异步函数 task_b，展示在 task_a 中以异步任务形式运行的函数
async def task_b():
    # 获取当前线程信息，并打印
    task_b_thread = threading.current_thread()
    print("task_b thread is:", task_b_thread)
    # 获取当前异步任务信息，并打印
    print("task_b task is:", asyncio.current_task())

# 同步函数 task_c，展示在 task_a 中以线程池任务形式运行的函数
def task_c():
    # 获取当前线程信息，并打印
    task_b_thread = threading.current_thread()
    print("task_c thread is:", task_b_thread)
    # 直接运行 task_d，展示如何在同步函数中启动异步任务
    asyncio.run(task_d())

# 异步函数 task_d，展示由 task_c 启动的异步任务
async def task_d():
    # 获取当前线程信息，并打印
    task_d_thread = threading.current_thread()
    print("task_d thread is:", task_d_thread)
    # 获取当前异步任务信息，并打印
    print("task_d task is:", asyncio.current_task())

# 程序入口
if __name__ == "__main__":
    # 获取并打印主线程信息
    main_thread = threading.current_thread()
    print("main_thread is:", main_thread)
    # 运行 task_a，展示如何在事件循环中执行异步任务
    asyncio.run(task_a())
```

```python
main_thread is: <_MainThread(MainThread, started 27020)>
task_a thread is: <_MainThread(MainThread, started 27020)>
task_a task is: <Task pending name='Task-1' coro=<task_a()
task_b thread is: <_MainThread(MainThread, started 27020)>
task_b task is: <Task pending name='Task-2' coro=<task_b()
task_c thread is: <Thread(asyncio_0, started 27044)>
task_d thread is: <Thread(asyncio_0, started 27044)>
task_d task is: <Task pending name='Task-3' coro=<task_d()
```

在主线程中通过 `asyncio.run(task_a())` 启动事件循环执行 `task_a`，根据输出结果可以看到执行 `task_a` 的线程就是主线程；`task_a` 创建了一个新任务 `task_b`，此时两者在同一个事件循环中，所以执行它们的线程是同一个；`task_a` 中通过 `asyncio.to_thread(task_c)` 创建了一个任务 `task_c`，它是被放到另一个线程中执行；`task_c` 启动了 `task_d`，所以这两个是在同一个线程中。