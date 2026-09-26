# Matt Skills（第三方技能包入门）

**进阶篇。** 装完 [Codex](03-codex.md) / [pi](04-pi.md) 之后再来看。它不装工具，只装"工作流程"。

一句话：**把资深工程师的做事步骤写成 Markdown 文件，让智能体每次都照着做。** 官方页面：<https://www.aihero.dev/skills>

---

# 1 它是什么、解决什么问题

## 1.1 先说"技能（skill）"是什么

一个技能就是一个目录，里面放一个 `SKILL.md`：

```text
tdd/
├── SKILL.md          # 说明：什么时候用、按什么步骤做
└── mocking.md        # 附带文件（可选）
```

`SKILL.md` 开头是几行 frontmatter：

```markdown
---
name: tdd
description: 用红-绿-重构的循环写测试。当用户要"测试先行"地做功能或修 bug 时使用。
---

# TDD

1. 先写一个会失败的测试……
```

关键机制：**智能体启动时只读到每个技能的 `name` + `description`（描述里的"什么时候用"就是路由依据），命中任务后才去读 `SKILL.md` 全文。** 所以装几十个技能，也不会把上下文撑爆。

## 1.2 Matt Skills 是什么

Matt Pocock（Total TypeScript 作者）把自己日常用的技能开源了：<https://github.com/mattpocock/skills>（MIT 协议；原教程快照为 25 个技能，当前数量以仓库为准）。它不是"提示词合集"，而是**一条从想法到上线的完整工程流程**：先把需求问清楚 → 写成规格 → 拆成工单 → 测试先行地实现 → 按标准和规格双向评审。

## 1.3 和我们已经装的东西是什么关系

| 层 | 是什么 | 例子 |
| --- | --- | --- |
| **技能（skill）** | 一段 Markdown 流程说明，教模型"按什么步骤做" | `tdd`、`code-review`、`to-spec` |
| **扩展 / 插件** | 会执行代码的能力，能加工具、加命令 | pi 的 extensions、Codex 的 plugins |
| **MCP** | 把外部服务接进来当工具用 | context7 查文档、浏览器自动化 |
| **`AGENTS.md` / `CLAUDE.md`** | 每个仓库的长期约定（说什么语言、哪些目录不许动） | 见 [pi](04-pi.md) 第 4.3 节 |

**技能最省事的一点**：它是纯文件，格式是跨工具通用的。一条命令就能把它同时装给 Codex / pi / Claude Code / OpenCode / Cursor / Copilot 等 40 多个工具，不需要为每个工具改一份配置。

---

# 2 前置要求

| 项目 | 要求 | 检查方法 |
| --- | --- | --- |
| Node.js | **≥ 22.19**（`01-prerequisites.md` 里已经装过） | `node -v` |
| 至少一个智能体 | Codex / pi / Claude Code / OpenCode 任一已跑通 | 完成该工具教程第 5 节的三层验证 |
| 网络 | 能访问 GitHub（安装器从 GitHub 拉技能仓库） | — |

> 安装器本体是 npm 包 `skills`，用 `npx skills@latest` 调用，**不需要全局安装**。

---

# 3 安装

## 3.1 交互式（第一次推荐）

```PowerShell
npx skills@latest add mattpocock/skills
```

交互安装时重点确认：

1. **装哪些技能**——按空格勾选，**务必勾上 `/setup-matt-pocock-skills`**（第 4 节要用）。
2. **装给哪些智能体**——它会自动探测你电脑上装了哪些工具（认配置文件目录），也可以手动选。

安装时还要确认并记住最终选择的范围：项目级安装应在目标仓库内执行，全局安装可显式加 `-g`。第 6 节按对应范围验证，不要用全局列表判断项目级安装是否成功。

## 3.2 非交互式（复制即用）

装全套、装到全局（所有项目都能用）、指定三个工具：

```PowerShell
npx skills@latest add mattpocock/skills -g -s '*' -a codex pi claude-code -y
```

