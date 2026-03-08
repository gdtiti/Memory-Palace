# 改动落点清单

## 目标

这份清单用来回答：

> 我准备改某一类能力，代码大概率落在哪，旁边哪些文件也要一起看？

它不是精确到函数级别的“唯一答案”，但适合做第一轮影响面判断。

## 1. 改后端 HTTP 路由

典型场景：

- 新增/修改 browse、review、maintenance 接口

主落点：

- `backend/api/browse.py`
- `backend/api/review.py`
- `backend/api/maintenance.py`

常见连带影响：

- `backend/main.py`
  - 路由注册、健康检查聚合
- `backend/models/schemas.py`
  - 请求/响应模型
- `frontend/src/lib/api.js`
  - 前端调用封装
- 对应页面组件

容易漏：

- 鉴权依赖是否一致
- 前端 API 封装是否同步
- 对应页面测试是否同步

## 2. 改 MCP 工具面

典型场景：

- 改 `read_memory` / `create_memory` / `search_memory`
- 改 system URI、tool 返回契约、stdio/SSE 行为

主落点：

- `backend/mcp_server.py`
- `backend/run_sse.py`
- `backend/mcp_wrapper.py`

常见连带影响：

- `backend/runtime_state.py`
- `backend/db/sqlite_client.py`
- `backend/db/snapshot.py`
- `docs/skills/`
- `scripts/run_memory_palace_mcp_stdio.sh`

容易漏：

- stdio 与 SSE 两个面是否都一致
- `docs/skills/*` 的接入口径是否过时
- `test_mcp_error_contracts.py` 与 `test_mcp_stdio_e2e.py` 是否需要同步

## 3. 改写入治理 / Write Guard / Version Chain

典型场景：

- 改 write_guard 决策
- 改 create/update/delete 语义
- 改 alias、deprecated、version chain

主落点：

- `backend/db/sqlite_client.py`
- `backend/mcp_server.py`
- `backend/api/browse.py`
- `backend/api/review.py`

常见连带影响：

- `backend/db/snapshot.py`
- `backend/runtime_state.py`
- `frontend/src/features/memory/MemoryBrowser.jsx`
- `frontend/src/features/review/ReviewPage.jsx`

容易漏：

- Browse 与 MCP 两条入口是否行为一致
- rollback / snapshot 是否仍成立
- 前端 blocked 提示是否仍匹配后端返回

## 4. 改检索 / Intent / Retrieval Profile

典型场景：

- 改 query preprocess、scope_hint、intent classify
- 改 `keyword / semantic / hybrid`
- 改 MMR、embedding、reranker、降级策略

主落点：

- `backend/db/sqlite_client.py`
- `backend/mcp_server.py`
- `backend/api/maintenance.py`

常见连带影响：

- `deploy/profiles/`
- `.env.example`
- `docs/DEPLOYMENT_PROFILES.md`
- `docs/EVALUATION.md`
- `backend/tests/benchmark/`

容易漏：

- profile 文档与默认参数不同步
- scope_hint / filters 合并契约被改坏
- benchmark 回归入口没同步看

## 5. 改运行时 / 索引 / Vitality / Observability

典型场景：

- 改 write lane
- 改 index worker / retry / sleep consolidation
- 改 runtime 指标、cleanup、治理任务

主落点：

- `backend/runtime_state.py`
- `backend/api/maintenance.py`
- `backend/mcp_server.py`

常见连带影响：

- `frontend/src/features/maintenance/MaintenancePage.jsx`
- `frontend/src/features/observability/ObservabilityPage.jsx`
- `frontend/src/lib/api.js`

容易漏：

- 后端 summary 结构变化后，前端页面仍按旧字段读
- retry / cancel / rebuild 逻辑只改了后端没改 UI fallback

## 6. 改前端页面

典型场景：

- 改 Memory / Review / Maintenance / Observability 页面

主落点：

- `frontend/src/features/memory/MemoryBrowser.jsx`
- `frontend/src/features/review/ReviewPage.jsx`
- `frontend/src/features/maintenance/MaintenancePage.jsx`
- `frontend/src/features/observability/ObservabilityPage.jsx`

常见连带影响：

- `frontend/src/App.jsx`
- `frontend/src/lib/api.js`
- 对应后端 `backend/api/*.py`

容易漏：

- 只改页面状态，不改 API 错误提示
- 只改页面，不核对后端字段名和返回形状

## 7. 改前端鉴权 / API 客户端

典型场景：

- 改 runtime key / stored key 读取
- 改 `Authorization` / `X-MCP-API-Key`
- 改错误抽取与提示

主落点：

- `frontend/src/lib/api.js`
- `frontend/src/App.jsx`

常见连带影响：

- `backend/api/maintenance.py`
- `backend/run_sse.py`
- `backend/api/browse.py`
- `backend/api/review.py`

容易漏：

- browse/review/maintenance 都受保护，但只测了其中一个
- runtime 与 stored 两种 key 来源只覆盖了一种

## 8. 改部署 / Profile / 容器

典型场景：

- 改 compose、端口、环境变量、Dockerfile、profile 模板

主落点：

- `docker-compose.yml`
- `deploy/docker/`
- `deploy/profiles/`
- `.env.example`

常见连带影响：

- `docs/DEPLOYMENT_PROFILES.md`
- `scripts/docker_one_click.sh`
- `scripts/docker_one_click.ps1`
- `scripts/apply_profile.sh`
- `scripts/apply_profile.ps1`

容易漏：

- 文档口径没跟着改
- 一键脚本和 compose/env 模板不同步

## 9. 改 Skills / CLI 接入

典型场景：

- 改 canonical bundle
- 改 sync/install 脚本
- 改多 CLI 接入说明

主落点：

- `docs/skills/memory-palace/`
- `docs/skills/README.md`
- `docs/skills/SKILLS_QUICKSTART.md`
- `docs/skills/MEMORY_PALACE_SKILLS.md`
- `scripts/sync_memory_palace_skill.py`
- `scripts/install_skill.py`

常见连带影响：

- `scripts/run_memory_palace_mcp_stdio.sh`
- 不同 CLI 的 repo-local 或 user-scope 安装说明

容易漏：

- canonical bundle 改了，但镜像同步脚本没跟
- 文档仍写成“某客户端完全开箱即用”，但实际还有边界

## 改动前的最小自问

开始动手前，建议先问自己 4 个问题：

1. 这是 HTTP 管理面、MCP 工具面，还是前端壳层问题？
2. 真正核心逻辑在 `api/`、`runtime_state.py`，还是 `sqlite_client.py`？
3. 这次改动会不会顺带影响鉴权、快照、索引、前端字段形状？
4. 对应的 `llmdoc`、公开 `docs/`、测试入口是否需要同步？

如果这 4 个问题答不清，先回去读：

- `architecture/backend-runtime-and-api.md`
- `architecture/write-and-retrieval-pipeline.md`
- `reference/test-entry-map.md`
