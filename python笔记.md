# 一、类装饰器

## @classmethod

1. **绑定到类，而非实例**：类方法接收的第一个参数是类本身（通常命名为 cls），而不是类的实例（self）。
2. **无需实例化即可调用**：你可以直接通过 类名.方法名() 调用，也可以通过 实例.方法名() 调用。
3. **常用于工厂模式**：通过接收不同的参数来创建并返回类的实例。

```python
class Book:
    def __init__(self, title, author):
        self.title = title
        self.author = author

    @classmethod
    def from_string(cls, book_str):
        # 接收 "Title-Author" 格式的字符串并返回实例
        title, author = book_str.split("-")
        return cls(title, author)

# 直接调用类方法创建实例
book = Book.from_string("Python编程-Guido")
print(book.title)  # 输出: Python编程
```

## ABC & @abstractmethod

1. **定义抽象基类 (Abstract Base Class)**：`ABC` 是 Python 提供的一个基类，用于定义接口规范。
2. **强制子类实现方法**：使用 `@abstractmethod` 装饰的方法在子类中**必须**被重写（Override），否则子类也无法被实例化。
3. **防止直接实例化**：抽象基类本身不能被直接创建对象，它只作为模板存在。

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self):
        """子类必须实现计算面积的方法"""
        pass

class Square(Shape):
    def __init__(self, side):
        self.side = side
  
    def area(self):
        return self.side * self.side

# shape = Shape()  # 报错：无法实例化抽象类
sq = Square(5)
print(sq.area())  # 输出: 25
```

# 二、类型注释

## Callable[[str], str]

1. **含义**：表示一个**可调用对象**（如函数、lambda 表达式或实现了 `__call__` 的类实例）。
2. **结构解析**：`Callable[[参数类型1, 参数类型2, ...], 返回类型]`。
   - `[str]`：括号内表示参数列表，这里表示接收一个 `str` 类型的参数。
   - `str`：逗号后面表示返回值的类型。
3. **常见场景**：用于高阶函数（即接收函数作为参数或返回函数的函数）。

```python
from typing import Callable

def uppercase_formatter(text: str) -> str:
    return text.upper()

def process_text(text: str, formatter: Callable[[str], str]) -> str:
    # 接收一个字符串和一个格式化函数
    return formatter(text)

# 传入函数作为参数
result = process_text("hello", uppercase_formatter)
print(result)  # 输出: HELLO

# 传入 lambda 表达式
result_lambda = process_text("world", lambda s: s + "!")
print(result_lambda)  # 输出: world!
```

## Union[[Type1, Type2, ...]]

1. **含义**：表示一个变量或参数**可以是多种类型中的任意一种**（“联合”类型）。
2. **结构解析**：`Union[类型1, 类型2]`，意为“要么是类型1，要么是类型2”。
3. **常见场景**：常用于函数的参数类型提示或返回值类型提示，比如既支持传入单条字符串，也支持传入字符串列表进行批量处理。
4. **新版语法**：在 Python 3.10 及以上版本，可以用更直观的 `Type1 | Type2` 管道符语法替代 `Union`。

```python
from typing import Union, List

# 传统写法 (Python 3.5+)
def process_data(data: Union[str, List[str]]):
    # 在函数内部通常需要配合 isinstance 进行类型判断
    if isinstance(data, str):
        print(f"处理单条文本: {data}")
    else:
        print(f"批量处理 {len(data)} 条文本")

# Python 3.10+ 的现代写法
def process_data_new(data: str | list[str]):
    pass

process_data("hello")
process_data(["hello", "world"])
```

# 三、魔术方法

## __new__

1. **作用**：在内存中真正创建并返回一个类的实例对象。它是类的实例化第一步，先于 `__init__` 被调用。
2. **区别**：`__new__` 负责**创建对象**（可理解为分配内存），必须返回该类的实例；`__init__` 负责**初始化对象**（赋值属性），只能返回 `None`。
3. **常见用法**：常被用来实现**单例模式**（基于类变量缓存拦截限制多余实例的生成），或者继承不可变内置类型（如 tuple, str）。

```python
class Singleton:
    _instance = None
  
    def __new__(cls, *args, **kwargs):
        # 拦截对象创建过程，保证全局只创建一个实例
        if cls._instance is None:
            cls._instance = super().__new__(cls) # 调用父类object的__new__()方法创建实例，这里的cls不可以省略，不然创建的是父类实例。
        return cls._instance

