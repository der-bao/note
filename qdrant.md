# Qdrant 学习笔记与部署指南

## 1. 什么是 Qdrant？

**Qdrant** (读音为 "quadrant") 是一个开源的高性能**向量数据库**和**向量搜索引擎**。在现代 AI 应用（尤其是 RAG - 检索增强生成）中，它充当了 Agent 的“长期记忆库”。

### 核心概念

* **向量 (Vector)**：将文本、图像等非结构化数据通过 Embedding 模型转化为一串数字坐标。
* **集合 (Collection)**：类似于传统数据库的“表”，用于存储具有相同维度的向向量。
* **点 (Point)**：集合中的基本单元，包含：
  * **ID**: 唯一标识。
  * **Vector**: 向量数据。
  * **Payload**: 关联的元数据（如原始文本、用户ID、时间戳等，支持过滤查询）。
* **距离度量 (Distance Metric)**：计算向量相似度的方式，如 `Cosine` (余弦相似度)、`Euclidean` (欧氏距离) 等。

### 为什么在本项目中使用它？

1. **极速检索**：支持毫秒级的在大规模数据中进行相似度查找。
2. **混合查询**：可以同时进行“向量查找”和“标签过滤”（例如：查找关于“Python”的记忆，但仅限“张三”说过的）。
3. **持久化**：相比内存存储，Docker 部署后数据会持久化到磁盘中。

---

## 2. 本地 Docker 部署步骤

### A. 创建配置文件

在项目根目录下准备 `docker-compose.yml`：

```yaml
version: '3.8'

services:
  qdrant:
    image: qdrant/qdrant:latest
    container_name: hello-agents-qdrant
    ports:
      - "6333:6333"  # HTTP API & Dashboard
      - "6334:6334"  # gRPC
    volumes:
      - ./qdrant_data:/qdrant/storage  # 数据持久化挂载
    restart: unless-stopped
```

### B. 启动服务

在终端执行：

```bash
docker-compose up -d
```

### C. 访问管理界面

在浏览器打开：[http://localhost:6333/dashboard](http://localhost:6333/dashboard)

---

## 3. 项目配置修改 (.env)

启动本地服务后，需将项目中的 `.env` 配置从云端切换至本地：

```env
# 连接地址
QDRANT_URL=http://localhost:6333
# 本地模式通常无需 API Key
QDRANT_API_KEY=

# 基础参数
QDRANT_COLLECTION=hello_agents_vectors
QDRANT_VECTOR_SIZE=384  # 需与使用的 Embedding 模型一致
QDRANT_DISTANCE=cosine
```

---

## 4. 常用管理命令

| 命令                       | 描述                                 |
| :------------------------- | :----------------------------------- |
| `docker-compose up -d`   | 后台启动服务                         |
| `docker-compose stop`    | 停止服务                             |
| `docker-compose down`    | 停止并移除容器（不删数据）           |
| `docker-compose logs -f` | 查看运行日志                         |
| `rm -rf ./qdrant_data`   | **慎用**：删除所有本地向量数据 |
