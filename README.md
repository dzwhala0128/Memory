# Research Memory｜跨会话研究记忆索引

> 更新：2026-11-06 · 默认分支 `main` · 当前仓库为 **Public**
>
> 作用：为 ChatGPT / Codex 提供**可阅读、可维护、可分领域扩展**的工作上下文。这里的规则是仓库协作协议，**不是 ChatGPT 平台系统指令，也不是后台自动监听服务**。

## 优先阅读顺序

1. [SYSTEM/README.md](SYSTEM/README.md) — 跨项目的记忆规则与自动维护协议。
2. [SYSTEM/MEMORY_POLICY.md](SYSTEM/MEMORY_POLICY.md) — 分类、隐私、去重、验证、主动更新规范。
3. [PROJECTS/README.md](PROJECTS/README.md) — 各科研/工作项目的独立记忆。\n4. [TOPICS/README.md](TOPICS/README.md) — 不依附具体项目的非私人专题知识。
5. [PROJECTS/HERO/README.md](PROJECTS/HERO/README.md) — 当前 HerO × TextGrad 研究索引。
6. 根据正在做的任务选择 `PROJECTS/HERO/` 下的对应专题。

## 当前有效项目

| 领域 | 当前项目 | 阅读入口 | 状态 |
|---|---|---|---|
| HerO 事实核查与 TextGrad | **Final** | [PROJECTS/HERO/FINAL.md](PROJECTS/HERO/FINAL.md) | 当前开发工作区 |
| HerO 理论与研究 | 研究长期背景 | [PROJECTS/HERO/RESEARCH.md](PROJECTS/HERO/RESEARCH.md) | 持续维护 |
| HerO 实验 | 实验与验证记录 | [PROJECTS/HERO/EXPERIMENTS.md](PROJECTS/HERO/EXPERIMENTS.md) | 历史与现状分开记录 |
| HerO Web 工作台 | 对齐 Final 逻辑 | [PROJECTS/HERO/WORKBENCH.md](PROJECTS/HERO/WORKBENCH.md) | 下一阶段任务 |

## 新会话快捷指令

> 请读取 https://github.com/dzwhala0128/Memory 的 `main` 分支，先看 `SYSTEM/MEMORY_POLICY.md`，然后按当前任务读取 `PROJECTS/` 对应项目的 README 与专题文件。当前 HerO 开发项目只以 `Final` 为准。对科研重大结果、架构变化和已验证决策，在本次会话可调用 GitHub 时主动更新对应记忆文件，无需逐次征求确认；请先核对证据，保持公开仓库安全，并在写入后报告提交结果。若无权限或无法验证，明确说明。

## 使用边界

- **跨会话自动同步的前提**：新会话必须得知并读取本仓库的协议；GitHub 文档本身不会自动注入每个新会话。可将上述快捷指令放入 ChatGPT 的自定义指令以减少手动提醒。
- 不要把过去项目的运行结果自动算作 `Final` 的真实实验结果。
- 不保存私人经历、身份识别信息、服务器口令、API Key、SSH密钥或内部日志。
- 未来添加其他科研/工作项目时在 `PROJECTS/` 下另建目录；不依附项目的非私人主题放在 `TOPICS/`，不要混进 HerO 专题。