| 参数 | 作用 |
| --- | --- |
| `-g` / `--global` | 装到用户目录；**不加就是项目级**（装进当前仓库，可以提交给团队共享） |
| `-s '*'` | 装全部技能；也可以写名字，如 `-s tdd code-review to-spec` |
| `-a` | 目标工具，可以写多个：`codex`、`pi`、`claude-code`、`opencode`、`cursor`、`github-copilot`…… |
| `-y` | 不再逐个确认 |
| `--copy` | 复制文件而不是建链接（软链接出问题时用，见第 8 节） |
| `--all` | 等价于"全部技能 × 全部工具 × 免确认" |

只装三个先试试：

```PowerShell
npx skills@latest add mattpocock/skills -g -s ask-matt tdd code-review -a codex pi -y
```

先看清单再决定：

```PowerShell
npx skills@latest add mattpocock/skills --list
```

## 3.3 装去了哪里

安装器先放一份"正本"，再在各工具的技能目录里建链接指向它：

| 位置 | 路径 | 说明 |
| --- | --- | --- |
| **正本（全局）** | `%USERPROFILE%\.agents\skills\` | 实际选装的技能目录；版本记录在 `%USERPROFILE%\.agents\.skill-lock.json` |
| **正本（项目）** | `<仓库>\.agents\skills\` | 项目级安装放这里，会跟代码一起提交 |
| Claude Code | `%USERPROFILE%\.claude\skills\` | 全局；项目级是 `.claude\skills\` |
| Codex | `%USERPROFILE%\.agents\skills\`；也可能有 `.codex\skills\` 链接 | 全局检查共享正本及实际生成的链接；项目级读取 `.agents\skills\` |
| pi | `%USERPROFILE%\.pi\agent\skills\` | 全局；项目级是 `.pi\skills\`，同时认 `.agents\skills\` |
| OpenCode | `%USERPROFILE%\.config\opencode\skills\` | 全局；项目级认 `.agents\skills\` |

> Windows 上这些链接是 **Junction（目录联接）**，用 `Get-ChildItem` 的 `LinkType` 列能看到，**不需要开开发者模式**。好处是更新只改"正本"一处，各工具跟着变。

## 3.4 Claude Code 的插件路线（可选，二选一）

Claude Code 还能把整套技能当插件装，只读、随作者更新自动升级：

```PowerShell
claude plugins install mattpocock-skills
```

**两条路线不要同时装**，否则每个技能会出现两份同名副本。

---

# 4 每个仓库跑一次：`/setup-matt-pocock-skills`

技能本身到处一样，只有"你仓库的规矩"不一样。所以每个仓库要跑一次这个一次性配置，它只问三件事：

| 问题 | 推荐答案 | 会被写成 |
| --- | --- | --- |
| issue 放在哪 | GitHub Issues / GitLab / **本地 Markdown**（`.scratch/<功能>/`，没远程仓库也能用）/ 其他 | `docs/agents/issue-tracker.md` |
| 标签叫什么 | 保留五个标准名 `needs-triage` / `needs-info` / `ready-for-agent` / `ready-for-human` / `wontfix` | `docs/agents/triage-labels.md` |
| 领域文档放哪 | 单上下文：根目录一个 `CONTEXT.md` + `docs/adr/`（多仓库才需要 `CONTEXT-MAP.md`） | `docs/agents/domain.md` |

它还会在 `CLAUDE.md` / `AGENTS.md` 里补一段 `## Agent skills`，指向上面三个文件。之后 `/to-tickets`、`/triage` 就知道该往哪里发、该贴什么标签，不必每次问你。

三个坑先记住：

