# PROJECTS｜按研究领域或工作项目分类

每个项目单独管理，避免未来不同研究方向、工具或工作资料互相覆盖。

| 项目ID | 名称 | 当前状态 | 入口 |
|---|---|---|---|
| `HERO` | HerO × TextGrad · AVeriTeC | 活跃；`Final`为现行开发项目 | [HERO/README.md](HERO/README.md) |

## 新增项目的约定

未来用户需要增加其他科研或非HerO工作知识时，按这种结构新增：

```text
PROJECTS/
├── HERO/
│   ├── README.md
│   └── ...
└── NEW_PROJECT/
    ├── README.md
    ├── CONTEXT.md
    ├── PROGRESS.md
    └── ...
```

- 新项目无需更改 `PROJECTS/HERO/`。
- 跨项目公共规则仍只放在 `SYSTEM/`。
- 新项目的README说明范围、当前活跃仓库/分支、术语和专题链接。
- 不为尚未正式开始的项目伪造实验结论。
