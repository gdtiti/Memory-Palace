# 测试入口地图

## 测试分层总览

这个仓库的测试大致分三层：

- `backend/tests/`
  - 后端契约、集成、运行时与安全测试
- `backend/tests/benchmark/`
  - 检索质量、退化注入、profile 基准与数据集回归
- `frontend/src/**/*.test.*`
  - 前端页面、路由与 API 客户端契约测试

如果只是改功能，通常先从 `backend/tests/` 或前端页面测试开始；  
只有改检索质量、profile 或评测链路时，才优先看 `benchmark/`。

## 后端高价值入口

### Browse / 写队列 / 读副作用

- `backend/tests/test_browse_write_lane.py`
  - 验证 `browse.create/update/delete` 是否经过 write lane
- `backend/tests/test_browse_read_side_effects.py`
  - 验证 `GET /browse/node` 不会错误强化访问计数

适合改动：

- `backend/api/browse.py`
- browse 读写语义
- write lane 接线

### MCP 工具错误契约 / 基础兼容

- `backend/tests/test_mcp_error_contracts.py`
  - 验证 `read_memory`、`search_memory`、`create/update/delete/add_alias`
    的错误返回与只读域约束
- `backend/tests/test_mcp_stdio_e2e.py`
  - 加载 `scripts/evaluate_memory_palace_mcp_e2e.py` 跑 live stdio e2e

适合改动：

- `backend/mcp_server.py`
- MCP 返回格式
- stdio 启动链

### Review / Rollback

- `backend/tests/test_review_rollback.py`
  - 覆盖 path create/delete/alias rollback 与内部错误屏蔽

适合改动：

- `backend/api/review.py`
- `backend/db/snapshot.py`
- version chain / rollback 逻辑

### 搜索 / 意图 / scope_hint

- `backend/tests/test_search_memory_scope_hint_compat.py`
  - scope_hint 与 filters 合并策略
- `backend/tests/test_week3_intent_query.py`
  - preprocess、intent classify、strategy template 应用
- `backend/tests/test_week3_intent_strategy_templates.py`
  - `search_advanced()` 的策略模板与 MMR 元数据

适合改动：

- `backend/mcp_server.py`
- `backend/db/sqlite_client.py`
- intent / strategy / retrieval 参数逻辑

### 安全 / 鉴权

- `backend/tests/test_sensitive_api_auth.py`
  - browse/review 受保护接口的 API Key 基线
- `backend/tests/test_week6_maintenance_auth.py`
  - maintenance 鉴权、loopback override、Bearer/Header
- `backend/tests/test_week6_sse_auth.py`
  - SSE 鉴权与 streaming 行为
- `backend/tests/test_main_security_contracts.py`
  - `main.py` 的健康检查与 CORS 安全契约
- `backend/tests/test_main_security_baseline.py`
  - 安全基线补充

适合改动：

- `backend/api/maintenance.py`
- `backend/run_sse.py`
- `backend/main.py`
- 任意 API 鉴权逻辑

### 运行时 / 索引 / 治理

- `backend/tests/test_index_status_short_memory_stats.py`
  - `index_status()` 的运行时统计形状
- `backend/tests/test_write_lane_metrics.py`
  - 写队列指标
- `backend/tests/test_write_lane_wal_runtime.py`
  - WAL / 运行时相关行为
- `backend/tests/test_runtime_promotion_tracker_metrics.py`
  - promotion / compact 相关运行时指标
- `backend/tests/test_week7_index_job_retry_integration.py`
  - index job retry 集成
- `backend/tests/test_week6_vitality_cleanup.py`
  - vitality cleanup 链路
- `backend/tests/test_week7_task_governance.py`
  - 治理任务相关契约

适合改动：

- `backend/runtime_state.py`
- `backend/api/maintenance.py`
- 索引 worker / sleep consolidation / vitality

### 导入 / 学习 / 外部 guard

- `backend/tests/test_external_import_api_prepare.py`
- `backend/tests/test_external_import_api_execute_rollback.py`
- `backend/tests/test_external_import_guard_security.py`
- `backend/tests/test_auto_learn_explicit_service.py`

