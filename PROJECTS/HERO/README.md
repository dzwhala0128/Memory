# HERO｜HerO × TextGrad 研究记忆

> 更新时间：2026-11-07 · **当前科研实验主线：`hero-textgrad-research / clean-main`**  
> 本目录记录 HerO × TextGrad 科研工作线。**用户2026-11-07明确指出以 `clean-main` 为主线做实验**；代码、Prompt 和模型设置默认参考该分支当前文件，不再把 `Final` 视作唯一现行开发项目。过去实验须保留来源，不可混写。

## 阅读导航

| 文件 | 内容 |
|---|---|
| [CLEAN_MAIN.md](CLEAN_MAIN.md) | **当前主线**：clean-main 仓库与分支、研究边界及默认后续动作 |
| [FINAL.md](FINAL.md) | **独立/较早工作线**：Final 的历史流程、记录与待验证事项 |
| [RESEARCH.md](RESEARCH.md) | AVeriTeC、HerO、Native TextGrad、四分类规则、Gold隔离及研究动机 |
| [EXPERIMENTS.md](EXPERIMENTS.md) | 历史研究实验与Final测试待验证清单，明确区分版本 |
| [WORKBENCH_CLEAN_MAIN_V1.md](WORKBENCH_CLEAN_MAIN_V1.md) | **当前新工作台**：clean-main V1.0 全量 ZIP、冻结设计、部署待验收 |
| [WORKBENCH.md](WORKBENCH.md) | 旧工作台与 Final 对齐的历史计划（非当前工作台） |

## 当前已确定的工程方向

- **当前实验/代码主线：** [hero-textgrad-research 的 clean-main 分支](https://github.com/dzwhala0128/hero-textgrad-research/tree/clean-main)。默认以其真实代码和运行协议为准，优先核实当前提交。
- **新 Web 工作台：** 独立存放于服务器空目录 `project/Hero-view/`，以 clean-main 为源码基准；V1.0 全量 ZIP 已交付，服务器部署与真实 GPU/vLLM 尚待验收。
- **Final：** 独立或较早的工作线，已有科研与预检资料保留在 Final 专题；不能将 Final 的 Qwen3.5-9B、固定 2-shot 或其他配置自动归为 clean-main 的最新设置。
- **共同科研边界：** 初始 VP 冻结、Gold 与运行时隔离、Native Backward/TGD 语义严格区分、结构验证不替模型判标签。

## 外部代码来源边界

- **当前主线代码：** [hero-textgrad-research · clean-main](https://github.com/dzwhala0128/hero-textgrad-research/tree/clean-main)。
- 可能存在另一份公开仓库 [Hero-TTO](https://github.com/dzwhala0128/Hero-TTO) 供资料参考；**不能推断其内容就是服务器Final的最新版**。
- `Final` 的服务端改动如未同步 GitHub，仅在明确讨论该工作线时依用户提供的运行输出或可用工具核验；不得替代 clean-main。
