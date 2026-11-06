# Memory｜HerO × TextGrad 科研上下文索引

> 更新日期：2026-11-06（北京时间）  
> 用途：供未来 ChatGPT / Codex 会话快速恢复项目背景，减少重复解释。  
> **注意**：这是人工整理的研究上下文快照，不是 ChatGPT 内部记忆的完整导出，不会自动随新对话同步。当前仓库为 **Public**，只保存适合公开的信息。

## 🚀 新会话从这里开始

在新的 ChatGPT 会话中发送：

> 请先读取 GitHub 仓库 `https://github.com/dzwhala0128/Memory` 的 `main` 分支，依次阅读 `README.md`、`RESEARCH_CONTEXT.md`、`FROZEN_BASELINE.md`、`EXPERIMENT_HISTORY.md` 和 `COLLABORATION_GUIDE.md`。这是我的科研项目长期上下文，请依据其中的信息理解当前项目，但涉及实时代码、最新实验结果时，以仓库最新源码和实际输出为准。先告诉我你理解的项目现状与未解决事项。

也可以在 Codex 中使用同样提示；如果目标是改代码，请同时读取目标项目仓库相应分支的源码。

## 📁 文档目录

| 文件 | 内容 |
|---|---|
| [RESEARCH_CONTEXT.md](RESEARCH_CONTEXT.md) | 研究问题、HerO与TextGrad架构、数据集、四分类逻辑、实验原则 |
| [FROZEN_BASELINE.md](FROZEN_BASELINE.md) | 当前导师版运行项目、2-shot/6-shot、模型、运行命令、169条实验真实结果 |
| [EXPERIMENT_HISTORY.md](EXPERIMENT_HISTORY.md) | 历史实验代号、核心发现、V4-A盲模拟、研究演进 |
| [COLLABORATION_GUIDE.md](COLLABORATION_GUIDE.md) | 用户沟通偏好、代码阅读要求、结果汇报注意事项 |

## 🔗 相关代码仓库

- [Hero-TTO](https://github.com/dzwhala0128/Hero-TTO)：近期导师版 FrozenBaseline 项目（以实际提交为准）。
- [hero-textgrad-research · clean-main](https://github.com/dzwhala0128/hero-textgrad-research/tree/clean-main)：此前 HerO × TextGrad 研究项目及历史实验实现。
- [Memory](https://github.com/dzwhala0128/Memory)：跨会话研究上下文整理。

## ⚠️ 使用说明

1. 本仓库记录**已讨论的事实、设计决定、历史报告和待核验事项**；不是所有指标都经过独立复算。
2. 旧版研究代码与近期 FrozenBaseline 工程代码可能不同。修改或讨论某份代码时，**先确认仓库、分支、文件和提交版本**，不要把历史配置当现行实现。
3. 关键方法原则：**初始 HerO VP 冻结；后续 TextGrad 使用证据和反馈优化；运行中不使用金标签；结构验证器只检查结构合法性。**
4. 任何 Gold 评估只在最终预测冻结之后进行，避免数据泄漏。
5. 不要上传账号凭据、隧道认证信息、个人聊天记录、私有地址或包含敏感信息的服务器日志。
6. 若想保存更全面的私人信息，请先将此仓库改为 Private，再明确选择需要保存的范围；**Private 也不应存储密码/API Key**。

## 🧭 当前阶段

截至 2026-11-06：已经搭建并实际运行 `HerO_TextGrad_Advisor_FrozenBaseline`，确认单样本既能触发 `NO_MATERIAL_ISSUE` 无修改路径，也能触发 `MATERIAL_ISSUE` 优化路径。最近的169条实验均完成冻结，92条 `optimized`、72条 `no_change`、5条 `baseline_fallback`。详见 [FROZEN_BASELINE.md](FROZEN_BASELINE.md)。

---
*上下文快照需要定期人工或在会话中请求更新；它本身不会自动捕获新聊天。*
