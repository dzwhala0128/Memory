# 历史实验轨迹与研究结果（上下文索引）

> 整理日期：2026-11-06。以下是此前聊天中用户报告或讨论的实验记录，**未经本次会话重新计算**。不要把它当作正式论文的最终表格。涉及指标需回查对应输出、数据切分与Gold离线评价脚本。

## 1. 长期问题定义

研究主题：AVeriTeC 事实核查中的四分类 VP 预测，重点观察 NEE（证据不足）和 CE/C（冲突证据/选择性证据）性能，以及对Supported、Refuted的潜在退化。

研究并非通用地让模型“重新做一遍判决”，而是对已经得到的 VP Solution 基于现有QA证据进行定向反馈和文本优化。

## 2. 历史数据集与已报告指标

| 记录 | 报告的结果/问题 | 使用时需说明 |
|---|---|---|
| Rare73（NEE与CE/C困难集） | 原配置Accuracy约10.96%，后续报告达到约41.10% | 特定配置与子集；不能直接外推全500 |
| S/R100（Supported/Refuted） | 某些Rare73上有效的优化造成S/R样本回退 | 检查类间权衡，不能只报困难集改进 |
| 250–260的10样本段 | 历史 `exp014` 0.80 → `exp018` 1.00 | 属于小规模区间观察 |
| 250条样本运行审计 | 曾确认 `pred_count=250` | 不等于证明性能提升；当时是另一组模型/提示词环境 |
| 169条近期导师版 | `optimized=92, no_change=72, baseline_fallback=5` | **工程状态分布**，不是Accuracy或Macro-F1 |

历史四分类报告应优先给出Overall Accuracy、Macro-F1、每类Precision/Recall/F1、混淆矩阵和回退情况；评估流程必须与运行时Gold隔离。

## 3. 常见实验代号

### exp014：冻结HerO VP baseline

建立可复核的初始VP / baseline对照；用同一批固定QA作为后续优化基准，避免将证据差异误当优化效果。

### exp018：TextGrad VP Prompt优化

研究利用TextGrad反馈修订VP Prompt / 输出；用户早期比较过局部区间的前后变化。注意该实验与近期FrozenBaseline“每个样本对已存在Solution做实例优化”不完全相同。

### exp021：无金标签 TTL 与证据挑战

使用 `structured_challenge` critic、复合证据 CE/C rescue 等方式探索复杂标签识别，曾讨论Claim 259中CE/C与Refuted的冲突意见。经验教训：模型投票的margin并不是正确性的直接保证。

### exp037：迭代式Justification优化

包含Qwen critic / optimizer以及可选择迭代、极性一致性检查等版本。与当前统一的`max_iterations`设定不应混淆。

### exp038：基于exp037结果的Reader

用于检验经过改写的说明或推理文本传递给Reader后的作用。需回查当时具体脚本。

### exp040：反事实敏感性

通过改变Justification/证据条件，分析最终判断是否对关键证据关系敏感；是分析实验，不应直接视作生产门控。

### exp041：ReAct-lite

多轮Thought/Action/Observation、critic、optimizer、reader_safe_justification与risk_analysis的框架验证；重点例子为 **Claim 139、214、285**。曾讨论意外混入Claim 322以及Phase 1 Gate的问题；不要将322当成最初的三例之一。

## 4. V4-A Full169 blind simulation（2026-09起）

历史严格零样本盲模拟设想与实验入口：

```text
V4-A evaluator
   ↓
faithful backward v2 compact
   ↓
TextualGradientDescent optimizer
   ↓
structural validator
   ↓
frozen final prediction
```

- 固定169条样本。
- `max_iterations=1`。
- 不加载任何示例文件（包括Fixed-4/5/6/8），`demonstration_train_indices=[]`。
- 每个target独立构造Evaluator，禁止few-shot示例残留。
- Gold只能在预测冻结后做离线分析。
- 当时目标是使用 GPT-5.6 Sol 在云端做**model-side模拟**，**不是**在本地/服务器用Qwen复现同一个过程。
- 早期曾有V3-R、V3S等评估器和解析器版本；版本名本身不足以判断是否运行过真实模型。

历史文件线索（路径可能随代码迭代调整）：

- `docs/evaluator_v4a_full169.md`
- `scripts/eval_textgrad_full169_evaluator_v4a.py`
- `run_textgrad_full169_evaluator_v4a.sh`
- `data_store/fewshot_v4/evaluator_v4a_full169_blind_simulation_v1/`
- 历史项目仓库：[clean-main](https://github.com/dzwhala0128/hero-textgrad-research/tree/clean-main)

## 5. Fixed-6与近期2-shot是两套问题

- 历史Fixed-6示例训练索引：**1559、1247、165、1251、6、62**（4个`MATERIAL_ISSUE`与2个`NO_MATERIAL_ISSUE`）。
- 近期统一导师版默认2-shot使用训练索引 **165、6**。
- “严格zero-shot V4-A”与“带2-shot的近期FrozenBaseline”**不是同一设置**；不能在论文中把2-shot结果写成zero-shot。
- 新整理的 `fixed6.jsonl` 是待切换比较的材料，不代表近期统一运行器已经实现6-shot开关。

## 6. 研究方法上的稳定结论与风险

1. **改善Rare标签并不代表总体改善**：NEE/CE-C提升可能伴随Supported/Refuted回退。
2. **Evidence-first**：证据不相关或信息缺失时，单靠末端VP优化的性能上限受限；这为后续检索优化研究提供动机，但不是当前FrozenBaseline已经实现了检索优化的证据。
3. **区分训练/优化/判决**：Evaluator生成文本反馈、Backward形成文本梯度、Optimizer语义修订、Validator结构检查，各自职责不能混用。
4. **模型输出异常需单独报告**：`baseline_fallback`可能源于输出截断、解析错误或结构不合法，并非“No material issue”。
5. **未来正式论文实验**：必须固定运行清单与输出哈希，保留所有失败样本，给出可重现配置；不要根据Gold选择性调整单个目标样本。

## 7. 参考与工作入口

论文语境包括：HerO、TextGrad、Self-Refine、Reflexion、ReAct、Papelo、FZI-WIM，以及 AVeriTeC 工作。历史阶段曾开展Lucene BM25提速、模型服务并发和不同模型对照，**均不是近期单一Qwen9B TextGrad流水线的必选步骤**。