1. **它不会自动创建 GitHub 标签。** `triage-labels.md` 只是"名字对应关系"，标签本身仍需在 GitHub 仓库中存在，使用标准名也不会自动创建。用 `/wayfinder` 之前，`wayfinder:map` 这类标签也要手工先建好。
2. **它写 `CLAUDE.md` 还是 `AGENTS.md` 只看"哪个文件存在"，不看你在用哪个工具。** 仓库里如果有从 Claude Code 时代留下的 `CLAUDE.md`，你又在用 Codex，那段就写进了 Codex 不读的文件——手动把 `## Agent skills` 段挪到 `AGENTS.md` 即可。
3. **配置写在仓库里，是给人看的 Markdown。** 想改就手改，不用再跑一次；换了 issue 管理方式，或技能升级后行为对不上，就再跑一次。


使用 GitHub Issues 时，在目标仓库内先检查标签。以下命令需要已安装并登录 `gh`，**仅查看，不创建**：

```PowerShell
gh repo view --json nameWithOwner --jq .nameWithOwner
gh label list --limit 1000 --json name --jq '.[].name'
```

确认仓库无误后，只为列表中确实缺失的标签运行创建命令。例如仅缺 `needs-triage` 时：

```PowerShell
gh label create needs-triage --color BFD4F2 --description '待分类的问题'
```

其余四个名称按表逐个处理；不要用 `--force` 覆盖已有标签。若技能要求额外标签，也先检查再创建。这里只给操作说明，不要求安装技能时自动修改远端仓库。


---

# 5 怎么用：按流程走

## 5.1 主线（新手先跑这一条）

| 顺序 | 技能 | 干什么 |
| --- | --- | --- |
| 0 | `/ask-matt` | 不知道自己该用哪个 → 问它 |
| 1 | `/grill-me` | 它反过来拷问你，把想法里的漏洞问出来（对齐阶段） |
| 2 | `/grill-with-docs` | 同上，但边问边把术语写进 `CONTEXT.md`、把架构决策写成 ADR |
| 3 | `/to-spec` | 把聊清楚的结论固化成规格文档 |
| 4 | `/to-tickets` | 拆成"一个智能体能独立做完"的小工单 |
| 5 | `/implement` 或 `/tdd` | 测试先行地把规格实现出来 |
| 6 | `/code-review` | 两路并行评审：一路查"符不符合本仓库标准"，一路查"符不符合原始规格" |

## 5.2 全部 25 个技能

本节为原教程的 25 个技能历史快照，当前目录及选装数量以实际安装结果为准，不用于验收数量。

**主线（7）**

| 技能 | 一句话 |
| --- | --- |
| `setup-matt-pocock-skills` | 配置本仓库（第 4 节，每个仓库跑一次） |
| `ask-matt` | 路由器：告诉你现在该用哪个技能 |
| `grill-with-docs` | 被访谈，同时记录术语与决策 |
| `to-spec` | 把聊出来的共识写成规格 |
| `to-tickets` | 把规格拆成智能体能做的工单 |
| `implement` | 按规格测试先行地实现 |
| `code-review` | 对照标准与规格做双向评审 |

**探路 / 产出决策（3）**：`wayfinder`（把大工程画成"待决策地图"逐个定）、`prototype`（用一次性代码回答设计问题，然后删掉）、`research`（只读一手资料，给带引用的答案）

**诊断 / 收尾（5）**：`diagnosing-bugs`（从能复现的失败用例出发定位 bug）、`resolving-merge-conflicts`（逐个 hunk 解冲突）、`triage`（把原始 issue 分类成能直接开工的活）、`wizard`（生成一个脚本，引导人手工走完配置流程）、`improve-codebase-architecture`（找出最值得重构的模块，出一份可视化报告）

**生产力（6）**：`grill-me`（定主意之前先跟自己对一遍）、`handoff`（把长会话写成交接文档给下一个智能体）、`to-questionnaire`（把开放问题变成别人填的问卷）、`teach`（跨多次会话、层层递进地学一个主题）、`wait-what`（让啰嗦的模型用大白话再说一遍）、`writing-for-agents`（怎么写技能和给智能体看的文档）