obj1 = Singleton()
obj2 = Singleton()
print(obj1 is obj2)  # 输出: True
```

## __repr__

1. **作用**：返回对象的“官方”字符串表示，主要用于**调试和开发**。
2. **触发**：控制台输出对象、`repr(obj)`、或查看列表等容器内容时调起。
3. **区别**：`__str__` 面向用户（可读性）；`__repr__` 面向开发者（明确性）。
4. **惯例**：通常返回 `f"ClassName(attr='{val}')"` 格式。

```python
class Tool:
    def __init__(self, name): self.name = name
    def __str__(self): return f"Tool: {self.name}"
    def __repr__(self): return f"Tool(name='{self.name}')"

tool = Tool("Search")
# print(tool)   -> Tool: Search
# print([tool]) -> [Tool(name='Search')]
```

# 四、内置函数

## all()

1. **核心语义**：“全真即真”。如果迭代器中所有元素都为 `True`，则返回 `True`。
2. **短路特性**：只要遇到一个 `False`，立即返回 `False`，不再检查后续元素。
3. **空列表行为**：`all([])` 返回 `True`（因为没有元素为假）。
4. **常见用法**：配合生成器表达式进行条件校验。

```python
# 示例：检查是否所有参数都在字典中
required_params = ["name", "age"]
parameters = {"name": "Alice", "age": 25}

is_valid = all(p in parameters for p in required_params) # True

# 与 any() 对比
# any(): "一真即真"，只要有一个为 True 就返回 True。
```

## next()

1. **核心语义**：从迭代器（或生成器）中手动提取出**下一个元素**。
2. **安全取值（默认值兜底）**：`next(iterator, default)`。当迭代器被耗尽（没数据了）时，如果提供了 `default` 值，就会直接返回该值，而不会抛出 `StopIteration` 异常使程序崩溃。
3. **常见用法**：常与生成器表达式配合，用于“在序列中查找满足条件的第一个元素”，极其简练且性能极高（一旦找到就暂停，不浪费算力）。

```python
# 示例：查找列表中第一个符合条件的元素
numbers = [3, 7, 11, 14, 18]

# 配合生成器表达式，找到 14 就立刻停止
first_match = next((n for n in numbers if n > 10 and n % 2 == 0), None)
print(first_match)  # 输出: 14

# 若全不符合，安全返回默认值
not_found = next((n for n in numbers if n > 100), "查无此数")
print(not_found)    # 输出: 查无此数
```

## 字典的方法

### get()

1. **作用**：安全地从字典中获取指定键的值。如果键不存在，不会抛出 `KeyError`。
2. **语法**：`dict.get(key, default=None)`
3. **常见用法**：在不确定字典中是否包含某个键时，避免程序报错。

```python
data = {"name": "Alice"}
# 直接访问不存在的键会报错
# print(data["age"])  # 抛出 KeyError

# 使用 get()
print(data.get("age"))        # 默认返回 None
print(data.get("age", 18))    # 如果不存在，返回指定的默认值 18
```

### setdefault()

1. **作用**：安全地向字典中添加键值对。如果键**已存在**，则返回该键对应的值，什么也不做；如果键**不存在**，则将该键设置为你指定的默认值，并返回这个默认值。
2. **语法**：`dict.setdefault(key, default=None)`
3. **常见用法**：适用于需要在字典中给确实缺失的键“兜底”赋初值的场景。

```python
config = {"theme": "dark"}

# 键已存在，不会被覆盖，返回原本的值 "dark"
current_theme = config.setdefault("theme", "light") 
print(current_theme) # 输出: dark

# 键不存在，会将键值对添加进字典，并返回默认值 8080
port = config.setdefault("port", 8080)
print(config)        # 输出: {'theme': 'dark', 'port': 8080}
```

## 对象相关的内置函数

### hasattr()

1. **作用**：检查一个对象是否包含指定的属性或方法。如果存在返回 `True`，否则返回 `False`。
2. **语法**：`hasattr(object, 'name')` （注意：第二个参数 `name` 必须是字符串）。
3. **常见用法**：在动态提取配置、调用对象属性或执行鸭子类型判断时，用来提前做安全检查，以防止抛出 `AttributeError`。

```python
class Config:
    def __init__(self):
        self.port = 8080

config = Config()

