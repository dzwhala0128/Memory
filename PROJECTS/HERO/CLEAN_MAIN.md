# clean-main｜当前 HerO × TextGrad 科研实验主线

> 更新时间：2026-11-07（用户当次明确纠正）  
> **当前实验及代码分析的第一优先级：[`dzwhala0128/hero-textgrad-research` 的 `clean-main` 分支](https://github.com/dzwhala0128/hero-textgrad-research/tree/clean-main)。**  
> 此条是**用户最新明确指定的主线**，优先于此前记忆中的“Final 为唯一项目”说法。

## 1. 项目身份与决策优先级

- 用户明确表示：**“我现在还是以clean-main项目为主线做实验”**。
- 讨论最新实验、优化方案、TextGrad 实现、Prompt、Evaluator、Backward、Optimizer、验证器时，**默认只读检查 clean-main 当前真实仓库代码**；不要求用户重新粘贴可直接从 GitHub 读取的代码。
- `Final` 的已有记录保留在 [FINAL.md](FINAL.md)，属于**独立或较早的工作线资料**；不能用其模型、Few-shot、代码、实验结果或运行状态替代 clean-main，不能再称 Final 为“唯一现行科研项目”。
- `Hero-TTO` 也是独立参考仓库，不是 clean-main 的自动同步镜像。
- 如果用户明确指向某个其他目录/仓库开展新任务，以当次明确指令为准。通用项目阅读入口先用本文件，再进入相关专题。

## 2. 研究背景与核心语义

- HerO / AVeriTeC 四分类真实性预测：Supported、Refuted、Not Enough Evidence、Conflicting Evidence/Cherrypicking；重点研究 NEE 与 CE/C 的困难判定。
- 主流程以冻结初始 VP 的 Justification + Verdict、Claim、10 组 QA（可选 AF）为基础；重点分析 Evaluator → 原生 TextGrad `loss.backward()` → `TextualGradientDescent.step()` → 结构验证 → 冻结预测 → Gold 隔离的离线评价。
- **Evaluator** 负责评价/反馈；**Backward** 依据真实计算图将文字梯度传播到待优化的 Solution；**Optimizer** 负责对文本语义及必要的 Verdict 做改写；结构 Validator 不代替模型做正确性决策。
- 用户偏好术语**“无金标签/不使用金标签”**。Gold 标签不得作为模型运行时反馈、门控或结构验证依据。
- 用户正在探索**从 Optimizer 起步**的手动梯度与多版本对比实验，追溯 Feedback 和 Native Backward 如何影响 Verdict；在分析对比时，预期 Verdict 只是分析目标，不是 Gold/准确率。

## 3. 与新图形化工作台的关系

- 全新 HerO Visual Workbench v1.0 是针对**clean-main**的独立网页工作台，存放于服务器 `project/Hero-view/` 空目录中，完整 ZIP 由用户自行上传。
- 冻结设计、页面、梯度/反馈版本库、实验矩阵、阶段调试、历史溯源、交付状态见 [WORKBENCH_CLEAN_MAIN_V1.md](WORKBENCH_CLEAN_MAIN_V1.md)。
- 新工作台**不是**旧版 `工作台.zip` 的 `Final` 适配，也不是对 `Final` 研究运行代码的直接修改。
- 不得声称已经在 Server01 部署或真实跑过 GPU/vLLM 实验；截至本次记忆更新，用户尚未提交部署验收记录。

## 4. 已知研究线索（非“当前分支已复跑”结论）

- 历史实验：exp014/018/021/037/038/040/041，Rare73、S/R100、Full169；详细实验背景见 [EXPERIMENTS.md](EXPERIMENTS.md)。
- 近期研究过 Full169 的 V4-A blind simulation、Evaluator/Backward/TGD 的调用与提示词策略；核对具体逻辑须以 `clean-main` 文件内容为准。
- 2026-11-06 工作台开发过程只读核对过的 `clean-main` 提交：`4242ba01174bcac5bada2e9c55eda1c9cd80fd10`。这是**当次工作台代码审计基线**，不是保证后续永不变化的最新提交。
- 曾在代码中核对 `textgrad_source_native_hero/textgrad/optimizer/optimizer.py`、`textgrad_source_native_hero/textgrad/variable.py`、`textgrad_source_native_hero/textgrad/autograd/llm_ops.py`、`scripts/native_backward_backend_v1.py`，以及 `hero_extension/hero_validator.py` 等。再讨论当前逻辑时须重新核验实际文件。

## 5. 下次会话默认动作

1. 用户讨论 HerO 实验/代码/新工作台时，先读取本文件、[RESEARCH.md](RESEARCH.md) 和相关专题。
2. 需要代码判断时，优先访问 [clean-main](https://github.com/dzwhala0128/hero-textgrad-research/tree/clean-main) 当前源码，记录参考提交和路径；不要把历史 `Final` 说明冒充最新事实。
3. 如出现重要已确认的代码/实验/架构变化，在可访问 GitHub 且符合 [Memory Policy](../../SYSTEM/MEMORY_POLICY.md) 的前提下主动更新本仓库，并保留用户报告/静态审查/实际执行的证据等级。
