# HERO｜HerO × TextGrad 研究记忆

> 更新时间：2026-11-06 · **唯一现行代码工作项目：`Final`**  
> 本目录仅记研究主线和当前开发情况。代码、模型配置和脚本参数以 `Final` 真实文件为准；旧实验不得无依据归属于 `Final`。

## 阅读导航

| 文件 | 内容 |
|---|---|
| [FINAL.md](FINAL.md) | 当前项目 `Final`：目录、运行流程、已验证的工程现状与尚未解决的检查问题 |
| [RESEARCH.md](RESEARCH.md) | AVeriTeC、HerO、Native TextGrad、四分类规则、Gold隔离及研究动机 |
| [EXPERIMENTS.md](EXPERIMENTS.md) | 历史研究实验与Final测试待验证清单，明确区分版本 |
| [WORKBENCH.md](WORKBENCH.md) | 下一阶段：将网页工作台按Final项目逻辑对齐 |

## 当前已确定的工程方向

- 用户将个人后续实验全部以 `Final` 项目为中心展开。
- 不要把其他代码副本、历史运行记录或旧文件名当作当前开发入口。
- `Final` 采用预先冻结的初始VP；TextGrad阶段调用Qwen3.5-9B（原服务端口8005）；Evaluator—Backward—Optimizer—结构验证—冻结—离线评价。
- 用户计划将网页工作台的控件、模型调用、状态展示和数据流**逐项对应**到当前Final实际实现；目前这是下一阶段任务，**尚未报告完成**。

## 外部代码来源边界

- 既往研究参考代码：[hero-textgrad-research · clean-main](https://github.com/dzwhala0128/hero-textgrad-research/tree/clean-main)。
- 可能存在另一份公开仓库 [Hero-TTO](https://github.com/dzwhala0128/Hero-TTO) 供资料参考；**不能推断其内容就是服务器Final的最新版**。
- `Final` 的最新服务端改动若没有同步GitHub，必须以用户实际运行输出/现行文件为准；可用工具能直接访问时应主动核查，否则如实标记不可验证。