# 检查存在的属性
if hasattr(config, "port"):
    print(f"Server port: {config.port}")  # 输出: Server port: 8080

# 检查不存在的属性
if not hasattr(config, "url"):
    print("URL is not configured.")       # 输出: URL is not configured.
```

# 五、高级语法特性

## 生成器表达式 (Generator Expression)

1. **结构语法**：`(表达式 for item in iterable if 条件)`。和**列表推导式** `[...]` 的语法几乎完全一致，只是把外层的中括号换成了小括号 `()`。
2. **惰性求值（Lazy Evaluation）**：列表推导式会**立即**在内存中生成并保存整个列表数据，如果你有一百万条数据，列表推导式会瞬间填满你的内存。而**生成器表达式**只返回一个“承诺”——只有在被迭代调用时（如被 `for` 循环遍历或传入 `next()`、`all()` 等），才会按需“挤牙膏”式地一个一个产生数据。
3. **极致省内存**：对于处理海量数据或无法确定边界的无限序列时，生成器是防止内存溢出（OOM）的必要手段。
4. **配合内置函数**：直接当做参数传给 `sum()`, `max()`, `min()`, `all()`, `any()` 等支持接收迭代器的函数时，可以再**省略一层显式的小括号**。

```python
import sys

# 列表推导式：立即计算所有结果并占用大量连续内存
list_comp = [i**2 for i in range(1000000)]
print(sys.getsizeof(list_comp))  # 输出高达 8 MB

# 生成器表达式：立刻返回生成器对象，几乎不占内存
gen_expr = (i**2 for i in range(1000000))
print(sys.getsizeof(gen_expr))   # 输出仅 104 字节左右

# 按需获取并计算结果
print(next(gen_expr))  # 输出: 0
print(next(gen_expr))  # 输出: 1

# 作为内置函数参数时，可以省略掉生成器自己那一层()
total = sum(i**2 for i in range(10)) 
```

## async & await (异步编程)

1. **核心概念**：`async` 和 `await` 是 Python 中用于处理并发（异步 I/O）的关键字，主要用于网络请求、文件读写、数据库查询等耗时等待操作，避免阻塞主线程。
2. **定义和执行**：使用 `async def` 定义的函数称为“协程（Coroutine）”。直接调用协程函数只会返回一个协程对象，并不会运行。必须通过事件循环（如 `asyncio.run()`）或在其他协程中被 `await` 调用才会执行。
3. **交出控制权 (await)**：`await` 操作符用于挂起当前协程，将控制权交还给事件循环，去执行其他任务。它后面跟的必须是一个**可等待对象**（Awaitable），比如另一个协程对象或 `Task`。
4. **极大地提升效率**：在 I/O 密集型任务中，通过 `asyncio.gather()` 等方法并发执行多个协程，可以大幅缩短总的等待时间。

```python
import asyncio
import time

async def fetch_data(task_id: int):
    print(f"任务 {task_id}: 开始执行...")
    # 模拟网络请求等耗时 I/O 操作 (非阻塞等待)
    await asyncio.sleep(2)  
    print(f"任务 {task_id}: 执行完成！")
    return f"Result_{task_id}"

async def main():
    start_time = time.time()
    
    # 并发执行 3 个异步任务
    # fetch_data() 返回的是协程对象，gather 会将它们打包并发调度
    results = await asyncio.gather(
        fetch_data(1),
        fetch_data(2),
        fetch_data(3)
    )
    
    print(f"所有结果: {results}")
    print(f"总耗时: {time.time() - start_time:.2f} 秒")  # 大约 2 秒，而不是 6 秒

# 启动事件循环作为程序的入口点
# asyncio.run(main()) 
```

# Temp、杂七杂八

## python导包

### 1、内部文件

### 2、外部库

- Anaconda虚拟环境下外部包的位置：`Anaconda\envs\agent_venv\Lib\site-packages\hello_agents`

## 合并参数

```
    # 合并参数
    llm_params = {
        "temperature": self.config.temperature,
        "max_tokens": self.config.max_tokens,
        **kwargs                                    # 当kwargs中包含"temperature","max_tokens"时，会直接覆盖掉
    }

    # 优化
    llm_params = self.config.to_dict() # 获取全量默认配置      确保self.config都是可以传入方法的参数！！
    llm_params.update(kwargs)          # 用手动参数覆盖
```
