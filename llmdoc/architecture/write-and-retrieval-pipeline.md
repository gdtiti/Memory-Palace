# 写入与检索主链

## 为什么需要单独一篇

这个仓库最容易混淆的地方，是“HTTP 页面写入”和“MCP 工具写入”共享了很多底层能力，
但并不完全是同一条链。看清这件事，排障速度会快很多。

## 两类写入入口

### 入口 A：MCP 工具入口

文件：`backend/mcp_server.py`

核心工具：

- `read_memory`
- `create_memory`
- `update_memory`
- `delete_memory`
- `add_alias`
- `search_memory`
- `compact_context`
- `rebuild_index`
- `index_status`

这条链更偏“Agent 直接调用能力层”。

### 入口 B：HTTP Browse 入口

文件：`backend/api/browse.py`

核心端点：

- `GET /browse/node`
- `POST /browse/node`
- `PUT /browse/node`
- `DELETE /browse/node`

这条链更偏“Dashboard 产品操作层”。

## 写入时的共用能力

无论从 MCP 还是 Browse 进入，底层都围绕这几件事展开：

1. 校验 domain / path
2. 调用 `db/sqlite_client.py` 的 `write_guard()`
3. 经 `runtime_state.write_lanes` 串行化写入
4. 落到 SQLite 数据层
5. 由后台索引 worker 处理后续索引任务

共享的关键文件：

- `backend/mcp_server.py`
- `backend/api/browse.py`
- `backend/runtime_state.py`
- `backend/db/sqlite_client.py`

## MCP 写入链的特殊点

MCP 链比 Browse 链多一层**显式快照与审查语义**。

关键点在 `backend/mcp_server.py`：

- `_snapshot_memory_content()`
- `_snapshot_path_meta()`
- `get_session_id()`

也就是说，MCP 写入主链更接近：

```text
MCP tool
  -> parse_uri / validate
  -> snapshot
  -> write_guard
  -> write lane
  -> sqlite write
  -> index worker
```

这也是为什么 `review` 页面与 `db.snapshot` 更贴近 MCP 侧的改动审计语义。

## Browse 写入链的特殊点

Browse 端点也走 write guard 和 write lane，但重点是 UI 上的节点管理，
当前文件 `backend/api/browse.py` 里并没有像 MCP 层那样显式做一套
snapshot helper 编排。

更准确的理解是：

```text
Dashboard browse action
  -> validate domain/path
  -> write_guard
  -> write lane
  -> sqlite create/update/remove_path
```

因此如果问题是“页面按钮点了没生效”，优先看 `browse.py`；  
如果问题是“为什么这次 AI 修改能在 review 里回看和回滚”，优先看 `mcp_server.py`。

## 检索主链

对外入口虽然有多个，但检索核心都收敛到 `backend/db/sqlite_client.py`。

从职责上看，检索链包含：

1. 查询预处理
2. 意图分类
3. 策略模板选择
4. `keyword / semantic / hybrid` 检索
5. 可选 rerank 与降级说明

可以把它理解成：

```text
search request
  -> sqlite_client preprocess
  -> intent classify
  -> strategy choose
  -> retrieve candidates
  -> rerank / degrade reasons
```

外部常见入口包括：

- `backend/mcp_server.py` 的 `search_memory`
- `backend/api/maintenance.py` 的 `POST /maintenance/observability/search`

## 哪些文件最值得先看

### 想看“为什么 write guard 做出这个决定”

先看：

1. `backend/api/browse.py`
2. `backend/mcp_server.py`
3. `backend/db/sqlite_client.py`
4. `backend/runtime_state.py`

### 想看“为什么写入成功但搜不到”

先看：

1. `backend/db/sqlite_client.py`
2. `backend/runtime_state.py`
3. `backend/api/maintenance.py`
4. `backend/tests/test_search_memory_scope_hint_compat.py`

### 想看“为什么 compact / rebuild / index_status 不符合预期”

先看：

1. `backend/mcp_server.py`
2. `backend/runtime_state.py`
3. `backend/api/maintenance.py`

## 容易踩坑的认知点

- 不要把 Browse 写入和 MCP 写入当成完全同一条链
  - 它们共享底层写入能力，但外层编排目标不同
- 不要把“能写入”误当成“能被高质量检索到”
  - 中间还隔着索引与检索策略
- 不要只盯某个 API 文件
  - 这类问题通常至少跨 `api/`、`runtime_state.py`、`sqlite_client.py`

## 推荐下一跳

- 要看后端总入口：
  - 读 `backend-runtime-and-api.md`
- 要看前端如何触发这些接口：
  - 读 `dashboard-and-auth-flow.md`
