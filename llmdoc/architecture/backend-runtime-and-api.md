# 后端运行时与 API 地图

## 为什么先看这篇

如果你要回答“服务怎么启动”“为什么接口需要 API Key”“运行时队列和索引是谁在管”，
这篇是后端最短入口。

## 启动与生命周期入口

HTTP 管理面的主入口是 `backend/main.py`，建议按下面顺序读：

1. `_try_restore_legacy_sqlite_file()`
   - 兼容旧库文件名，必要时恢复历史 SQLite 数据
2. `lifespan()`
   - 启动时初始化 SQLite、调用 `runtime_state.ensure_started()`
   - 关闭时调用 `runtime_state.shutdown()` 并释放数据库连接
3. `app.include_router(...)`
   - 注册 `review`、`browse`、`maintenance` 三组 HTTP 路由
4. `/health`
   - 汇总数据库与运行时状态，是部署后最先该看的接口

一句话理解：

```text
backend/main.py
  -> SQLite init / legacy restore
  -> runtime_state.ensure_started()
  -> register routers
  -> expose /health and /docs
```

## 后端其实有三种启动面

如果只盯 `backend/main.py`，很容易误以为这是一个单纯的 FastAPI CRUD 服务。
更准确的理解是：

- HTTP 管理面
  - `backend/main.py`
- MCP stdio 面
  - `backend/mcp_server.py`
- MCP SSE 面
  - `backend/run_sse.py`

其中：

- `backend/mcp_wrapper.py`
  - 更像启动包装层，主要处理 Windows 下的 stdio 包装细节
- `backend/run_sse.py`
  - 把 `mcp.sse_app()` 暴露成独立 ASGI 服务

一句话理解：

```text
同一套 SQLite + runtime_state
  <- FastAPI HTTP 管理面
  <- MCP stdio 工具面
  <- MCP SSE 工具面
```

## 统一鉴权入口

统一鉴权函数在 `backend/api/maintenance.py` 的
`require_maintenance_api_key()`。

它的行为要点：

- 从 `MCP_API_KEY` 读取服务端配置的密钥
- 允许通过两种头部传入：
  - `X-MCP-API-Key`
  - `Authorization: Bearer <token>`
- 若开启 `MCP_API_KEY_ALLOW_INSECURE_LOCAL`，且请求来自 loopback，
  可以在未配置 API Key 时走本地不安全豁免

这个函数不只保护 `maintenance`：

- `backend/api/maintenance.py`
  - 路由级统一依赖
- `backend/api/review.py`
  - 路由级统一依赖
- `backend/api/browse.py`
  - 各端点显式 `Depends(require_maintenance_api_key)`

也就是说，Dashboard 的 `browse / review / maintenance` 都属于受保护接口。

## 三组 HTTP 路由各管什么

### `browse`

文件：`backend/api/browse.py`

职责：

- 浏览树状记忆节点
- 创建、更新、删除单个节点
- 读取子节点、gist、breadcrumbs、alias

特点：

- 使用 `write_guard()` 决定是否允许创建/更新
- 通过 `runtime_state.write_lanes.run_write()` 串行化写入
- 更偏“产品 UI 操作接口”

### `review`

文件：`backend/api/review.py`

职责：

- 列出 session / snapshot
- 查看 diff
- rollback / integrate
- 审查 deprecated memory

特点：

- 面向“写入后的人类复核”
- 核心依赖 `db.snapshot` 和当前 memory version chain

### `maintenance`

文件：`backend/api/maintenance.py`

职责：

- 索引 worker 状态与任务管理
- vitality 衰减与清理审批
- orphan 清理
- import / explicit learn
- observability 搜索与概览

特点：

- 是最像“系统控制台”的 API 组
- 同时承载安全敏感操作与运行态观察

## 运行时子系统

核心文件是 `backend/runtime_state.py`。

建议优先关注这些对象：

- `WriteLaneCoordinator`
  - 负责 session lane + global lane 两层写入协调
- `SessionSearchCache`
  - 维护进程内、按 session 的检索缓存
- `GuardDecisionTracker`
  - 记录 write guard 决策事件，供 observability 使用
- `IndexTaskWorker`
  - 异步处理索引重建、重试、sleep consolidation 等任务
- `VitalityDecayCoordinator`
  - 负责记忆活力衰减相关调度
- `RuntimeState`
  - 把上面这些运行时组件装配在一起

一句话理解：

```text
RuntimeState
  = 写入协调 + 检索缓存 + guard 观测 + 索引后台任务 + vitality 调度
```

## 遇到什么问题先看这里

- 服务起不来
  - 先看 `backend/main.py`
- `/health` 不正常
  - 先看 `backend/main.py` 和 `backend/runtime_state.py`
- 所有 Dashboard 接口都 401
  - 先看 `backend/api/maintenance.py`
- 写入卡住、队列堆积、索引任务异常
  - 先看 `backend/runtime_state.py`
- review / rollback 行为不对
  - 先看 `backend/api/review.py` 和 `backend/db/snapshot.py`

## 推荐下一跳

- 想看记忆怎么写进去、怎么搜出来：
  - 读 `write-and-retrieval-pipeline.md`
- 想看前端为什么会 401、按钮点了走哪个 API：
  - 读 `dashboard-and-auth-flow.md`
- 想看 Docker / skills / 多 CLI 怎么接这几种启动面：
  - 读 `deployment-and-client-integration.md`
