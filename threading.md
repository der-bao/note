# Python `threading` 模块学习文档

`threading` 是 Python 标准库中用于处理**多线程（Multi-threading）**的核心模块。它允许你在同一个进程中同时执行多个任务。

---

## 1. 核心概念

* **进程 (Process)**: 操作系统分配资源（如内存）的最小单位。
* **线程 (Thread)**: 进程内的执行单元，是 CPU 调度的最小单位。同一个进程下的多个线程共享内存（变量等）。
* **GIL (全局解释器锁)**: CPython 解释器的特性。它保证同一时刻只有一个线程在执行 Python 字节码，这意味着 Python 的多线程在计算密集型任务中**不能**实现真正的并行。
* **适用场景**: `threading` 最适合处理**I/O密集型任务**（如网络请求、文件读写、数据库查询），不适合计算密集型任务（建议用 `multiprocessing`）。

---

## 2. 基础用法

### 2.1 创建和启动线程

最直接的方式是实例化 `threading.Thread` 并传入目标函数。

```python
import threading
import time

def worker(name, delay):
    print(f"线程 {name} 开始执行")
    time.sleep(delay)
    print(f"线程 {name} 执行结束")

# 1. 创建线程
t1 = threading.Thread(target=worker, args=("A", 2))
t2 = threading.Thread(target=worker, args=("B", 1))

# 2. 启动线程
t1.start()
t2.start()

# 3. 等待线程结束（阻塞主线程）
t1.join()
t2.join()

print("主线程结束")
```

### 2.2 继承 Thread 类

适合更复杂的线程逻辑封装。

```python
import threading
import time

class MyThread(threading.Thread):
    def __init__(self, name, delay):
        super().__init__()  # 必须调用父类初始化
        self.name = name
        self.delay = delay
  
    def run(self):
        # start() 方法被调用时，会自动执行 run() 中的代码
        print(f"[{self.name}] start")
        time.sleep(self.delay)
        print(f"[{self.name}] finish")

t = MyThread("CustomThread", 1)
t.start()
t.join()
```

---

## 3. 守护线程 (Daemon Thread)

守护线程是在后台运行的支撑性线程。**如果主线程和其他非守护线程结束了，进程会直接退出，不管守护线程有没有执行完。**

```python
import threading
import time

def background_task():
    while True:
        print("后台运行中...")
        time.sleep(1)

# 设置为守护线程
t = threading.Thread(target=background_task, daemon=True)
# t.setDaemon(True) # 老写法
t.start()

time.sleep(3)
print("主线程即将退出，守护线程也会立刻被终止。")
```

---

## 4. 线程同步与锁 (Lock / RLock)

因为多线程共享全局变量，如果两个线程同时修改同一个变量，会导致**数据竞争（Data Race）**。必须加锁保护。

### 4.1 没有锁的情况（数据竞争）

如果不使用锁，两个线程同时修改同一个变量（非原子操作，如 `+= 1`），会导致最终结果错误。

```python
import threading

counter = 0

def increment_without_lock():
    global counter
    for _ in range(100000):
        counter += 1

threads = [threading.Thread(target=increment_without_lock) for _ in range(5)]
for t in threads: t.start()
for t in threads: t.join()

print(f"没有锁的最终结果: {counter} (大概率不是 500000)")
```

### 4.2 互斥锁 (Lock)

```python
import threading

counter = 0
lock = threading.Lock() # 创建锁

def increment():
    global counter
    for _ in range(100000):
        # 方式 1: 手动获取和释放
        # lock.acquire()
        # counter += 1
        # lock.release()

        # 方式 2: 使用 context manager (推荐), 自动获取和释放，即使抛出异常也能保证释放
        with lock:
            counter += 1

threads = [threading.Thread(target=increment) for _ in range(5)]
for t in threads: t.start()
for t in threads: t.join()

print(f"最终结果: {counter} (预期是 500000)")
```

### 4.3 可重入锁 (RLock)

如果一个线程在没有释放锁的情况下，试图再次获取**同一把锁**，普通的 `Lock` 会导致**死锁 (Deadlock)**。
`threading.RLock()` 允许同一个线程多次获取同一把锁（必须 `release` 相同的次数，采用with语句 ）。

### 4.4 线程局部存储 (threading.local)

`threading.local()` 用于在多线程环境中为每个线程创建一份独立的数据副本。即使多个线程由于共享全局变量而访问的是同一个 `local` 对象，它们也只能看到并修改自己绑定的那份数据，从而巧妙地避免了数据冲突，也免去了繁琐的加锁过程。这种方式在很多场景下比加锁更优雅高效。

**典型应用场景：**

* **数据库连接池**：为每个线程分配独立专属的连接对象，避免跨线程乱用连接。
* **Web 框架中的会话管理**（如 Flask 的 `request`）：确保每个请求（由独立线程处理）拥有独立的用户状态数据，不互相干扰。

