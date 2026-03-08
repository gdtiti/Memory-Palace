# 源码阅读顺序

## 目标

这份清单回答一个很具体的问题：

> 第一次进入 Memory Palace 仓库时，按什么顺序读，最不容易迷路？

它追求的是“最稳的第一轮阅读路径”，不是覆盖所有实现细节。

## 推荐顺序

### 第 0 步：先建立文档心智模型

先读：

1. `llmdoc/index.md`
2. `llmdoc/overview/project-overview.md`
3. `llmdoc/architecture/backend-runtime-and-api.md`
4. `llmdoc/architecture/write-and-retrieval-pipeline.md`

为什么：

- 先知道系统分成 HTTP 管理面、MCP 工具面、前端 Dashboard、部署/skills
- 再进源码时，就不容易把“产品接口”“MCP 接口”“运行时子系统”混成一层

### 第 1 步：看服务总入口

先看：

1. `backend/main.py`
2. `frontend/src/App.jsx`
3. `docker-compose.yml`

为什么：

- `backend/main.py` 说明 HTTP 服务如何启动、注册路由、暴露 `/health`
- `frontend/src/App.jsx` 说明页面壳层怎么组成
- `docker-compose.yml` 说明系统在运行态其实是 `backend + sse + frontend` 三服务

### 第 2 步：看 MCP 与共享运行时

先看：

1. `backend/mcp_server.py`
2. `backend/run_sse.py`
3. `backend/runtime_state.py`

为什么：

- `mcp_server.py` 是 MCP 工具面主入口，也是快照、session、工具编排层
- `run_sse.py` 说明 SSE 面如何把 MCP 暴露出去
- `runtime_state.py` 说明写队列、索引 worker、观测、vitality 调度如何共享

### 第 3 步：看真正的数据与检索内核

先看：

1. `backend/db/sqlite_client.py`
2. `backend/db/snapshot.py`
3. `backend/models/schemas.py`

为什么：

- `sqlite_client.py` 是写入、检索、write guard、version chain 的核心实现
- `snapshot.py` 解释 review / rollback 的快照语义
- `schemas.py` 有助于快速理解 API / 工具层数据结构

### 第 4 步：按场景看具体路由

如果你关心：

- 树形浏览 / 页面写入
  - 看 `backend/api/browse.py`
- diff / rollback / integrate
  - 看 `backend/api/review.py`
- 索引、清理、导入、观测
  - 看 `backend/api/maintenance.py`

这一步不要一开始就三份全精读，先按任务选一条主线。

### 第 5 步：回到前端数据连接点

先看：

1. `frontend/src/lib/api.js`
2. `frontend/src/features/memory/MemoryBrowser.jsx`
3. `frontend/src/features/review/ReviewPage.jsx`
4. `frontend/src/features/maintenance/MaintenancePage.jsx`
5. `frontend/src/features/observability/ObservabilityPage.jsx`

为什么：

- `api.js` 说明 API Key 注入、错误抽取、接口契约适配
- 四个 feature 页面分别对应四个产品视图，是前端的真正业务入口

### 第 6 步：最后再看部署与接入细节

先看：

1. `deploy/profiles/`
2. `deploy/docker/`
3. `scripts/docker_one_click.sh`
4. `scripts/apply_profile.sh`
5. `docs/skills/`
6. `scripts/sync_memory_palace_skill.py`
7. `scripts/install_skill.py`

为什么：

- 这部分很重要，但不是第一轮理解业务主链的起点
- 更适合在“已经知道系统怎么工作”之后，再看如何落到不同环境或不同 CLI

## 一张简化顺序图

```text
llmdoc
  -> backend/main.py + frontend/src/App.jsx + docker-compose.yml
  -> backend/mcp_server.py + backend/run_sse.py + backend/runtime_state.py
  -> backend/db/sqlite_client.py + backend/db/snapshot.py
  -> backend/api/*.py
  -> frontend/src/lib/api.js + frontend/src/features/*
  -> deploy/ + scripts/ + docs/skills/
```

## 按任务快速分支

### 想查 401 / API Key / 浏览器页面空白

先走：

1. `frontend/src/lib/api.js`
2. `frontend/src/App.jsx`
3. `backend/api/maintenance.py`

### 想查“为什么写入被挡 / why write_guard”

先走：

1. `backend/mcp_server.py`
2. `backend/api/browse.py`
3. `backend/db/sqlite_client.py`
4. `backend/runtime_state.py`

### 想查“为什么能 rollback / diff”

先走：

1. `backend/api/review.py`
2. `backend/db/snapshot.py`
3. `frontend/src/features/review/ReviewPage.jsx`

### 想查“为什么搜不到 / 检索效果不对”

先走：

1. `backend/db/sqlite_client.py`
2. `backend/mcp_server.py`
3. `backend/runtime_state.py`
4. `backend/api/maintenance.py`

## 第一轮不建议深挖的地方

下面这些内容不是没价值，而是**不适合第一轮**：

- `backend/tests/datasets/`
  - 数据量大，容易把注意力拉到 benchmark 样本而不是主链
- `backend/tests/benchmark/helpers/`
  - 更适合做评测或性能回归时再看
- `docs/images/`
  - 主要是展示资产，不是实现入口
- `deploy/profiles/*/*.env`
  - 先知道 profile 概念，再下潜参数细节
- 大量一次性脚本
  - 先看 `docker_one_click`、`apply_profile`、`sync/install skill` 这类主入口

## 最后提醒

- 第一轮阅读的目标是找到**主链和边界**
- 真正改代码前，仍要回到具体文件重新定位函数和调用点
- 如果要进入验证阶段，下一步直接看 `reference/test-entry-map.md`
