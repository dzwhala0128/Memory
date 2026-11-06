# 新会话读取与主动维护启动指令

将下面这段文字设置到 ChatGPT 自定义指令，或在新的ChatGPT/Codex会话开始时发送：

```text
我的公开科研记忆仓库：https://github.com/dzwhala0128/Memory
科研讨论开始时，请先读取main分支的README.md和SYSTEM/MEMORY_POLICY.md，
然后根据本次任务读取PROJECTS下相应项目的README和专题文档。

对当前HerO × TextGrad，唯一现行开发项目是Final。
凡是在会话中完成可核实的重要实验、代码架构或设计决策变更，
当GitHub连接可用时，请按SYSTEM/MEMORY_POLICY.md主动更新
Memory对应分类下的文件，无需每次向我重复询问授权。
只保存适合公开的科研/工作信息，不保存隐私和密钥；
区分历史实验、用户报告、待验证与当前代码事实；
提交完成后提供GitHub链接。不可访问仓库则明确说明。
```

## 为什么仍建议放入自定义指令？

- GitHub里的 `SYSTEM/` 是**用户定义的仓库分类**，不是OpenAI后台系统提示。
- 新聊天不一定自动知道该仓库；自定义指令是更实际的“新会话触发入口”。
- 即使配置了入口，也只有新会话真实访问GitHub且具备写入权限时，才能执行自动写回。
- 它不等于任何不经交互的后台定时同步。
