# HerO × TextGrad｜实验与验证日志

> 用户于2026-11-07明确将科研实验主线设为 **[clean-main](CLEAN_MAIN.md)**。本文件的原有历史结果与 `Final` 阶段快照分开记录，均不得未经重新验证就视为 clean-main 现行结果。

## A. 既有研究实验（历史线索）

| 实验名 / 子集 | 当时讨论或报告的结论 | 说明 |
|---|---|---|
| `exp014` | 初始VP baseline对照 | 用于与后续TextGrad比较 |
| `exp018` | VP Prompt优化 | 小样本历史对照中曾观察到准确率变化 |
| `exp021` | 无金标签的结构化证据挑战与复合CE/C研究 | 涉及Claim 259 |
| `exp037` | 迭代Justification与选择性更新 | 曾使用极性一致性检查 |
| `exp038` | 基于迭代输出的Reader检查 | 与主优化阶段须区分 |
| `exp040` | 反事实Justification敏感性分析 | 偏诊断性质 |
| `exp041` | ReAct-lite框架验证 | 主要样本139、214、285 |
| `Rare73` | 历史指标有10.96%→41.10%改善的报告 | 特定配置和子集，不能外推 |
| `S/R100` | 历史优化曾造成Supported/Refuted回退 | 提醒做跨标签稳定性对照 |

### V4-A Full169 blind simulation（历史研究方案）

- 独立的zero-shot Full169模拟设计，`max_iterations=1`，不加载Fixed系列Few-shot。
- Evaluator → Faithful backward → TextualGradientDescent → Structural validator → Frozen prediction。
- 必须区别于**当前Final默认2-shot**的实际条件。
- 历史研究代码可参考 [hero-textgrad-research/clean-main](https://github.com/dzwhala0128/hero-textgrad-research/tree/clean-main)。

### 历史实验使用原则

- 实验结论须标出数据切分、模型、Prompt、Few-shot、优化迭代次数、Gold离线评价。
- Accuracy/F1不得与 `optimized/no_change/fallback` 工程状态相混。
- 用户报告过的结论不等于已由当前环境独立复算。

## B. Final 独立工作线的历史快照（来自此前用户终端输出）

**2026-11-06**

- `--count 169 --af original --preflight` 返回 `valid=true`。
- 报告169条冻结VP、每条10 QA，默认2-shot，初始VP与模型调用均为0。
- `scripts/check_env.sh` 被整包哈希清单中的文档项阻断（先是清理说明，后是许可说明）。用户已要求开发项目不校验无关文档。
- 已建议将哈希检查限定为关键输入数据；**后续执行结果尚未收到**。
- **还不能将任何历史全量169优化结果写成Final已经真实运行完成。**

## C. 实验记录模板（注明所用项目/分支）

每次产生可核实新结果时，追加：

```text
日期/时区：
实验run-id：
研究项目/仓库/分支（默认 clean-main）及源码版本/变更文件：
数据集/样本选择：
AF/QA方案：
Evaluator / Backward / Optimizer模型及服务配置：
Few-shot与Prompt版本：
预检结果：
真实执行结果（optimized/no_change/fallback/错误）：
冻结产物：
Gold离线评价（如有）：
结论与下一步：
证据来源（日志文件、提交链接或用户终端输出）：
```

## D. 当前研究主线与待办

1. 用户默认以 `hero-textgrad-research/clean-main` 开展 HerO × TextGrad 实验；实验结论、运行路径及指标必须按该分支实际协议核对，不能沿用 Final 快照。
2. clean-main 新图形化工作台 V1.0 现处全量 ZIP 已交付、Server01 部署及模型端到端验收待完成阶段，见 [WORKBENCH_CLEAN_MAIN_V1.md](WORKBENCH_CLEAN_MAIN_V1.md)。
3. 旧工作台对齐 Final 的历史未完成事项保留在 [WORKBENCH.md](WORKBENCH.md)，除非用户明确切换到 Final，否则不作为当前优先任务。
