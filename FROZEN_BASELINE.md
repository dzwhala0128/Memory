# 当前可运行项目：HerO_TextGrad_Advisor_FrozenBaseline

> 最近已知状态：2026-11-06（北京时间）。下面记录的是已经在对话中确认的实际工程运行情况，**不是实时读取服务器状态**。以最新代码/输出为准。

## 1. 核心原则（必须保留）

- **初始 Baseline VP 已冻结**：每个样本直接读取已存在的 `Justification + Verdict`，不让Qwen重新做初始VP。
- 当前调用的 TextGrad阶段模型为 **Qwen3.5-9B**，用户的 vLLM/OpenAI兼容服务在 **8005端口**，常用模型标识 `qwen3.5-9B`；上下文预算曾配置为 **8192**。
- **2-shot** 为当前默认 Evaluator 示例形式，使用 `fewshot/fixed2.jsonl`；新创建的6-shot文件不会自动生效。
- 优化执行顺序：Evaluator → Native Backward（仅实质问题时）→ TextualGradientDescent Optimizer → 结构验证 → 冻结最终预测 → 离线对比Gold。
- 当无实质问题时 `status=no_change`，属于正常成功路径；`status=optimized` 代表优化执行且通过结构验证；`status=baseline_fallback` 代表异常回退，不能视作优化成功。
- 不应修改原始 HerO 复现源码来“修”当前统一TextGrad流程。

## 2. 关键文件与目录

| 路径 | 作用 |
|---|---|
| `run.sh` | 导师版统一入口，`qwen`执行，`compare`离线对比 |
| `pipeline/unified.py` | 读取固定样本、组装Prompt、统一阶段调用、冻结 |
| `pipeline/qwen_engine.py` | 8005模型请求、Token预算与请求追踪 |
| `pipeline/native_backward.py` | 调用原生 `loss.backward()` |
| `pipeline/validator.py` | 结构/合法四分类验证，不能替代语义优化 |
| `pipeline/compare_metrics.py`、`pipeline/evaluate.py` | 离线指标计算 |
| `configs/qwen9b.json` | 当前9B模型/各阶段的主要配置 |
| `prompts/evaluator_system.txt` | Evaluator系统规则 |
| `prompts/evaluator_user.txt` | 输入模板（Claim / AF / QA / 初始Solution）与Few-shot插入标记 |
| `prompts/native_backward_system.txt` | Native Backward规则 |
| `prompts/native_optimizer_system.txt` | Optimizer规则 |
| `prompts/optimizer_constraints.json` | Optimizer的结构约束 |
| `fewshot/fixed2.jsonl` | 现行默认2-shot |
| `fewshot/fixed6.jsonl` | 近期生成的6-shot文件，可选择上传，当前不自动启用 |
| `data/targeted40/` | 40条声明三套AF/QA输入 |
| `data/full169/runtime.json` | 169条冻结VP + 1690组QA |
| `data/full500/runtime.json` | 500条冻结VP + 5000组QA |
| `data/evaluation_only/` | 离线评价数据，不能混进运行时Prompt |
| `textgrad_native/` | Native TextGrad源码 |
| `hero/` | 独立保留的原始HerO复现代码 |
| `outputs/<run-id>/` | 每次运行的阶段追踪、冻结结果、后续离线指标 |

部分 `configs/*.yaml` 和 `docs/` 是历史实验配置/说明，**目前 `run.sh qwen` 的关键配置在 `configs/qwen9b.json`**；请以最新源码验证细节。

## 3. AF/QA输入的区分

- `--af human63`：40条Targeted样本范围内的人工拆分AF方案（共63条AF）与筛选QA。
- `--af merged67`：同一40条目标样本范围内的合并方案（共67条AF）与筛选QA。
- `--af original`：不使用拆分AF，按原始QA输入；169/500使用此模式。
- **63/67是方案里AF总量，不是每个样本各有63/67条AF。**
- 1/10/40是Targeted40子集选择；169和500为各自固定数据集，不应简单看作Targeted40的延伸。

## 4. 已使用的运行命令

在已配置好Python环境和运行中的9B服务的项目根目录：