**底层参考（4，一般被别的技能引用）**：`codebase-design`（深模块设计词汇）、`domain-modeling`（打磨并记录项目术语）、`grilling`（访谈方法论本体）、`tdd`（红-绿-重构的规则）

## 5.3 在哪个工具里怎么敲

各家的"显式调用"写法不一样，这是最容易卡住的地方：

| 工具 | 怎么触发技能 |
| --- | --- |
| **Claude Code** | 直接输入 `/tdd`、`/code-review`（文档里的写法就是它） |
| **pi** | `/skill:tdd`（pi 的技能命令带 `skill:` 前缀） |
| **Codex** | 输入 `/skills` 从列表里挑，或在提示词里写 `$tdd`；**注意 Codex 不认 `/tdd` 这种写法** |
| **OpenCode** | 按 `description` 自动命中，直接说"用 tdd 技能先写测试"即可 |
| **所有工具** | 兜底一招：直接说"按 `code-review` 技能做一遍"，模型多数会自己去读 `SKILL.md` |

> 想不起来该用哪个：敲 `/ask-matt`（Claude Code）、`/skill:ask-matt`（pi）、`$ask-matt`（Codex），描述你现在的处境，它会指路。

---

# 6 验证（第一次必须做）

按安装时选择的范围执行其中一组命令。

**项目级：先进入安装技能的目标仓库。**

```PowerShell
Get-Location
npx skills list
Get-ChildItem '.agents/skills' | Select-Object Name
```

**全局：检查用户目录。**

```PowerShell
npx skills list -g
Get-ChildItem "$env:USERPROFILE\.agents\skills" | Select-Object Name
```

再按实际选装的客户端检查目录；无需检查未选择的工具。把下表中的路径代入 `Get-ChildItem '<路径>' | Select-Object Name, LinkType, Target`，其中 `%USERPROFILE%` 在 PowerShell 命令中换成 `$env:USERPROFILE` 并使用双引号。

| 客户端 | 项目级（相对仓库根目录） | 全局 |
| --- | --- | --- |
| Codex | `.agents/skills` | `%USERPROFILE%\.agents\skills`；安装器若另建 `.codex\skills` 链接，也检查其目标 |
| pi | `.pi/skills`，也可读取 `.agents/skills` | `%USERPROFILE%\.pi\agent\skills` |
| Claude Code | `.claude/skills` | `%USERPROFILE%\.claude\skills` |
| OpenCode | `.agents/skills`，安装器若另建 `.opencode/skills` 也检查其目标 | `%USERPROFILE%\.config\opencode\skills` |

使用链接安装时核对 `Target` 指向相应范围的正本；使用 `--copy` 时没有链接属性是正常的。验收以选装技能存在、内容可读取并能在客户端调用为准，不要求总数等于 25，也不要删除其他来源的技能。

然后在目标项目内新开客户端会话确认。下面以 `ask-matt` 为例；未选装它时换成自己已安装的技能：

| 工具 | 验证动作 |
| --- | --- |
| Claude Code | 输入 `/ask-matt`，应该能拉起技能 |
| pi | 输入 `/skill:ask-matt`；刚改过技能文件时先 `/reload` |
| Codex | 输入 `/skills`，列表里应能看到这些技能 |
| OpenCode | 输入“使用 ask-matt 技能”，确认读取了对应 `SKILL.md` |

> 历史本机快照（2026-09-25）：25 个技能装在 `%USERPROFILE%\.agents\skills`，Claude Code 与 pi 的全局技能目录下有指向它的 Junction。此记录不作为你的安装路径、范围或数量的验收标准。

---

# 7 日常维护

```PowerShell
npx skills list -g                       # 看装了哪些（别名 npx skills ls）
npx skills check                         # 检查有没有新版本
npx skills update -g                     # 升级全局技能
npx skills find code-review              # 在 skills.sh 上搜别的技能
# 先用上面的 list 核对技能名、来源及安装范围；只移除确认不再需要的项
npx skills remove -g tdd code-review     # 示例：仅这两个名字，保留交互确认
npx skills use mattpocock/skills@tdd     # 不安装，直接用这一个（会打印提示词）
```

