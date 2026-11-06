# HerO × TextGrad｜研究背景与基本原则

## 1. 研究目标

面向AVeriTeC自动事实核查四分类，把Native TextGrad文本梯度用于VP阶段的实例级优化：根据Claim/QA对初始Justification与Verdict进行反馈和改写，重点研究 `Not Enough Evidence`（NEE）以及 `Conflicting Evidence/Cherrypicking`（CE/C）的识别困难。

四标签：

| 标签 | 说明 |
|---|---|
| Supported | 证据充分支持 |
| Refuted | 证据充分反驳 |
| Not Enough Evidence | 无足够证据判定支持/反驳 |
| Conflicting Evidence/Cherrypicking | 相关证据之间存在冲突或选择性呈现问题 |

重点警惕：**没有证据不等于反驳**；不同时间、主体、条件的说法不一定构成真正冲突；AF是声明拆分和澄清，不是独立事实证据。

## 2. HerO原始多阶段思路

`Claim → Evidence retrieval / reranking → Question Generation → Veracity Prediction → Justification + Verdict`。

历史研究中使用过HyDE-FC检索、BM25/SFR排序、QG、Llama系列模型的VP；但当前Final优化链路**不重算检索/QG/初始VP**，而是读入已有冻结Solution，围绕其优化。

## 3. Native TextGrad语义边界

1. **Evaluator**：基于证据识别当前Solution中的实质性推理/证据问题，提供文字反馈。
2. **Backward**：Native TextGrad `loss.backward()` 把反馈沿文本计算图传播到Solution。
3. **Optimizer**：`TextualGradientDescent.step()` 对Solution做语义修订，可根据证据改写Verdict。
4. **结构验证器**：新增工程组件，只检查结构合法性和四分类标签，**不负责语义判决、不使用目标Gold**。
5. **冻结、离线评价**：先锁定最终输出，之后才允许用Gold计算Accuracy、Macro-F1、分类别结果。

Gold不能进入Evaluator、Backward、Optimizer、门控、验证器的运行时上下文。中文术语优先用“无金标签/不使用金标签”。

## 4. 子集与实验背景

- AVeriTeC dev全量常用500条。
- Full169固定169条；Targeted40有AF拆分与原始QA不同输入方案。
- Rare73与S/R100为早期分析复杂类别提升及普通标签退化的对照。
- 历史实验含TextGrad优化、不同critic/optimizer、选择性迭代和反事实敏感性等。**这些历史结果并不自动代表Final运行结果**。

## 5. 研究与写作习惯

- 研究问题：事实核查模型面对复杂证据关系（NEE、CE/C）的判定可靠性。
- 论文Introduction倾向四段：背景与问题 → 证据驱动方法及不足 → VP优化挑战/动机 → 方法和贡献。
- 讨论代码应根据用户当前指定的源码仓库/分支读取具体实现；不凭记忆代替代码证据。
- 相关参考：HerO、TextGrad、Self-Refine、Reflexion、ReAct、Papelo、FZI-WIM、AVeriTeC相关研究。对具体论文主张需阅读原论文再引用。
