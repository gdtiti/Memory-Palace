# llmdoc 索引

## 目的

`llmdoc/` 是给 Codex / Agent 使用的最小项目文档层，用来快速建立
Memory Palace 的工作心智模型，避免每次都从源码和零散说明里重新摸索。

这里不替代仓库现有的公开文档；更详细的产品、技术和部署说明仍以
`README.md`、`README_CN.md`、`docs/TECHNICAL_OVERVIEW.md`、
`docs/GETTING_STARTED.md` 与 `docs/DEPLOYMENT_PROFILES.md` 为准。

## 目录地图

- `overview/project-overview.md`
  - 项目目标、核心能力、技术栈、运行与部署要点
- `architecture/retrieval-map.md`
  - 关键模块职责、主数据流/控制流、按场景定位源码入口
- `architecture/backend-runtime-and-api.md`
  - 后端启动、鉴权、三组 HTTP 路由与运行时子系统
- `architecture/write-and-retrieval-pipeline.md`
  - MCP / HTTP 两条写入入口与统一检索主链
- `architecture/dashboard-and-auth-flow.md`
  - 前端路由壳层、API Key 注入、四大页面职责
- `architecture/deployment-and-client-integration.md`
  - Docker/Profiles、脚本、skills 与多客户端接入
- `guides/doc-maintenance.md`
  - 何时更新文档、如何保持最小准确、如何与代码变更同步
- `guides/source-reading-order.md`
  - 第一次进入仓库时的稳定阅读顺序与避坑提醒
- `guides/change-impact-checklist.md`
  - 常见改动类型对应的落点、连带影响与同步清单
- `reference/coding-conventions.md`
  - 命名、错误处理、测试与验证约定
- `reference/test-entry-map.md`
  - 后端、前端、benchmark 测试入口地图
- `agent/`
  - 临时调查报告、任务草稿、一次性分析产物

## 建议阅读顺序

1. 先读 `overview/project-overview.md`
2. 再读 `architecture/retrieval-map.md`
3. 进入后端主链时读 `architecture/backend-runtime-and-api.md`
4. 排查写入/搜索时读 `architecture/write-and-retrieval-pipeline.md`
5. 排查前端或 401/页面数据时读 `architecture/dashboard-and-auth-flow.md`
6. 涉及部署、技能或 CLI 接入时读 `architecture/deployment-and-client-integration.md`
7. 准备开始看源码时读 `guides/source-reading-order.md`
8. 准备改代码时读 `guides/change-impact-checklist.md`
9. 需要执行任务时读 `guides/doc-maintenance.md`
10. 动手修改前查 `reference/coding-conventions.md`
11. 需要缩小验证范围时查 `reference/test-entry-map.md`

## 当前项目的一句话理解

Memory Palace 是一个面向 AI Agent 的长期记忆系统：后端用 FastAPI +
SQLite + MCP 提供可写、可检索、可审计的记忆能力，前端用 React
仪表盘提供 Memory / Review / Maintenance / Observability 四大视图，
并通过 Docker 与 profile 脚本支持多档位部署。

## 深入阅读入口

- 产品与快速上手：`README.md`、`README_CN.md`
- 技术全景：`docs/TECHNICAL_OVERVIEW.md`
- 客户端与 MCP/Skills：`docs/skills/`
- 部署：`docker-compose.yml`、`deploy/`、`docs/DEPLOYMENT_PROFILES.md`
- 故障排查：`docs/TROUBLESHOOTING.md`

## 首轮深读建议

如果你是第一次进入这个仓库，推荐按下面顺序走一遍：

1. `overview/project-overview.md`
2. `architecture/backend-runtime-and-api.md`
3. `architecture/write-and-retrieval-pipeline.md`
4. `architecture/dashboard-and-auth-flow.md`
5. `architecture/deployment-and-client-integration.md`
6. `guides/source-reading-order.md`
7. `reference/test-entry-map.md`

这样可以先建立“后端主链 -> 写入/检索 -> 前端壳层 -> 部署接入”的框架，
再决定是否继续下潜到某个 `api/`、`db/`、`features/` 或 `scripts/` 目录。
