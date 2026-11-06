# HerO Visual Workbench v1.0｜独立 clean-main 研究工作台

> 更新：2026-11-07（亚洲时区）｜发行包日期：2026-11-06  
> **状态：独立完整源码 ZIP 已在 ChatGPT 会话中交付；尚未获得 Server01 部署或真实 GPU/vLLM 端到端验收反馈。**  
> 本文件记录**当前科研实验主线 clean-main 所配套的新工作台工作线**。用户2026-11-07再次明确：**当前研究实验也仍以 clean-main 为主线**，而 [FINAL.md](FINAL.md) 属于独立/较早工作线；勿与 [WORKBENCH.md](WORKBENCH.md) 的旧工作台→Final 对齐计划混同。

## 1. 最新用户明确指令与工程边界

- **本工作台的源码基准**：[hero-textgrad-research / clean-main](https://github.com/dzwhala0128/hero-textgrad-research/tree/clean-main)，而非 `Final` 或 `Hero-TTO`。在讨论/修改此工作台前须读取 `clean-main` 实际逻辑；用户不希望被反复要求贴出项目代码。
- **工作台独立部署目录**：服务器 `project/Hero-view/`。用户明确说明**该目录是空的**，仅作为工作台存放位置，助手无法直接访问 Server01；不要假设服务器已有旧版工作台，也不要发送一串增量补丁。
- **交付方式**：制作包含全部源文件的独立 ZIP，用户上传并解压到空 `Hero-view`；不得把未上传 ZIP 当成服务器已部署。
- **研究源码隔离**：工作台单独维护前/后端、实验记录、配置；不直接修改研究仓库、不打包权重/个人配置/密钥。
- 用户已**一次性确认并冻结设计决策**，以后不要再重复用一连串 A/B/C 逐项询问。新增实质性需求或明确有冲突的约束再讨论。

## 2. 已冻结的产品需求（用户明确确认）

| 领域 | 约定 |
|---|---|
| 视觉与导航 | 浅米白、高留白、简洁图标/小节点、连线；左侧导航 + 中央图形画布 + 右侧快速编辑抽屉 + 可全屏的阶段编辑器 |
| 画布 | 标准 TextGrad 流程图 + 自由实验图；支持多个同类节点、复制、类型兼容的接线、A/B 对照 |
| 核心阶段 | Evaluator、Native Backward、TextualGradientDescent Optimizer 均可独立编辑、独立执行、独立查看请求/响应 |
| 研究操作入口 | **从 Optimizer 优先开始探索**手工 Gradient 对 Verdict/Justification 的影响，再向 Backward、Evaluator 回溯 |
| 原生/自定义 | Native TGD、有效计算图 Native Backward、Custom 请求**分别标明来源**；不能将直接模型请求冒充 TextGrad 反传 |
| 编辑与版本 | 草稿自动保存、人工保存正式不可变版本、运行冻结输入快照；原始 Evaluator Feedback 只读，复制生成可编辑反馈 |
| 实验矩阵 | Claim × Gradient 版本（Optimizer）和 Claim × Feedback 版本（Backward）；每个 Claim 具有自己的原始/人工版本，可选择任意实验单元 |
| 数据传递 | Backward 梯度多选批量导入 Optimizer；严格按 Claim ID 匹配，仅导入不自动运行；同一源 run 与内容快照重复导入跳过 |
| 实验对照 | 冻结 Initial VP、原生梯度 Optimizer 结果、人工梯度 Optimizer 结果；Verdict 目标命中和 Gold 准确率必须明确区分 |
| 人工干预 | 标准复现与人工干预完整流程并存；人工反馈/梯度来源有明确标识；运行前预设替换或断点时修改均可 |
| 调试 | 每阶段执行前/后断点；单步、继续、重试分支；批量任务默认不阻塞、可对指定 Claim 设交互断点 |
| 批量与失败 | 默认串行、并发可配；失败记载后继续、不自动重试，人工重试生成独立 attempt；暂停只停止派发新任务 |
| 配置和资源 | 每阶段独立服务/模型/生成参数，可用统一默认；支持 Claim/QA/AF/冻结 VP 导入与人工编辑，但需标识来源 |
| 首版优先级 | 新画布 → Optimizer 独立实验 → Backward/Evaluator → 矩阵/调试/追溯 → 打包部署 |

补充关键语义：
- 不同 Claim 的**具体 Gradient/Feedback 文字**不同，不能在实验矩阵里强行共享同一条原始梯度。
- `Expected Verdict` 在实验中是**分析元数据**，默认按 Claim 固定；不是 Gold，不能自动出现在模型请求中。
- Native Backward 不能只靠一段反馈文字宣称保存了 TextGrad 原始计算图；重放需要重建并核对可反向传播的依赖。
- Optimizer 语义更新 Verdict 是 TGD 的职责；结构 Validator 只负责格式及边界校验，不代替模型做标签选择。
- **无金标签运行**与冻结后 Gold 离线评价隔离；用户偏好中文术语“无金标签/不使用金标签”。

## 3. 页面架构（已冻结）

1. **工作流画布**：标准链与自由图，Claim/QA/AF → 冻结 VP → Evaluator → Backward → Optimizer → Validator/Results。节点快速编辑与全屏。
2. **阶段实验室**：左样本/版本，中详细输入及完整请求，右原始输出/解析/Diff/Trace。
3. **实验矩阵**：Optimizer 的 Claim×Gradient，Backward 的 Claim×Feedback，单元格选择、批处理和结果对照。
4. **结果与溯源**：基线/干预结果对比、Justification 差异、来源链 E run→Feedback→B run→Gradient→O run。
5. **运行监控**：队列/并发/失败/重试、前后断点、暂停派发、实验快照。
6. **资源与设置**：运行数据导入、Claim/QA/AF/冻结 VP、Prompt/Parser/协议版本、模型配置。

## 4. 2026-11-06 V1.0 交付事实与验证边界

**已由本会话实际检查的发行包：**
- 文件名：`HerO-Visual-Workbench-v1.0-full.zip`，会话中创建后提供下载；顶层直接是源码文件，**不是旧版差分补丁**。
- 容器内 ZIP 检查：25 个归档条目，约 65 KB；`unzip -t` 返回无损坏。
- ZIP SHA-256：`10d22417d5278eeba2f63c2f1d63c408993dbaf5c491b98788ede45020996d1e`（用于辨认这次交付工件；记忆仓库**不存 ZIP 本体**）。
- 文件：`frontend/index.html`、`frontend/assets/app.js` 与 `style.css`；`backend/{app,jobs,stages,store}.py`；`tests/test_api.py`；`install.sh`、`run.sh`、`stop.sh`、`.env.example`、`release_manifest.json`、部署及使用文档。
- `release_manifest.json` 记载的研究源码审计基线：`clean-main@4242ba01174bcac5bada2e9c55eda1c9cd80fd10`，不保证服务器当前 checkout 仍处于此提交。
- 发行说明报告：Python/JS 语法检查、12 项离线 API 测试、Chromium 离线 UI 画布/编辑器/矩阵交互检查。**这些是发行包记录的测试结果，不等于 Server01 验收**。
- 发行包描述的功能：双画布、独立 Evaluator/Backward/Optimizer、Native/Custom 模式、反馈/梯度库、矩阵、任务和调试/历史。具体真实服务行为要以服务器部署后的运行日志确认。
- 此版本的 Native Backward 是 **Forward Replay 受控重建**，与真实新鲜 Evaluator 前向执行必须分开统计；“完整流水线自动编排”不代表某个历史论文协议的逐字段严格复现。

**尚未验证或尚未做：**
- 用户是否已上传 ZIP 到 `Hero-view`、已安装依赖或启动网页：**未知**。
- Server01 上实际模型服务可访问、真实 GPU/vLLM、真实 Native Backward/TGD 完整执行：**未验收**。
- 完整的历史实验协议切换、论文级复现、官方 AVeriTeC Gold 离线评分：**不能宣称已完成**。
- 不能把本次工作台视为已经与 `Final` 对齐；现有 [WORKBENCH_INTERFACE_MAP.md](WORKBENCH_INTERFACE_MAP.md) 是**另一条旧工作台→Final 静态映射线**。

## 5. 下次开展工作时的优先行动

1. 用户上传并解压该**完整 ZIP** 到服务器 `Hero-view`；设置 `HERO_RESEARCH_ROOT` 指向实际 `clean-main` 项目 checkout，确认单独的模型服务地址及本地回环访问（工作台没有公网鉴权，不应直接公网暴露）。
2. 先验证页面、数据导入、独立 Optimizer 及实际 vLLM；再验证 Native Backward、Evaluator、矩阵和批量运行。产生实际错误以 Server01 日志为准，不仅凭旧记忆诊断。
3. 如用户后来要求将**新**工作台转为 `Final` 适配器，必须视为**新的明确任务**：先取得或核对 Final 当前可访问的源码、参数及真实输出，不要静默把工作台的 clean-main 后端当作 Final 后端。
4. 后续版本更新应写明日期、发行包版本、代码来源/提交、已验证与未验证事项、用户实际部署结果；避免将演示结果、单测通过或“目标标签命中”写成真实科研指标。

## 6. 相关文档

- 新版需求冻结文件：会话生成 `HerO_Visual_Workbench_Design_Frozen_v1_20261106.md`（用户会话工件，不在 GitHub 记忆库中备份）。
- 新版 ZIP：`HerO-Visual-Workbench-v1.0-full.zip`（用户会话交付工件，SHA 见上；不是 GitHub Memory 中的源码）。
- **当前科研主线：** [CLEAN_MAIN.md](CLEAN_MAIN.md)（`clean-main` 研究工程）；[FINAL.md](FINAL.md) 是独立/较早的工作线记录。
- 旧工作台向 Final 对齐的历史计划：[WORKBENCH.md](WORKBENCH.md)、[WORKBENCH_INTERFACE_MAP.md](WORKBENCH_INTERFACE_MAP.md)。
