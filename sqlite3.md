#　sqlite3

## 1. 介绍

...

## 2. 连数据库

```
conn = sqlite3.connect("my_db.db")  # 如果数据库路径"my_db.db"不存在则自动创建；如果传入"：memory:"代表在内存中创建，程序结束则丢失数据
cursor = conn.cursor()              # 定义游标，通过cursor执行sql指令
```

## 3. 建表

```
"""
# CREATE TABLE 建表 ； IF NOT EXISTS  确保建表不报错

# 数据类型
    - INTEGER ：整数
    - TEXT : 文本
    - REAL : 浮点数
    - BLOB：二进制数据
    - NULL：空值

# 约束条件:
    - PRIMARY KEY : 主键，唯一标识该行数据（如用户的专属ID），全表不能有该值相同的两行。
    - AUTOINCREMENT : 自动递增
    - NOT NULL : 不能为空
    - UNIQUE : 唯一
    - DEFAULT : 默认值
    - FOREIGN KEY : 外键，用于与其他表关联，保证引用的数据必须存在（如评论表里的 user_id 必须在用户表里的id存在）。

"""

cursor.execute("""
    CREATE TABLE IF NOT EXISTS user (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    age INTEGER,
    address TEXT
)
""")
```

## 4. 插入 (INSERT)

```python
# 插入单条数据  INSERT INTO 表名 (列1, 列2, 列3) VALUES (值1, 值2, 值3)
cursor.execute("INSERT INTO user (name, age, address) VALUES (?, ?, ?)", ("张三", 25, "北京"))

# 批量插入多条数据
users = [
    ("李四", 30, "上海"),
    ("王五", 28, "广州")
]
cursor.executemany("INSERT INTO user (name, age, address) VALUES (?, ?, ?)", users)

conn.commit()  # 必须提交事务，数据才会真正写入数据库
```

## 5. 查看 (SELECT)

```python
# SELECT 要查的列 FROM 表名
# 查询后 - fetchall() 返回全部结果； - fetchone（） 返回第一条结果； - fetchmany(n) 返回前n条结果

# 查询所有数据  * 代表所有的字段 
cursor.execute("SELECT * FROM user")
all_users = cursor.fetchall()  # 获取所有结果 [('张三', 25...), ('李四', 30...)]
for user in all_users:
    print(user)

# 条件查询 (WHERE)
cursor.execute("SELECT name, age FROM user WHERE age > ?", (28,))
older_users = cursor.fetchall()

# 获取单条数据
cursor.execute("SELECT * FROM user WHERE id = ?", (1,))
one_user = cursor.fetchone() 
```

## 6. 改 (UPDATE)

```python
# UPDATE 表名 SET 列1=新值, 列2=新值 WHERE 条件;
cursor.execute("UPDATE user SET address = ? WHERE name = ?", ("深圳", "李四"))
conn.commit()  # 记住更新后需要 commit
```

## 7. 删 (DELETE)

```python
# DELETE FROM 表名 WHERE 条件
cursor.execute("DELETE FROM user WHERE age < ?", (26,))
conn.commit()

# 注意！！！如果忘了加 WHERE，会清空整张表！
# cursor.execute("DELETE FROM user") 

# 最后断开连接
conn.close()
```

## 8. 级联删除 (ON DELETE CASCADE)

**级联删除（Cascade Delete）**是在设置外键（`FOREIGN KEY`）时非常实用的一种自动清理机制。

* **作用**：当“主表”中某条记录被删除时，数据库会自动把你指定的“从表”里所有引用了该记录的数据一并删除，防止产生无主的“孤立关联数据（Orphan Data）”。
* **优点**：保证数据完整性，并在代码层面省去了手动清理所有关联表的繁琐步骤。

**代码示例**：

```sql
CREATE TABLE memory_concepts (
    memory_id TEXT NOT NULL,
    concept_id TEXT NOT NULL,
    -- 设置联合主键
    PRIMARY KEY (memory_id, concept_id),
    -- 设置外键并开启级联删除
    FOREIGN KEY (memory_id) REFERENCES memories (id) ON DELETE CASCADE,
    FOREIGN KEY (concept_id) REFERENCES concepts (id) ON DELETE CASCADE
)
```

*在以上结构中，只要你执行了一次 `DELETE FROM memories WHERE id = '某记忆'`，连带的 `memory_concepts` 中的相关绑定记录也会被数据库底层自动且瞬间销毁。*


## 9. 索引（Index）

**索引（Index）**类似于一本书的“目录”或者“索引页”。它的主要作用是极大地提高数据库中检索特定数据的速度。

* **原理**：当表中有大量数据时，如果没有索引，数据库必须逐行扫描（Full Table Scan）来寻找匹配的行，非常缓慢。建了索引后，数据库可以直接通过类似目录的数据结构快速跳到目标行。
* **优缺点**：
    * **优点**：大幅提升数据查询（`SELECT`）、条件过滤（`WHERE`）和多表连接（`JOIN`）的速度。
    * **缺点**：1. 索引本身会占用额外的存储空间；2. 降低数据写入速度（`INSERT`、`UPDATE`、`DELETE`变慢），因为每次修改数据时，相关的索引也必须同步更新。
* **适用场景**：适合给经常作为搜索条件（经常放在 `WHERE` 后面）或用于表关联的字段加索引。

**代码示例**：

```sql
-- 给 user 表的 age 字段建一个名为 idx_user_age 的单个索引
CREATE INDEX IF NOT EXISTS idx_user_age ON user (age);

-- 之后当你执行 SELECT * FROM user WHERE age = 25 时，速度会得到质的飞跃
```