卸载按技能名匹配，不按仓库来源自动隔离。同名技能若已换来源，要先确认；全局移除也可能影响多个客户端链接。保留交互确认，项目级安装应在项目内处理并省略 `-g`。命令范围见 [安装器官方说明](https://github.com/vercel-labs/skills)。

新版本换掉了模板，`docs/agents/` 里的配置可能过时——某个下游技能的行为和文档对不上时，**重跑一次 `/setup-matt-pocock-skills`** 是便宜且有效的修法。

---

# 8 常见问题

| 现象 | 可能原因、检查顺序与下一步 |
| --- | --- |
| `npx` 找不到，或下载特别慢 | Node.js 没装好，或 npm 源太慢。见 [前置工具](01-prerequisites.md)（含镜像设置） |
| 装完了，工具里看不到技能 | ① 没新开会话；② `-a` 里漏了你在用的工具；③ 工具版本太老不支持 skills；④ pi 里改过文件要 `/reload` |
| 每个技能出现两份 | 插件路线（`claude plugins install`）和 skills.sh 路线都装了。留一种，另一种删掉 |
| Windows 报软链接权限错误 | 加 `--copy` 重装，改成复制文件 |
| Codex 里 `/tdd` 报"Unrecognized command" | Codex 不支持 `/技能名` 写法：用 `$tdd` 或 `/skills` 挑选 |
| 跑完 `/setup-matt-pocock-skills`，`/to-tickets` 还是乱猜 issue 位置 | ① 没**在这个仓库**里跑；② 配置被写进了 `CLAUDE.md` 而你在用 Codex → 手改到 `AGENTS.md` |
| 提示找不到标签 / 建 issue 失败 | 先检查标签是否存在，再用 `gh label create` 创建缺失项；使用标准名也要检查；`/wayfinder` 的标签也要先手工建 |
| 报 GitHub API 限流（403） | 设 `GITHUB_TOKEN`（有 `gh` 登录的话它会自动兜底） |
| 不想要安装器的匿名统计 | 当前会话：`$env:DO_NOT_TRACK = "1"`；想永久就写进用户环境变量 |
| 想重装 Matt Skills | 先 `npx skills list -g`，核对来源后按技能名逐项移除并重装；不要删除共享 `.agents/skills` 目录或锁文件，那里可能还有其他来源 |

---

# 9 版本与来源

历史技能集版本和记录日期统一见 [README 第 4 节](README.md#4-版本基线与核验状态)。`npx skills@latest` 的 `latest` 是动态标签，不是固定验证版本；`npx skills` 的实际版本可能受本地安装和缓存影响。数量和技能名称以当前安装清单及作者仓库为准。

更新后先检查技能描述、命令和仓库约定是否仍匹配，再进行实际任务；不要因为本文列出 25 个就删除额外的其他来源技能。

---

# 10 参考资料

- 官方页面（含 7 课免费邮件课程）：<https://www.aihero.dev/skills>
- 技能目录与逐条说明：<https://www.aihero.dev/skills-catalog>
- `/setup-matt-pocock-skills` 详解：<https://www.aihero.dev/skills-setup-matt-pocock-skills>
- 仓库源码（MIT）：<https://github.com/mattpocock/skills>
- 技能市场（可搜别的技能）：<https://skills.sh>
- 安装器本体（支持 40+ 工具的清单）：<https://github.com/vercel-labs/skills>
- 技能格式规范：<https://agentskills.io>
- 中文翻译版（想直接看中文说明可先用它，命令与目录名不变）：<https://github.com/vinvcn/mattpocock-skills-zh-CN>
- Codex 侧的技能用法：<https://developers.openai.com/codex/skills>
- pi 侧的技能说明：见 [pi](04-pi.md) 第 7.3 节