```bash
# 单样本，曾真实验证过的Claim 59
bash run.sh qwen --count 1 --claim-id 59 --af human63 --run-id demo_1_59 --freeze

# Targeted40中前10条（人工拆分AF）
bash run.sh qwen --count 10 --af human63 --run-id demo_10 --freeze

# Targeted40中40条（人工拆分AF）
bash run.sh qwen --count 40 --af human63 --run-id demo_40 --freeze

# 169条，原始QA、不拆分AF
bash run.sh qwen --count 169 --af original --run-id demo_169 --freeze

# 只做输入/环境预检（无模型调用）
bash run.sh qwen --count 169 --af original --preflight

# 冻结后计算离线指标（不会重跑模型）
bash run.sh compare --run-id demo_169
```

`--run-id` 应每次使用新名字，避免覆盖；完整生产运行前先做 `--preflight`。169条已冻结结果可单独执行 `compare`。

## 5. 2-shot 与 6-shot 的具体情况

历史检查发现：

- 2-shot由 `pipeline/unified.py` 中固定的 `fewshot/fixed2.jsonl` 路径读取，通过 `fewshot_text()` 渲染，并注入 `prompts/evaluator_user.txt` 的 `<<FEW_SHOT_EXAMPLES>>` 标记。
- 现有2-shot示例训练索引是 **165和6**。
- 已基于历史Fixed-6整理出6-shot，训练索引 **1559、1247、165、1251、6、62**，历史Fixed-6出处在 `hero-textgrad-research` 的 `clean-main` 分支；其中4个 `MATERIAL_ISSUE`、2个 `NO_MATERIAL_ISSUE`。
- **只是把 `fixed6.jsonl` 放到 `fewshot/` 不会启用**；此前代码还检查默认2-shot索引，若要切换，需要审查固定索引逻辑和提示词（曾有“The following two demonstrations”字样）。
- 尚未确认现行公开仓库已加入6-shot切换参数，**不要捏造 `--fewshot 6` 为可用参数**。

## 6. 已成功验证的真实运行

在早先对话中由用户在Server01执行并报告：

- `check_env.sh`通过；Python 3.11环境、Native TextGrad可用；项目离线单元测试先前通过18项。
- `--count 500 --preflight` 通过，报告500条冻结VP、各10 QA、初始VP调用0、模型调用0、默认Few-shot 2。
- Claim 9：Evaluator `NO_MATERIAL_ISSUE` → `status=no_change`、未调用Optimizer。
- Claim 59：Evaluator `MATERIAL_ISSUE` → Native Backward与Optimizer执行 → 结构验证通过 → `status=optimized`；输出字段显示修改后的Verdict为 NEE。
- 因此两条关键分支**在工程上均已验证**，但不代表整套数据集100%无错误或模型标签正确。

## 7. 2026-11-06 169条真实批量实验

批次：`night_20261106_010401_169_original`。

| 状态 | 数量 | 说明 |
|---|---:|---|
| `optimized` | 92 | 执行优化并通过结构检查 |
| `no_change` | 72 | 保留原Solution，无需改动 |
| `baseline_fallback` | 5 | 发生异常，回退原始冻结Baseline |
| 合计 | **169** | 所有预测均写出并冻结 |

回退Claim：**135、192、77、57、399**。

诊断：

- Claim 135、57：Evaluator模型输出未正常停止，报 `Truncated/non-stop response; baseline fallback`。
- Claim 77、399：Native Backward阶段模型输出未正常停止，同类报错。
- Claim 192：Optimizer输出的 `verdict` 未通过四分类结构检查（`Invalid four-class verdict`）。
- **尚未根据原始 `finish_reason` / `usage` 确认到底是token上限还是其他原因**；不可把“推测长度截断”写成已证实事实。
- 总结：正常路径164/169，约97.04%；**不是169/169无错误**。
- 此批量脚本由于严格结果检查，将包含回退样本的169条标记 `RESULT_CHECK_FAILED`，并跳过自动 `compare`；离线指标仍可以在冻结后独立计算，需保留5条回退的事实。
- 当时用户决定先不处理5条回退，优先接受大多数流程跑通的工程状态；未来如要恢复，必须保留原始冻结结果，不能悄悄覆盖。

## 8. 研究/工程继续开展时需注意

1. 任何新实验标明具体：仓库和分支、代码版本、输入集、AF方案、Few-shot、Prompt、模型、预算、Gold隔离、运行ID。
2. 优化失败与“算法选择no_change”是不同情形；必须在报告中分开统计。
3. 增加6-shot、重试失败样本或修正格式时，应使用新运行版本，先做小规模预检。
4. 用户目前更关注导师能按命令复现与代码结构清晰；**不要在仅要求工程使用说明时强行转入论文指标争论**。