```python
import threading

# 创建一个线程本地数据对象
local_data = threading.local()

def task(value):
    # 给当前线程绑定一个独立的 name 属性
    local_data.value = value
  
    # 每个线程只能拿到自己设置的值
    print(f"线程 {threading.current_thread().name} 的值：{local_data.value}")

# 创建 3 个线程
t1 = threading.Thread(target=task, args=("线程1",), name="T1")
t2 = threading.Thread(target=task, args=("线程2",), name="T2")
t3 = threading.Thread(target=task, args=("线程3",), name="T3")

t1.start()
t2.start()
t3.start()
```

---

## 5. 高级同步原语

### 5.1 事件 (Event)

用于线程间的通信和状态同步。一个线程发出信号，其他等待的线程开始执行。

```python
import threading
import time

# 1. 创建Event对象（默认状态为False）
event = threading.Event()

# 定义子线程任务（等待信号触发）
def task(name):
    print(f"子线程 {name}：等待启动信号...")
    event.wait()  # 阻塞等待，直到信号被触发
    print(f"子线程 {name}：收到信号，开始执行任务")
    time.sleep(1)
    print(f"子线程 {name}：任务执行完毕")

# 创建3个子线程
threads = []
for i in range(3):
    t = threading.Thread(target=task, args=(f"thread-{i}",))
    t.start()
    threads.append(t)

# 主线程延迟2秒，模拟准备工作
time.sleep(2)
print("主线程：准备完毕，发送启动信号！")
event.set()  # 触发信号，唤醒所有等待的子线程

# 等待所有子线程完成
for t in threads:
    t.join()
print("所有子线程任务完成")
```

### 5.2 信号量 (Semaphore)

控制同时访问某个公共资源的线程最大数量。（类似于停车场的车位闸机）

```python
import threading
import time

# 只允许 3 个线程同时执行
semaphore = threading.Semaphore(3)

def access_resource(id):
    with semaphore:
        print(f"线程 {id} 获得了资源")
        time.sleep(2)
        print(f"线程 {id} 释放了资源")

threads = [threading.Thread(target=access_resource, args=(i,)) for i in range(10)]
for t in threads: t.start()
```

### 5.3 条件变量 (Condition)

允许一个或多个线程等待，直到收到另一个线程的通知。经典应用场景：**生产者-消费者模型**。

```python
import threading
import time

queue = []
condition = threading.Condition()

def producer():
    with condition:
        print("生产者：生产了一个物品")
        queue.append("item")
        condition.notify() # 通知等待的消费者

def consumer():
    with condition:
        while not queue: # 防止虚假唤醒
            print("消费者：队列为空，等待中...")
            condition.wait() # 释放锁并阻塞，直到收到 notify() 后重新获取锁
        item = queue.pop(0)
        print(f"消费者：消费了 {item}")

t2 = threading.Thread(target=consumer)
t1 = threading.Thread(target=producer)

t2.start()
time.sleep(1)
t1.start()
```

---

## 6. 其他实用工具

### 6.1 定时器 (Timer)

在指定的延迟时间后，执行一次函数。属于 `Thread` 的子类。

```python
import threading

def hello():
    print("3秒钟到了！")

# 延迟 3.0 秒执行 hello 函数
timer = threading.Timer(3.0, hello)
timer.start()

# 如果在时间到之前想取消
# timer.cancel() 
```

### 6.2 栅栏 (Barrier)

让一组线程等待，直到所有线程都到达这个同步点（栅栏），然后一起继续放行执行。

```python
import threading
import time

barrier = threading.Barrier(3) # 需要 3 个线程到达

def racer(name, delay):
    time.sleep(delay)
    print(f"{name} 到达起跑线，等待其他人...")
    barrier.wait() # 阻塞，直到 3 个线程都调用了 wait()
    print(f"{name} 开始冲刺！")

threading.Thread(target=racer, args=("A", 1)).start()
threading.Thread(target=racer, args=("B", 2)).start()
threading.Thread(target=racer, args=("C", 4)).start()
```

---

## 7. 最佳实践总结

1. **IO密集选多线程，CPU密集选多进程 (`multiprocessing`) 或协程 (`asyncio`)**。
2. 任何涉及到修改共享资源（列表追加、全局变量计算）的操作，**必须加锁 `Lock`**，且强烈推荐使用 `with lock:` 语法。
3. 把耗时且无关紧要的后台任务设为**守护线程 (`daemon=True`)**。
4. 使用 `Event` 控制线程启停，远比用全局变量 `is_running` 加 `time.sleep()` 轮询要优雅和高效。
5. 线程池：实际开发中，更推荐使用 `concurrent.futures.ThreadPoolExecutor` 来管理和复用线程，而不是手动创建和维护一大堆 `Thread` 对象。
