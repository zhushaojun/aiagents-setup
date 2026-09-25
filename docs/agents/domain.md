# Domain Docs

工程技能在探索本仓库时如何消费领域文档。

## 探索之前先读这些

- 仓库根目录的 **`CONTEXT.md`**，或
- 若存在仓库根目录的 **`CONTEXT-MAP.md`**：它指向每个上下文各自的 `CONTEXT.md`。读取与当前主题相关的每一个。
- **`docs/adr/`**：读取与你即将动手的区域相关的 ADR。多上下文仓库中还要检查 `src/<context>/docs/adr/` 里上下文范围内的决策。

若这些文件不存在，**静默继续**。不要指出它们缺失，也不要主动建议创建。`/domain-modeling` 技能（经由 `/grill-with-docs` 和 `/improve-codebase-architecture` 触达）会在术语或决策真正被解决时按需创建它们。

## 文件结构

单上下文仓库（本仓库属于此类）：

```
/
├── CONTEXT.md
├── docs/adr/
│   ├── 0001-example-decision.md
│   └── 0002-example-decision.md
└── （本仓库为纯文档仓库，正文 markdown 位于仓库根目录）
```

多上下文仓库（根目录存在 `CONTEXT-MAP.md`）：

```
/
├── CONTEXT-MAP.md
├── docs/adr/                          ← 系统级决策
└── src/
    ├── ordering/
    │   ├── CONTEXT.md
    │   └── docs/adr/                  ← 上下文相关决策
    └── billing/
        ├── CONTEXT.md
        └── docs/adr/
```

## 使用 glossary 的词汇

当你的产出要命名某个领域概念（issue 标题、重构提案、假设、测试名）时，使用 `CONTEXT.md` 中定义的术语。不要漂移到 glossary 明确回避的同义词。

如果你需要的概念还不在 glossary 里，这是一个信号：要么你在发明项目不使用的语言（重新考虑），要么存在一个真实缺口（记下来交给 `/domain-modeling`）。

## 标记 ADR 冲突

如果你的产出与既有 ADR 矛盾，明确暴露出来，而不是悄悄覆盖：

> _Contradicts ADR-0007 (event-sourced orders), but worth reopening because…_