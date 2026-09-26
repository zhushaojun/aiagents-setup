# AGENTS.md

本仓库为 AI Agent 配置教程文档仓库（纯 Markdown + 图片）。

## 文档约定

- **正文在仓库根目录**（与 `README.md` 同级）。`docs/` 只放给智能体看的仓库工作流规范，**不要把正文教程挪进 `docs/`**。
- **文件名用 `NN-kebab-case`**：`NN` 两位数字就是推荐阅读顺序（`00` 总览 → `10` 附录），名字用英文小写连字符（如 `06-claude-code.md`），文档 H1 标题保持中文。`README.md` / `AGENTS.md` 不编号。新增一篇教程时，取一个未被占用的数字，并同步更新 README 第 3 节的文档地图。
- 文件改名/改号时，必须同步更新所有交叉链接（`rg` 一遍旧名确认无残留）。
- **截图放 `images/`**，文件名为 `<工具>-<用途>.png`（如 `codex-vscode-extension.png`）。正文用 `![图里能看到什么](images/xxx.png)` 引用，alt 文本要写具体内容而不是“截图”。
- **缺图的位置用 HTML 注释占位**：`<!-- TODO 截图：要拍什么 -->`。用 `rg 'TODO 截图' -g '!AGENTS.md'` 找出全部待补截图（本文件这一行是说明，不算）。

## Agent skills

### Issue tracker

Issues live in this repo's GitHub Issues (via the `gh` CLI). See `docs/agents/issue-tracker.md`.

### Triage labels

The five canonical triage roles map 1:1 to same-named label strings. See `docs/agents/triage-labels.md`.

### Domain docs

Single-context: `CONTEXT.md` + `docs/adr/` at the repo root. See `docs/agents/domain.md`.
