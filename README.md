# AI Coding 智能体安装配置手册

面向**使用指定 NewAPI 服务的 Windows 内部学员**。主线采用 PowerShell 7；服务地址为 `https://newapi.ttxs.site ， 账号联系王义江`。Linux 步骤作为补充，其他供应商需自行核对协议与凭据。

目标是先跑通一个智能体，再按需求增加工具。本文配置示例不含真实密钥；已有配置要先备份并合并，不能整份覆盖。分享自己的配置前仍需检查其他凭据、本机路径和插件设置。

# 1 推荐路径

**准备环境 → 配置凭据 → 跑通 Codex → 验证读文件和执行命令 → 按需增加其他工具。**

| 顺序 | 工具 | 在这套教程中的用途 |
| --- | --- | --- |
| 主力 | [Codex](03-codex.md) | 日常读代码、修改和命令执行，CLI / IDE / 桌面入口 |
| 次主力 | [pi](04-pi.md) | 尝试不同模型与终端扩展 |
| 辅助 | [OpenCode](05-opencode.md) | 灵活接入供应商，按需使用 Console 模型 |
| 辅助 | [Claude Code](06-claude-code.md) | 使用其工具流程与生态，单独验证第三方模型兼容性 |
| 进阶 | [Orca](07-orca.md) | 已有 CLI 跑顺后，用 worktree 管理并行任务 |

这是我们的使用偏好，不是模型或客户端的普遍能力排名。比较方法见 [总览](00-overview.md)。技能包与供应商管理按需添加，不要求一次装齐。

# 2 最短成功步骤

1. 按 [前置工具](01-prerequisites.md) 安装 Node.js ≥22.19.0、Git for Windows 和 PowerShell 7，运行第 5 节自检；VS Code 可选。
2. 按 [统一接入](02-unified-access.md) 第 2 节设置用户级环境变量，确认当前进程能读取到非空值。必要时完整重启终端或桌面应用。
3. 按 [Codex](03-codex.md) 第 3、4 节安装 CLI 并配置 NewAPI。已有配置先备份再合并。
4. 在 PowerShell 7 中创建独立练习仓库并验证请求：

```PowerShell
$practiceDir = Join-Path $env:USERPROFILE ('agent-practice-' + [guid]::NewGuid().ToString('N').Substring(0,8))
New-Item -ItemType Directory -Path $practiceDir -ErrorAction Stop | Out-Null
Set-Location -LiteralPath $practiceDir
git init
if ($LASTEXITCODE -ne 0) { throw 'Git 初始化失败，请先检查 Git 安装。' }
$probeText = 'check-' + [guid]::NewGuid().ToString('N')
Set-Content -LiteralPath 'agent-check.txt' -Value $probeText -Encoding utf8
Write-Output "核对文本：$probeText"
codex exec -c 'model_provider="newapi"' --model gpt-6-sol "只回复两个字：可用"
```

无需先提交文件。`codex exec` 默认要求 Git 仓库，因此不要直接在用户目录执行。收到“可用”只说明基本请求成功；继续按 [Codex 第 5 节](03-codex.md#5-验证第一次使用必须做)验证读取文件和只读命令。实际请求会使用中转额度。

# 3 文档地图

正文在仓库根目录，截图在 `images/`。编号表示推荐阅读顺序；`docs/agents/` 为仓库维护规范，读者可跳过。

| 文件 | 内容 | 什么时候看 |
| --- | --- | --- |
| [00-overview.md](00-overview.md) | 选型依据、客户端与模型的区别 | 可选背景 |
| [01-prerequisites.md](01-prerequisites.md) | 安装前置工具、自检、练习目录 | 首次安装 |
| [02-unified-access.md](02-unified-access.md) | 凭据、地址、默认模型与快照、诊断、备份合并 | 配置任何工具前 |
| [03-codex.md](03-codex.md) | Codex 安装、配置、三层验证与权限 | 主线必看 |
| [04-pi.md](04-pi.md) | pi 的模型配置、Shell 与扩展 | 按需添加 |
| [05-opencode.md](05-opencode.md) | OpenCode v2 配置、Console 与验证 | 按需添加 |
| [06-claude-code.md](06-claude-code.md) | Claude Code 配置、权限与兼容性 | 按需添加 |
| [07-orca.md](07-orca.md) | worktree 并行、依赖处理、Orca CLI | 至少一个 CLI 跑顺后 |
| [08-cc-switch.md](08-cc-switch.md) | 多供应商管理与本地路由 | 确有切换需求时 |
| [09-matt-skills.md](09-matt-skills.md) | 技能包安装、使用与选择性卸载 | 想固化工作流程时 |
| [10-appendix-full-config.md](10-appendix-full-config.md) | 本机进阶配置片段及适用限制 | 基础用法跑顺后 |

# 4 版本基线与核验状态

以下是原教程的**本机记录基线**，不是最新版承诺。2026-09-26 修订核对了文档和示例语法，未重新调用真实模型；正文版本输出与截图可能来自不同安装时点。

| 工具 | 原记录版本 | 记录日期 | 查看本机版本 |
| --- | --- | --- | --- |
| Codex | 0.156.x；正文另有 0.157.0 输出示例，后者未在本次重新验证 | 2026-09-25 | `codex --version` |
| pi | 0.87.x | 2026-09-25 | `pi --version` |
| Claude Code | 2.1.x | 2026-09-25 | `claude --version` |
| OpenCode | `@opencode/cli` 2.0.x | 2026-09-25 | `opencode --version` |
| Orca | 1.4.x | 2026-09-25 | `orca status --json` 的 `appVersion` |
| Matt Skills | v1.2、25 个技能（历史快照） | 2026-09-25 | `npx skills list -g` 查看已安装项，`npx skills check` 检查更新 |

`@latest` 是动态安装标签，不是验证版本。升级后应记录实际版本，并重做各工具的三层验证。模型默认值、清单快照和实时查询入口统一见 [统一接入第 4 节](02-unified-access.md#4-当前可用的模型清单)。

# 5 常见问题入口

| 现象 | 从哪里查 |
| --- | --- |
| 缺变量、401、重连、超时 | [统一接入第 6 节](02-unified-access.md#6-常见报错对照) |
| 能回答但不能执行命令 | [前置工具](01-prerequisites.md) 的 Git Bash 检查及各工具权限说明 |
| Codex 提示不在 Git 仓库 | 按本页第 2 节创建练习仓库 |
| 模型名不存在或无可用渠道 | [统一接入第 4、6 节](02-unified-access.md#4-当前可用的模型清单) |
| OpenCode 配置字段不识别 | [OpenCode 第 3、4 节](05-opencode.md#3-安装)，确认安装的是 v2 |
| Claude Code 图片或长会话异常 | [Claude Code 第 8 节](06-claude-code.md#8-选型与兼容性注意事项) |
| Orca 启动失败或依赖缺失 | [Orca 第 8 节](07-orca.md#8-常见问题) |

# 6 后续验证与维护

- 配置与密钥引用方式见 [统一接入](02-unified-access.md)，详细故障原因不在首页重复维护。
- 真实中转请求、工具调用、图片输入、长上下文和输出上限仍需在对应版本与模型组合下验证；短文本成功不能替代这些测试。
- 截图用于辨认入口；额外模型、主题与版本以图片旁的说明为准。官方来源列在各篇末尾。
