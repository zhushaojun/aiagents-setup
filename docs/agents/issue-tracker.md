# Issue tracker: GitHub

本仓库的 issue 与 spec 存放在 GitHub Issues。所有操作使用 `gh` CLI。

## 约定

- **创建 issue**：`gh issue create --title "..." --body "..."`。多行正文用 heredoc。
- **读取 issue**：`gh issue view <number> --comments`，用 `jq` 过滤评论，并同时取出 labels。
- **列出 issue**：`gh issue list --state open --json number,title,body,labels,comments --jq '[.[] | {number, title, body, labels: [.labels[].name], comments: [.comments[].body]}]'`，配合 `--label` / `--state` 过滤。
- **评论**：`gh issue comment <number> --body "..."`
- **加/去标签**：`gh issue edit <number> --add-label "..."` / `--remove-label "..."`
- **关闭**：`gh issue close <number> --comment "..."`

仓库从 `git remote -v` 推断；在 clone 内运行时 `gh` 会自动识别。

## Pull requests as a triage surface

**PRs as a request surface: no.** _(若本仓库把外部 PR 视为需求请求，改成 `yes`；`/triage` 会读这个标记。)_

当设为 `yes` 时，PR 走与 issue 相同的标签和状态，使用对应的 `gh pr` 命令：

- **读取 PR**：`gh pr view <number> --comments`，diff 用 `gh pr diff <number>`。
- **列出待 triage 的外部 PR**：`gh pr list --state open --json number,title,body,labels,author,authorAssociation,comments`，仅保留 `authorAssociation` 为 `CONTRIBUTOR`、`FIRST_TIME_CONTRIBUTOR`、`NONE` 的项（丢弃 `OWNER`/`MEMBER`/`COLLABORATOR`）。
- **评论 / 打标签 / 关闭**：`gh pr comment`、`gh pr edit --add-label`/`--remove-label`、`gh pr close`。

GitHub 的 issue 与 PR 共用同一编号空间，因此裸写 `#42` 可能是任意一种：先用 `gh pr view 42` 解析，失败再退回 `gh issue view 42`。

## 当技能说「publish to the issue tracker」

创建一个 GitHub issue。

## 当技能说「fetch the relevant ticket」

执行 `gh issue view <number> --comments`。

## Wayfinding 操作

由 `/wayfinder` 使用。**map** 是一个 issue，**child** issue 作为 ticket。

- **Map**：一个带 `wayfinder:map` 标签的 issue，承载 Notes / Decisions-so-far / Fog 正文。`gh issue create --label wayfinder:map`。
- **Child ticket**：作为 map 的 GitHub sub-issue 关联（对 sub-issues endpoint 调用 `gh api`）。若未启用 sub-issues，则把 child 加入 map 正文的 task list，并在 child 正文顶部写 `Part of #<map>`。标签：`wayfinder:<type>`（`research`/`prototype`/`grilling`/`task`）。被认领后指派给推进的开发者。
- **Blocking**：使用 GitHub **原生 issue dependencies**，这是规范且 UI 可见的表达。用 `gh api --method POST repos/<owner>/<repo>/issues/<child>/dependencies/blocked_by -F issue_id=<blocker-db-id>` 添加边，其中 `<blocker-db-id>` 是阻塞者的数字 **database id**（`gh api repos/<owner>/<repo>/issues/<n> --jq .id`，**不是** `#number` 也不是 `node_id`）。GitHub 通过 `issue_dependencies_summary.blocked_by` 报告（仅未关闭的阻塞者，即实际闸门）。若 dependencies 不可用，退回在 child 正文顶部写 `Blocked by: #<n>, #<n>`。所有阻塞者关闭后 ticket 即解锁。
- **Frontier 查询**：列出 map 的未关闭 children（`gh issue list --state open`，限定在 map 的 sub-issues / task list），剔除有未关闭阻塞者（`issue_dependencies_summary.blocked_by > 0`，或 `Blocked by` 行中仍有未关闭 issue）或有 assignee 的项；按 map 顺序取第一个。
- **认领**：`gh issue edit <n> --add-assignee @me`，这是本 session 的第一次写入。
- **解决**：`gh issue comment <n> --body "<answer>"`，然后 `gh issue close <n>`，再把上下文指针（gist + link）追加到 map 的 Decisions-so-far。