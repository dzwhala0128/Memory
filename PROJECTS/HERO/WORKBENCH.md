# 网页工作台｜与Final项目对齐计划

> 记录日期：2026-11-06  
> **状态：计划中，尚未证明网页工作台已与Final代码对齐。**

## 1. 已确定的目标

用户接下来准备把现有图形化/网页工作台，**对齐当前 `Final` 项目实际逻辑与运行结果**。当前开发对象只有Final；不能引用其他历史工程作为工作台运行依据。

## 2. 对齐时要逐项核对

| 工作台视图/能力 | Final实际接口应核对的内容 |
|---|---|
| 数据集与样本选择 | `--count`、`--claim-id`、支持的子集、真实claim ID |
| AF输入模式 | `human63`、`merged67`、`original`的输入来源与适用范围 |
| 冻结初始VP | 只读取现有Justification/Verdict；不能界面无提示地重新预测 |
| Evaluator | `STATUS`与`FEEDBACK`，Few-shot文件和Prompt位置 |
| Native Backward | 实际触发条件、梯度记录与错误处理 |
| Optimizer | TextualGradientDescent调用和优化后Solution |
| 结构校验 | 独立显示结构合法/不合法，不能替用户或模型强制判标签 |
| 最终冻结/回退 | `optimized`、`no_change`、`baseline_fallback`区别 |
| 离线评价 | Gold只在冻结后加载，不能混入运行时接口 |
| 实验日志 | run-id、阶段请求/响应、失败原因和输出路径 |
| 模型服务 | 当前Qwen3.5-9B服务与端口配置的状态、错误呈现 |

## 3. 下一次具体实施前的核对顺序

1. 阅读 **Final** 的 `run.sh`、`pipeline/unified.py`、相关Prompt/配置、数据解析和输出Schema，以实际调用逻辑形成明确的后端接口表。
2. 阅读现有网页工作台源码/页面结构，将每个UI控件映射到Final参数或状态；标出缺口、重复控制和错误流程，再按小步骤修改并运行验证。

**不得假设**此前的工作台TXT描述就是现在最终实现；若只有设计草图或压缩包，必须检查实际内容。不得声称已经对齐，除非完成静态检查与必要的真实交互验证。

## 4. 验收标准（待用户确认细节）

- 工作台不会隐式重做初始VP。
- 展示的Prompt、Few-shot、AF模式等均反映真实Final配置。
- 不把 `no_change`与`baseline_fallback`混同。
- 不在冻结前引入Gold。
- 后端错误、结构验证失败可定位；输出不会覆盖已有运行ID。
- 尽量保留源代码与手工命令的单一执行事实来源，避免UI自写另一套不一致的优化逻辑。


## 5. 2026-11-06 用户提供工作台归档的静态核查（非服务器验收）

**证据范围：** 本次会话用户提供的 `工作台.zip`；依次叠加 `v1.4 Native TextGrad full → v1.4.1 format patch → v1.5 batch patch → v1.6 universal patch → v1.6.1 → v1.6.2`，仅在隔离目录检查源码。不代表服务器现装版本；也未读取服务器 `Final` 的最新代码。

- **已核实的工作台形式：** `index.html`/JavaScript前端 + `server.py` API + `studio_textgrad_worker.py` 原生TextGrad适配 + `studio_batch.py` 批量模块 + `studio_batch_offline_metrics.py` 冻结后离线指标。自由工作台含01–09区域，覆盖Claim/AF、QA、输入节点、Evaluator Prompt、请求预览、模型响应、Backward/TGD、批次及离线指标。
- **离线检查结果：** 重建后的源码 Python语法编译、五份JavaScript语法检查均通过；`python -m unittest discover -s tests -q` 共52项通过。测试没有调用Server01实际模型，不能证明浏览器真机交互或Final端到端正常。
- **主要后端不一致：** 旧工作台的 `studio_textgrad_api.project_root()` 默认指向历史源码工作树；`studio_textgrad_worker.native_import()` 依赖 `textgrad_source_native_hero/` 与 `hero_extension.hero_validator`。不能仅通过更改路径或模型ID就认为它已复用 `Final/pipeline/unified.py` 的调度与validator。
- **Evaluator / 状态不一致：** v1.6.2可选 `E18` 输出 `CLAIM TARGET / QA CHECKS / NUMERICAL / SCOPE CHECK / CRITIQUE`，不是Final记忆中约定的 `STATUS: MATERIAL_ISSUE|NO_MATERIAL_ISSUE` + `FEEDBACK`。工作台批量执行器的旧路径在Evaluator后直接进入Backward/TGD，未核实有Final的 `NO_MATERIAL_ISSUE` 跳过逻辑。
- **配置与数据源不一致：** 自由工作台默认模型仍为旧的 `qwen36-atomic:8006`（TextGrad区域另默认9B:8005）；旧批量Targeted40读取历史 `experiments/s1_qa_span_targeted40_v1`；旧离线金标签检索路径指向历史项目。需要分别与Final现行模型、默认2-shot、AF模式、冻结数据、离线评估对齐。
- **输出术语不一致：** 工作台存在 `optimized_candidate` / `format_normalized_candidate` / `baseline_structural_fallback` 等旧状态；Final记忆中的 `optimized` / `no_change` / `baseline_fallback` 必须以实际代码的枚举及语义为准。

**下一步：** 先只读核验服务器Final的 `run.sh`、`pipeline/unified.py`、模型/Prompt/Few-shot配置、AF/QA schema、每阶段状态、run-id与冻结产物；形成接口与字段映射表。之后把现有工作台作为前端/展示层，优先通过单一Final后端适配器调用实际业务逻辑，而不是保留旧工作台独立实现的另一套TextGrad流程。未完成Final源代码对比前，不声明已对齐或已可运行。
