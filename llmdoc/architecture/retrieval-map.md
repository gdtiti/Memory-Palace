# 检索地图

## 推荐先读的架构文档

在正式搜源码前，建议先按这个顺序读：

1. `backend-runtime-and-api.md`
2. `write-and-retrieval-pipeline.md`
3. `dashboard-and-auth-flow.md`
4. `deployment-and-client-integration.md`

这样能先把“服务入口、写入/检索、前端壳层、部署接入”四块拼起来，
再决定往 `api/`、`db/`、`features/` 还是 `scripts/` 下潜。

## 核心模块与职责

- `backend/main.py`
  - FastAPI 入口、生命周期管理、CORS、路由注册、健康检查
- `backend/mcp_server.py`
  - MCP 工具实现、记忆快照、URI 解析、写入主链路编排
- `backend/runtime_state.py`
  - Write Lane、Index Worker、vitality 衰减与运行时调度
- `backend/db/sqlite_client.py`
  - 数据模型落库、CRUD、检索、Write Guard、gist、embedding/rerank
- `backend/api/browse.py`
  - 面向前端的记忆浏览与增删改接口
- `backend/api/review.py`
  - 快照、diff、rollback、integrate 相关接口
- `backend/api/maintenance.py`
  - 索引、清理、导入/学习任务、observability 相关接口
- `backend/run_sse.py`
  - SSE 传输与 MCP API Key 鉴权入口
- `frontend/src/App.jsx`
  - 页面路由总入口与鉴权壳层
- `frontend/src/lib/api.js`
  - 前端统一 API 客户端与运行时鉴权注入
- `frontend/src/features/*`
  - 四个页面的业务视图与交互逻辑
- `deploy/` 与 `scripts/`
  - 部署档位、容器入口、运维脚本和辅助工具

## 主控制流

### 1. Dashboard 请求链路

```text
Browser
  -> frontend/src/App.jsx
  -> frontend/src/lib/api.js
  -> backend/main.py
  -> api/browse.py | api/review.py | api/maintenance.py
  -> runtime_state.py / db/sqlite_client.py
```

优先用于排查：

- 页面加载失败
- API Key 注入问题
- 某个页面接口 401 / 404 / 500
- 仪表盘状态展示与后端状态不一致

### 2. MCP 写入链路

```text
AI Client
  -> backend/mcp_server.py
  -> Write Guard
  -> Snapshot
  -> Write Lane
  -> SQLite 写入
  -> Index Worker
```

优先用于排查：

- 记忆创建/更新行为异常
- 误写入、重复写入、应为 NOOP 却发生 UPDATE
- 写入成功但搜索不到
- rollback 或 snapshot 异常

### 3. 检索链路

```text
search_memory / HTTP search
  -> sqlite_client.py
  -> query preprocess
  -> intent classify
  -> strategy choose
  -> keyword / semantic / hybrid retrieve
  -> rerank / degrade reasons
```

优先用于排查：

- 搜索召回差
- 意图识别不对
- embedding 或 reranker 降级
- 不同 deployment profile 下检索结果差异

## 按场景找文件

### 场景 A：想看“系统从哪里启动”

先看：

1. `backend-runtime-and-api.md`
2. `backend/main.py`
3. `frontend/src/App.jsx`
4. `docker-compose.yml`

### 场景 B：想看“记忆写入为什么这样判定”

先看：

1. `write-and-retrieval-pipeline.md`
2. `backend/mcp_server.py`
3. `backend/db/sqlite_client.py`
4. `backend/runtime_state.py`
5. `backend/tests/test_week2_write_guard.py`

### 场景 C：想看“为什么能回滚 / 看 diff”

先看：

1. `backend-runtime-and-api.md`
2. `backend/api/review.py`
3. `backend/db/snapshot.py`
4. `frontend/src/features/review/ReviewPage.jsx`

### 场景 D：想看“为什么搜索结果不稳定 / 有降级”

先看：

1. `write-and-retrieval-pipeline.md`
2. `backend/db/sqlite_client.py`
3. `backend/tests/test_search_memory_scope_hint_compat.py`
4. `backend/tests/benchmark/`
5. `docs/EVALUATION.md`

### 场景 E：想看“前端某个页面的数据从哪里来”

先看：

1. `dashboard-and-auth-flow.md`
2. `frontend/src/features/<page>/`
3. `frontend/src/lib/api.js`
4. 对应的 `backend/api/*.py`

### 场景 F：想看“部署和环境变量怎么配”

先看：

1. `deployment-and-client-integration.md`
2. `docker-compose.yml`
3. `deploy/profiles/`
4. `deploy/docker/`
5. `.env.example`
6. `docs/DEPLOYMENT_PROFILES.md`

### 场景 G：想看“技能、MCP、外部客户端如何接入”

先看：

1. `deployment-and-client-integration.md`
2. `docs/skills/README.md`
3. `docs/skills/SKILLS_QUICKSTART.md`
4. `backend/mcp_server.py`
5. `backend/run_sse.py`

## 修改前的最小检查清单

- 这是前端问题、后端问题，还是部署/环境问题？
- 是 HTTP 路由链路，还是 MCP 工具链路？
- 是“写入错误”，还是“检索错误”，还是“展示错误”？
- 有没有现成测试可直接缩小定位范围？

只要先回答这四个问题，基本就能快速进入正确文件。
