# HerO 网页工作台 ↔ Final 接口映射基线（2026-11-06）

> 状态：**静态映射完成；Server01 当前 Final 源码一致性与真实交互尚待验收**。
> 本页由实际检查用户上传的 `工作台.zip`（重建 v1.6.2）与 `dzwhala0128/Hero-TTO@main` 参照源码得出。后者不是服务器 Final 最新文件夹的已验证镜像。未改动工作台或 Final 运行代码；本地工作台 52 项单元测试通过，不代表真实模型实验通过。

## 1. UI → 现有工作台代码 → Final 参照入口

| UI | 已检查的工作台代码/API | 目标对应 |
|---|---|---|
| 01 Claim / AF | `studio_free.js`；拆分 POST `/api/chat` | `pipeline/unified.py:selected(), render_af()`、冻结 AF 数据 |
| 02 QA 列表与筛选 | `studio_free.js`；前端缓存/编辑 | `selected(), render_qa()`、`data/*/runtime.json` |
| 03 独立输入节点 | `studio_free.js` extra nodes | Final 无任意额外输入合同；只能作为沙盒 |
| 04 Prompt / 模型 / Few-shot | `studio_free.js` E18等自由模板 | Final `prompts/`、`fewshot/fixed2.jsonl`、`configs/qwen9b.json` |
| 05 完整请求预览 | `studio_free.js` 前端模板拼接 | Final `evaluator_input()` 的实际最终请求 |
| 06 Evaluator Raw / Parsed | `/api/chat`、可选自由 parser | Final `parse_feedback()` 严格 `STATUS/FEEDBACK` |
| 07 Native Backward / TGD | `studio_textgrad.js` → `studio_textgrad_api.py` → `studio_textgrad_worker.py` | Final `run_one()` → `native_backward.py` → TGD → `validator.py` |
| 08 批量执行/历史/导出 | `studio_batch.js` → `studio_batch.py`；`/api/batch/jobs` | Final `run.sh qwen` + `pipeline/unified.py` + `outputs/<run-id>/` |
| 09 离线评价 | `studio_batch_offline_metrics.py` | Final `pipeline/compare_metrics.py` / `evaluate.py` |
| 旧节点画布 | `app.js`；`/api/chat` | Final 无任意节点图；保留为沙盒 |
| Stage A/B 独立调试 | `studio_debug.js`；`/api/chat` | 不是当前 Final 主链 |
| 历史左右对比 | `studio_debug.js`；`/api/history` | 对接 Final run-id → claim-id → 各阶段日志 |

## 2. 已核实的旧 HTTP 接口

- 通用模型：`GET /api/status`、`POST /api/chat`。
- 自由实验历史：`GET /api/history`、`GET /api/history/{id}`。
- Native TextGrad：`GET /api/textgrad/status`、`POST /api/textgrad/backward`、`POST /api/textgrad/optimizer`、`GET /api/textgrad/runs`、`GET /api/textgrad/runs/{id}`。
- 旧批次：`GET /api/batch/targeted40`、`GET/POST /api/batch/jobs`、`GET /api/batch/jobs/{id}`、`POST .../pause`、`POST .../resume`、`GET .../export`、`GET/POST .../offline-metrics`。

这些 **均不是** Final 当前的原生 HTTP 接口；现有工作台通过自己的后端和独立 Worker 执行业务逻辑。

## 3. Final 参照真实数据契约（服务器待复核）

- 入口：`run.sh qwen` → `pipeline.unified`；支持 `--count 1|10|40|169|500`、`--af auto|human63|merged67|original`、`--claim-id`（仅 count=1）、`--preflight`、`--run-id`、`--freeze`、`--resume`。
- 读取已有冻结 Initial VP；`initial_vp_model_calls=0`。对 `count<=40` 可用人工/合并 AF 或 original QA；169/500 仅 original。每次使用相应冻结 QA/AF 输入，不无提示重做 VP。
- 参照代码默认 Qwen3.5-9B、8005、8192 context、固定 `fewshot/fixed2.jsonl` 的2-shot、no-think。
- 主流程：Evaluator `STATUS: MATERIAL_ISSUE|NO_MATERIAL_ISSUE` + `FEEDBACK` → **两种 STATUS 都执行 Native backward** → `NO_MATERIAL_ISSUE` 返回 `no_change`；另一种才执行 TGD+结构校验，成功 `optimized`，异常 `baseline_fallback`。
- Final 输出：`outputs/<run-id>/run.json`、`progress.jsonl`、`claims/<claim-id>/result.json` 和阶段请求/响应/审计文件、`predictions.json`、`freeze.json`；冻结后才离线 `compare`。
- `optimized/no_change/baseline_fallback` 是不同的执行语义，不能与旧工作台 `optimized_candidate/format_normalized_candidate/baseline_structural_fallback` 直接等同。

## 4. 高优先级缺口

1. 旧工作台输入可自由编辑，Final 正式实验必须使用可追踪的冻结来源；建议隔离“Final 正式复现”和“自由探索”。
2. 工作台 E18 等自由 Evaluator Prompt、Parser 与 Final 的真实 2-shot、严格 `STATUS/FEEDBACK` 不同；应展示 Final 服务端真实请求，而非浏览器自行拼接。
3. 旧批次 Evaluator 有响应就继续 TGD，缺少 Final 的 STATUS 分支。
4. 旧 Worker 引用 `textgrad_source_native_hero` 和 `hero_extension.hero_validator`，与 Final 的 `textgrad_native` / `pipeline/validator.py` 不同；不能仅重命名路径。
5. 旧工作台默认允许格式兼容、auto-fit、可变模型/上下文/thinking；Final 在参照代码中严格 JSON 校验、固定模型、分阶段预算、无静默截断。
6. 旧批次 `MAX_ITEMS=100`，不能原样实现 Final 169/500；历史 Targeted40 与 Gold 来源不是 Final 对应路径。
7. 离线指标应读取真实 freeze/hash 和完整目标集合，不能将失败样本排除后的部分准确率冒充全量结果。

## 5. 目标改造边界（设计，未实施）

保留界面展示和已有自由实验能力；新增 Final Adapter 读取真实配置、冻结样本、预检、run-id、进度、单条阶段 Trace、结构状态、freeze 和离线指标。建议单独使用 `/api/final/*` 命名空间，不复用老 `/api/chat` 伪装正式运行；最终由唯一的 Final `run.sh` / `pipeline/unified.py` 执行。页面本身不复制另一套 Evaluator / backward / optimizer 逻辑。

**下一次开始修改之前**：仍要只读复核 Server01 上当前 Final 的 `run.sh`、`pipeline/unified.py`、`qwen_engine.py`、`native_backward.py`、`validator.py`、数据/Prompt/Few-shot/config 和真实 outputs。不能把本页静态映射宣传为已完成部署、已经连接服务器或通过端到端验收。

## 6. 来源

- 工作台：当前会话用户提供 `工作台.zip`，按 v1.4→v1.4.1→v1.5→v1.6→v1.6.1→v1.6.2 叠加。
- Final 参照： https://github.com/dzwhala0128/Hero-TTO/tree/main
- 现行项目身份与历史约束：本目录 `FINAL.md`、`RESEARCH.md`、`EXPERIMENTS.md`、`WORKBENCH.md`。
