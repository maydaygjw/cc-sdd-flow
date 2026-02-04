# 项目技术标准规范

## 1. 开发环境

* **语言**：Python（>= 3.12）

* **主要 Web 框架**：FastAPI（`fastapi`）

* **ASGI 服务器**：Uvicorn（`uvicorn`）

* **数据库 ORM**：SQLAlchemy 2.x（`sqlalchemy`）

* **数据校验**：Pydantic（`pydantic`）

* **本地运行方式**：

  ```bash
  uvicorn app.main:app --reload
  ```

* **依赖管理**：pip 

* **新增依赖**：

  ```bash
  pip install <package_name>
  ```

---

## 2. 后端技术标准

### 2.1 框架选择
- **后端框架**: FastAPI (高性能，支持异步，API文档自动生成)
- **数据库**: MySQL (事务支持，扩展性强)
- **缓存**: Redis (状态存储、会话管理)
- **消息队列**: Redis (任务队列)
- **MQTT客户端**: paho-mqtt (Python MQTT客户端库)
- **异步处理**: asyncio + uvloop (高并发处理能力)

### 2.2 API设计规范
- 使用RESTful API设计风格
- 统一响应格式：
  ```
  {
    "code": 0,           // 0表示成功，非0表示失败
    "message": "success", // 结果描述
    "data": {}           // 业务数据
  }
  ```
- 使用标准HTTP状态码
- 参数验证和错误处理

### 2.3 数据库设计规范
- 使用MySQL作为主要数据存储
- 遵循数据库命名规范（snake_case）
- 正确使用索引优化查询
- 数据一致性保障

---

## 3. 前端技术标准

### 3.1 框架选择
- **前端框架**: Vue.js 3 (现代化UI框架)
- **CSS框架**: Tailwind CSS (实用优先的CSS框架)

### 3.2 用户体验规范
- 界面响应时间 < 2秒
- 支持主流浏览器 (Chrome, Firefox, Safari, Edge)
- 响应式设计，支持桌面和移动设备访问

### 3.3 安全规范
- 使用HTTPS传输
- 输入验证和输出编码
- 防止常见的Web安全漏洞

---

## 4. 系统架构标准

### 4.1 部署架构
- 采用单一服务架构 (Monolithic Architecture)
- 模块化代码设计，便于将来按需拆分为微服务
- 容器化部署 (Docker)
- 支持水平扩展

### 4.2 代码架构
- 采用模块化设计，各功能模块职责分离
- 遵循单一职责原则
- 便于将来按内部模块拆分为独立微服务

#### 4.2.1 分层架构（严格遵守）
```
interface/ - 接口层（RESTful API、CLI、MQ消费者等）
  ├── routes/   - RESTful API 路由定义
  └── cli/      - 命令行接口（如有需要）
entity/    - 数据库实体模型（SQLAlchemy ORM Model）
schema/    - 领域对象模型（Pydantic Model，用于数据校验和序列化）
services/  - 业务逻辑层
dao/       - 数据访问层（Data Access Object）
api/       - 外部 API 接口封装
utils/     - 公共工具函数
```

**各层职责和返回类型要求：**

| 层级 | 职责 | 返回类型 | 说明 |
|------|------|----------|------|
| **interface/routes** | 处理HTTP请求，参数验证，调用service层 | `dict` (JSON) | 调用 `.model_dump()` 或 `.to_dict()` 将 Schema 转为 JSON |
| **interface/cli** | 处理命令行参数，调用service层 | 无返回值或基本类型 | 直接输出到控制台 |
| **api** | 封装外部API调用，处理HTTP通信 | `Schema` 对象 | 将外部API响应转换为 Schema 对象 |
| **services** | 实现业务逻辑，协调dao和api层 | `Schema` 对象 | 可以组合多个 Entity 或外部数据，返回 Schema |
| **dao** | 数据库CRUD操作，事务管理 | `Entity` 对象或 `List[Entity]` | 只负责数据库操作，不包含业务逻辑 |
| **entity** | 数据库表映射 | - | SQLAlchemy ORM 模型定义 |
| **schema** | 数据传输对象 | - | Pydantic 模型定义，用于验证和序列化 |

#### 4.2.2 强类型优先
- **DAO 层**：返回 Entity 对象，不返回 Dict
- **Service 层**：返回 Schema 对象，不返回 Dict
- **API 层**：返回 Schema 对象，不返回 Dict
- **Interface 层**：调用 Schema 的 `.model_dump()` 方法返回 Dict/JSON
- 使用 Pydantic 定义 Schema（领域对象模型）
- 使用 SQLAlchemy ORM 定义 Entity（数据库实体模型）

#### 4.2.3 数据流向
- **外部API数据流**: `Route → Service → API → 外部 API → Schema → Service → Route (.model_dump()) → JSON`
- **数据库数据流**: `Route → Service → DAO → Entity → Service (转换) → Schema → Route (.model_dump()) → JSON`
- **混合数据流**: `Route → Service → (DAO + API) → (Entity + Schema) → 业务处理 → Schema → Route (.model_dump()) → JSON`

**示例说明：**
```python
# DAO层 - 返回 Entity
class DeviceDAO:
    def get_device_by_id(self, device_id: str) -> DeviceEntity:
        return db.query(DeviceEntity).filter_by(id=device_id).first()

# Service层 - 返回 Schema
class DeviceService:
    def get_device_info(self, device_id: str) -> DeviceSchema:
        entity = device_dao.get_device_by_id(device_id)
        return DeviceSchema.model_validate(entity)

# Route层 - 返回 JSON
@router.get("/devices/{device_id}")
def get_device(device_id: str) -> dict:
    schema = device_service.get_device_info(device_id)
    return schema.model_dump()
```

### 4.3 身份认证
- HTTP Basic Authentication (简单基本认证)
- 后续可根据需要升级为更复杂的认证机制

### 4.4 监控与日志
- 提供健康检查接口
- 结构化日志记录
- 性能指标监控

---

## 5. 代码质量标准

### 5.1 代码规范
- 遵循PEP 8 Python代码规范
- 统一的代码格式化
- 有意义的变量和函数命名
- 适当的注释和文档
- **所有文档字符串使用中文**
- **代码注释使用中文（当需要时）**

---

## 6. 性能与可靠性要求

### 6.1 性能指标
- API响应时间 < 200ms
- MQTT消息处理延迟 < 100ms
- 支持并发连接 > 1000个设备
- 系统可用性 > 99.5%

### 6.2 扩展性
- 支持添加新类型的设备或功能
- API设计保持向后兼容性
- 模块化设计便于功能扩展

### 6.3 可靠性
- 设备连接成功率 > 99%
- 数据持久化保证

### 6.4 前端用户体验要求
- 管理后台界面响应时间 < 2秒
- 支持主流浏览器 (Chrome, Firefox, Safari, Edge)
- 响应式设计，支持桌面和移动设备访问

---

## 7. 测试与代码检查

### 7.1 测试命令

* **运行全部测试**：

  ```bash
  pytest -v
  ```

* **运行指定模块或文件的测试**：

  ```bash
  pytest tests/test_users.py -v
  ```

* **生成代码覆盖率报告**（可选）：

  ```bash
  pytest --cov=app --cov-report=term-missing
  ```

### 7.2 测试标准
- 集成测试覆盖核心业务流程
- 自动化测试流程

### 7.3 代码检查

* **代码检查（Lint）**：提交代码前需执行：

  ```bash
  ruff check .
  ```

  或（如项目使用 flake8）：

  ```bash
  flake8 .
  ```

---
