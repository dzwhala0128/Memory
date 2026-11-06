# Final｜独立/较早工作线记录（非当前 clean-main 主线）

> 原始状态日期：2026-11-06（北京时间）。**2026-11-07 用户纠正：当前科研实验主线为 [`clean-main`](CLEAN_MAIN.md)，不是 Final。** 以下 Final 记录仅供独立/历史核对，不代表 clean-main 的当前配置或实验结果。本记录依据用户终端输出及此前讨论，不代表已登录服务器核查。

## 1. 项目身份与范围

- **项目名称**：`Final`
- **角色**：曾使用的独立科研代码工作区；保留其当时的实验设定和故障背景，不再标为用户当前唯一主线。
- **实际工作目录**：服务器 `.../project/Final/`；需要执行命令时由用户在当前终端切换到该目录，避免在公开记忆中保存个人账号目录。
- **改动原则（更新）**：用户后续科研实验默认以 `clean-main` 为准；仅当用户明确要求继续 Final 工作时，才针对 Final 检查或改动其自身源码。
- **重要**：历史运行结果不能在未重新运行或核对文件前宣称是在Final里产生。

## 2. 当前设计与真实数据流

```text
已冻结初始 VP（Justification + Verdict）
             │
       Claim + QA (+ 可选 AF)
             ↓
  Evaluator（含现行 2-shot）
        ├─ NO_MATERIAL_ISSUE → 保留原Solution
        └─ MATERIAL_ISSUE
                ↓
        Native TextGrad loss.backward()
                ↓
        TextualGradientDescent Optimizer
                ↓
        结构验证器（仅合法性）
                ↓
             冻结预测
                ↓
        Gold隔离的离线指标
```

- 初始VP不能通过9B重新生成；9B用于优化链路，不替换冻结基线。
- 默认Evaluator Two-shot在 `fewshot/fixed2.jsonl`；另有6-shot材料但**不能据此假设Final已支持自动切换**。
- Evaluator输出 `STATUS: MATERIAL_ISSUE` / `NO_MATERIAL_ISSUE` 与 `FEEDBACK`；AF仅用于澄清Claim、不能算独立证据。
- Optimizer负责语义层面改写Justification及必要的Verdict，Validator不负责选择正确标签。
- 最终预测先冻结、再离线用Gold算指标；运行阶段不得读目标金标签。

## 3. 关键文件索引（以现行Final源码为准）

| 相对路径 | 作用 |
|---|---|
| `run.sh` | 统一命令入口 |
| `pipeline/unified.py` | 读取冻结输入并调度Evaluator→Backward→Optimizer→结构检查/冻结 |
| `pipeline/qwen_engine.py` | OpenAI兼容9B调用，保存阶段请求和原始响应 |
| `pipeline/native_backward.py` | 调用Native TextGrad `loss.backward()` |
| `pipeline/validator.py` | 结构及四分类格式检查 |
| `pipeline/compare_metrics.py` / `pipeline/evaluate.py` | 冻结后的离线评价 |
| `configs/qwen9b.json` | Qwen3.5-9B与阶段生成预算 |
| `prompts/` | Evaluator、Backward、Optimizer外置提示词 |
| `fewshot/` | 实例示例文件 |
| `data/targeted40/` | 40条目标样本的AF/QA变体 |
| `data/full169/runtime.json` | 169条已冻结VP及原始QA |
| `data/full500/runtime.json` | 500条已冻结VP及原始QA |
| `textgrad_native/` | Native TextGrad源码 |
| `hero/` | 原始HerO流水线源码 |
| `scripts/check_env.sh` / `scripts/check_data.py` | 环境/数据检查 |
| `data/manifests/package_sha256.json` | 打包时期的哈希清单；不等于日常开发所需不变式 |

## 4. 运行方式（命令来源为既有入口，需以Final当前文件再次核实）

```bash
# 已进入Final项目根目录、Python环境就绪且9B模型服务启动

# 单条拆分AF：选择Claim 59
bash run.sh qwen --count 1 --claim-id 59 --af human63 --run-id final_one_59 --freeze

# 10条拆分AF
bash run.sh qwen --count 10 --af human63 --run-id final_10 --freeze

# 40条拆分AF
bash run.sh qwen --count 40 --af human63 --run-id final_40 --freeze

# 169条原始QA，不拆分AF
bash run.sh qwen --count 169 --af original --run-id final_169 --freeze

# 不调用模型的输入预检
bash run.sh qwen --count 169 --af original --preflight

# 冻结完成后离线比较
bash run.sh compare --run-id final_169
```

**注意**：`--run-id` 每次需使用独立名称，避免覆盖输出。上述全流程命令曾被用于同系列代码；截至本文件更新，仅有下面所述的Final预检来自用户实测，不能把其余命令写成Final已全部跑通。

## 5. Final在当前会话中实际报告的状态

用户进入 `Final` 后执行：

- `bash run.sh qwen --count 169 --af original --preflight`：**有效（valid=true）**，169条冻结VP全部存在，QA每条10组，默认模型Qwen3.5-9B，默认2-shot，`initial_vp_model_calls=0`，`model_calls=0`。
- `bash scripts/check_env.sh`：曾因固定哈希清单记录了未复制的文档 `CLEANUP_REPORT.md` 而中断；用户将它从清单中移除后，又因 `LICENSES.md` 缺失而中断。
- 已讨论把 `scripts/check_data.py` 中“所有包内文件哈希逐项校验”调整成“仅对关键数据的哈希校验”，让后续改Prompt/源码不被旧清单阻断；**尚未收到这一修改执行后 `check_env.sh` 全通过的证据**。
- 因此**不能宣称Final环境检查已完全通过或169条在Final中跑通**；进一步操作应先读取Final的现行脚本和结果再判断。

## 6. 近期约束与风险

- 同一个Qwen3.5-9B服务可服务多个客户端，但必须避免并行运行时GPU拥塞、服务中断。
- 现行默认2-shot；是否启用6-shot须核对Final代码的Few-shot路径/固定示例校验。
- `finish_reason` 不正常、结构输出不合法等故障可以导致baseline回退，不能把回退等价于模型判断“无需修改”。
- 开发版本允许改动Prompt/源码；不应因历史整包哈希清单阻止一切迭代，但仍需保护数据和Gold隔离。
- 这条关于旧工作台对齐 Final 的计划是历史背景。**当前独立新工作台面向 clean-main**，见 [WORKBENCH_CLEAN_MAIN_V1.md](WORKBENCH_CLEAN_MAIN_V1.md)。
