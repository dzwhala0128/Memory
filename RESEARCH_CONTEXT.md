# 研究背景与方法上下文

> 上下文整理时间：2026-11-06。此文件记录过去会话中反复明确的设计目标；不要据此代替最新源码审计。

## 1. 研究任务

主线：在 AVeriTeC 自动事实核查任务上，将 **TextGrad 的文本梯度优化思想**实例化到 **HerO 的 Veracity Prediction（VP，真实性预测）阶段**，重点研究证据不足和证据冲突等复杂判断。

AVeriTeC 使用四个标签：

1. `Supported`：证据足以支持声明。
2. `Refuted`：证据足以反驳声明。
3. `Not Enough Evidence`（NEE）：现有证据不足以得出支持或反驳结论。
4. `Conflicting Evidence/Cherrypicking`（CE/C）：存在需要综合比较的冲突证据，或声明涉及选择性呈现证据的问题。

研究动机不是仅改善二分类准确率，而是提高模型对 NEE、CE/C 等证据关系的判断能力，同时避免误伤 Supported/Refuted。

**重要区别**：缺失证据 ≠ 反驳证据；看似相反的证据不一定讨论同一个可比较命题；AF（Atomic Facts）用于拆解/澄清声明，**本身不算独立外部证据**。

## 2. 原始 HerO 流程（与近期导师版分开）

历史研究中的 HerO 主要是多阶段证据驱动事实核查：

```text
Claim
  ↓
HyDE-FC / evidence retrieval
  ↓
BM25 + SFR rerank (常取 Top-10)
  ↓
Question Generation (每条证据生成问题，常为10组QA)
  ↓
HerO Veracity Prediction
  ↓
Justification + 四分类 Verdict
```

历史原流程使用 Llama 体系（例如 QG Llama-3-8B、VP Llama-3.1-70B-HerO）；**不能把这个历史配置误写为近期 FrozenBaseline 版本的初始VP运行模型**。

早期项目亦做过 Lucene 检索提速（`pyserini`、BM25、LuceneSearcher），属于历史研究实现，不应在近期仅优化 VP 的命令中自动触发检索重算。

## 3. TextGrad 优化的实际语义

当前目标是优化**已经生成的 VP Solution（Justification + Verdict）**，而不是重新生成初始VP：

```text
冻结 HerO/历史基线 VP
    + Claim + QA证据 + 可选AF
          ↓
Evaluator：识别是否存在实质性证据/推理问题
          ↓
STATUS = NO_MATERIAL_ISSUE → 保留初始Solution
STATUS = MATERIAL_ISSUE
          ↓
Native TextGrad loss.backward()：生成文本梯度
          ↓
TextualGradientDescent：更新Solution（允许更新标签语义）
          ↓
结构验证器：只检查输出结构和合法四分类标签
          ↓
冻结 Final Prediction
          ↓
离线读取Gold评价（不回流进优化过程）
```

### 明确的设计边界

- **冻结初始VP**：在 TextGrad 执行期间不重新调用 HerO VP；输入的初始 Justification/Verdict 是已有文件。
- **Evaluator**：根据 Claim、QA（及说明用途的AF）检查现有 Solution，输出 `STATUS` 和 `FEEDBACK`。原则上不注入目标样本金标签；不要把重复信息误当独立证据。
- **Backward**：按 Native TextGrad 反向路径将评价反馈传给可优化变量。
- **Optimizer**：`TextualGradientDescent.step()` 负责语义修订，包括必要时修改 Verdict。
- **Structural Validator**：是后来附加的工程保障，原生 TextGrad 不自带该环节；它**只验证形式合法性，不应替代 TextGrad 做语义判决，也不能以Gold纠正标签**。
- **Gold使用边界**：运行中 Evaluator、Backward、Optimizer、门控和结构检查不得使用目标金标签；仅在最终预测冻结后进行离线评价。
- **失败回退**：保留 Baseline 是工程安全机制；回退不能算作一次正常优化成功。
- **切忌误称**：中文请统一使用“无金标签/不使用金标签”，避免用不精确缩写混淆实验条件。

## 4. 相关数据子集

- AVeriTeC dev：常用 **500** 样本；历史工作经常采用每条声明10组QA。
- `Full169`：169条固定集合，在历史盲模拟/近期 FrozenBaseline 中使用，但不同实验配置不能混同。
- `Targeted40`：40条目标声明，有原始QA和两套拆分AF输入。
- `Rare73`：偏重 NEE 与 CE/C 的历史诊断子集。
- `S/R100`：Supported/Refuted 回退风险的历史对照子集。

任何实验的性能结论须注明：**数据集/子集、固定基线、提示词/模型版本、优化次数、评价方式**。

## 5. 学术写作的既有路线

计划围绕“复杂证据关系判断困难”组织论文，Introduction 常用四段式：

1. 事实核查背景，直接切入 NEE/CE-C 难题；
2. 证据驱动多阶段方法的能力与局限（不是只介绍HerO）；
3. VP阶段反馈/文本优化的动机与挑战；
4. 提出方法、核心设计与贡献。

研究中反复参考：HerO、TextGrad、Self-Refine、Reflexion、Papelo、FZI-WIM、ReAct及相关AVeriTeC事实核查工作。具体引文和方法归属应以论文原文核实，不能仅凭本快照引用。

## 6. 代码入口

- 历史研究：[hero-textgrad-research · clean-main](https://github.com/dzwhala0128/hero-textgrad-research/tree/clean-main)
- 近期导师版：[Hero-TTO](https://github.com/dzwhala0128/Hero-TTO)

**未来会话必须先问“讨论的是历史实验代码还是近期 FrozenBaseline 代码？”或在用户已明确时直接读取相应仓库；不要把不同版本拼成一个未经验证的流程。**