适合改动：

- import / explicit learn 服务
- 外部导入 guard
- maintenance 的作业型接口

## 前端高价值入口

### 应用壳层与鉴权入口

- `frontend/src/App.test.jsx`
  - 路由跳转、Set/Update API key、runtime key badge
- `frontend/src/lib/api.contract.test.js`
  - API 客户端契约、runtime/stored 鉴权注入、observability/orphans 路由
- `frontend/src/lib/api.test.js`
  - `extractApiError()` 这类错误抽取逻辑

适合改动：

- `frontend/src/App.jsx`
- `frontend/src/lib/api.js`

### 页面级入口

- `frontend/src/features/memory/MemoryBrowser.test.jsx`
  - write_guard blocked 反馈、快速切换 path 的 stale response 保护
- `frontend/src/features/review/ReviewPage.test.jsx`
  - integrate/reject 双击保护、partial success、session 切换 stale protection
- `frontend/src/features/maintenance/MaintenancePage.test.jsx`
  - orphan/vitality 操作链、prepare/confirm 错误恢复
- `frontend/src/features/observability/ObservabilityPage.test.jsx`
  - retry fallback、job not found、统一 retry 端点语义

适合改动：

- 对应 `frontend/src/features/*/*.jsx`

## Benchmark 入口

这些测试更适合在“功能已对、要看质量或回归”时使用：

- `backend/tests/benchmark/test_search_memory_contract_regression.py`
- `backend/tests/benchmark/test_benchmark_retrieval_profiles.py`
- `backend/tests/benchmark/test_benchmark_degradation_injection.py`
- `backend/tests/benchmark/test_write_guard_quality_metrics.py`
- `backend/tests/benchmark/test_compact_context_gist_quality.py`
- `backend/tests/benchmark/test_profile_abcd_real_runner.py`

适合改动：

- retrieval profile
- write_guard 质量策略
- compact/gist 质量
- benchmark harness

## 按改动类型快速选测试

### 改 `backend/api/browse.py`

优先看：

1. `backend/tests/test_browse_write_lane.py`
2. `backend/tests/test_browse_read_side_effects.py`

### 改 `backend/mcp_server.py`

优先看：

1. `backend/tests/test_mcp_error_contracts.py`
2. `backend/tests/test_read_memory_include_ancestors.py`
3. `backend/tests/test_search_memory_scope_hint_compat.py`
4. `backend/tests/test_index_status_short_memory_stats.py`

### 改 `backend/api/review.py` 或 `backend/db/snapshot.py`

优先看：

1. `backend/tests/test_review_rollback.py`
2. `frontend/src/features/review/ReviewPage.test.jsx`

### 改检索 / intent / strategy / MMR

优先看：

1. `backend/tests/test_search_memory_scope_hint_compat.py`
2. `backend/tests/test_week3_intent_query.py`
3. `backend/tests/test_week3_intent_strategy_templates.py`
4. 必要时再看 `backend/tests/benchmark/`

### 改鉴权 / 安全 / CORS / SSE

优先看：

1. `backend/tests/test_sensitive_api_auth.py`
2. `backend/tests/test_week6_maintenance_auth.py`
3. `backend/tests/test_week6_sse_auth.py`
4. `backend/tests/test_main_security_contracts.py`
5. `frontend/src/lib/api.contract.test.js`

### 改前端页面

优先看：

1. 页面对应的 `frontend/src/features/*/*.test.jsx`
2. `frontend/src/App.test.jsx`
3. `frontend/src/lib/api.contract.test.js`

## 最容易漏掉的测试

- 只改了 `api.js`，没看 `api.contract.test.js`
- 只改了 `browse.py` 写入，没看 read side effect
- 只改了 retry/rebuild UI，没看 `ObservabilityPage.test.jsx`
- 只改了检索返回，没看 scope_hint / intent / MMR 相关测试
- 只改了后端鉴权，没看前端 runtime/stored key 注入契约
