# Neo4j 学习笔记

## 1. Neo4j 简介
Neo4j 是一个高性能的 NoSQL 图形数据库，它将结构化数据存储在网络（从数学角度看是图）上而不是表格中。

### 核心概念
- **节点 (Nodes)**：图中的实体（如人、地点、事物）。
- **关系 (Relationships)**：连接节点的线，具有方向和类型。
- **属性 (Properties)**：节点和关系的详细信息（键值对）。
- **标签 (Labels)**：用于对节点进行分类（如 Person, City）。

### 核心优势
- **原生图存储**：索引自由的邻接（Index-free adjacency），处理复杂关联查询极快。
- **Cypher 查询语言**：类 SQL 的声明式图查询语言。
- **ACID 支持**：确保数据一致性。

---

## 2. 本地 Docker 部署 (Docker Compose)

在 `hello_agents_library` 项目中，我们通过修改 `docker-compose.yml` 实现了与 Qdrant 共存的部署方案。

### Docker Compose 配置
```yaml
  neo4j:
    image: neo4j:latest
    container_name: hello-agents-neo4j
    ports:
      - "7474:7474" # HTTP 访问
      - "7687:7687" # Bolt 协议
    volumes:
      - ./memory_data/neo4j/data:/data
      - ./memory_data/neo4j/logs:/logs
      - ./memory_data/neo4j/import:/var/lib/neo4j/import
      - ./memory_data/neo4j/plugins:/plugins
    environment:
      - NEO4J_AUTH=neo4j/password123
    restart: unless-stopped
    networks:
      - agent-network
```

### 部署步骤
1. **初始化目录** (PowerShell):
   ```powershell
   New-Item -ItemType Directory -Path "memory_data/neo4j/data", "memory_data/neo4j/logs", "memory_data/neo4j/import", "memory_data/neo4j/plugins" -Force
   ```
2. **启动服务**:
   ```powershell
   docker-compose up -d
   ```

---

## 3. 访问与验证
- **管理界面**: [http://localhost:7474](http://localhost:7474)
- **初始账号**: `neo4j`
- **初始密码**: `password123`

### 常用测试语句 (Cypher)
```cypher
// 创建测试节点
CREATE (n:Agent {name: 'Copilot', version: 'G3F'}) RETURN n;

// 查看所有节点
MATCH (n) RETURN n LIMIT 25;
```
