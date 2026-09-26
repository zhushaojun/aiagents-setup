# 附：完整配置（我们机器上的进阶配置）

**这份文档是"抄作业参考"，不是必读内容。** 主文档给的是**精简版**；这里收录原教程的本机进阶配置快照，供你按需增补，不代表当前机器状态或推荐默认值。版本基线统一见 [README 第 4 节](README.md#4-版本基线与核验状态)。

**所有代码块均为配置片段或历史样例**，应按 [统一接入第 9 节](02-unified-access.md#9-已有配置的备份与合并)备份后合并；不要整份替换现有文件。缺少外层 JSON 对象或含省略号的块不能单独作为完整 JSON。

## ⚠️ 抄之前先看这三条

1. **绝对路径不能抄**：文中的 `C:\Users\zsj\...`、插件市场路径、hook 校验哈希都是本机专属，换台机器必须自己重新生成。
2. **有些配置会明显削弱或放开能力**，抄错了会很难排查（例如 `deny: ["Bash"]` 只会禁用 Bash 工具，不能覆盖 PowerShell 或其他执行工具）。
3. **建议增量抄**：每次只加一项，加完立刻验证（`codex exec "只回复两个字：可用"` 这类命令），坏了能马上定位。

---

# 1 Claude Code 完整配置要点

完整文件：`%USERPROFILE%\.claude\settings.json`（**不含密钥**：`env` 里不放 `ANTHROPIC_AUTH_TOKEN`，它来自用户级环境变量——本文仅展示无真实密钥的片段，自己的完整文件分享前仍需检查）

## 1.1 顶层开关（除 `env` 外）

| 配置项 | 我们的值 | 作用 | 建议 |
| --- | --- | --- | --- |
| `language` | `简体中文` | 界面与回复语言 | 抄 |
| `model` | `haiku` | 默认档位；`haiku` 这一档在我们映射里是 `gpt-6-luna`。注意截图里 `/model` 选中的是 `Custom model`（值就是 `ANTHROPIC_MODEL`），实际生效的是它 | **别照抄我们这个值**：[教程](06-claude-code.md)给的 `sonnet` 更适合当默认（映射到主力模型 `deepseek-v4.1-flash[1M]`）。`haiku` 是我们的历史选择，而且会被 `/model` 里的选择覆盖 |
| `alwaysThinkingEnabled` | `true` | 总是思考 | 抄 |
| `effortLevel` | `high` | 出力档位 | 抄 |
| `autoUpdatesChannel` | `stable` | 跟稳定版 | 抄 |
| `defaultShell` | `powershell` | 默认 shell | 抄（Windows） |
| `theme` / `tui` | `auto` / `fullscreen` | 主题、全屏 TUI | 随意 |
| `autoMemoryEnabled` | `false` | 关掉自动记忆 | 随意 |
| `disableBundledSkills` | `true` | 关掉内置技能 | **影响能力，先不要抄** |
| `disableArtifact` / `disableClaudeAiConnectors` / `disableRemoteControl` / `disableWorkflows` | `true` | 关掉若干联网/远端功能（我们只走中转，用不到） | 随意 |
| `attribution` | `{"commit": "", "pr": ""}` | 提交记录里不加 Claude 署名 | 看团队规范 |

## 1.2 `env` 块（可抄，但删掉密钥行）

JSON 片段（合并到对应对象；不能单独保存为完整 JSON）：

```text
"env": {
  "ANTHROPIC_BASE_URL": "https://newapi.ttxs.site",
  "ANTHROPIC_MODEL": "deepseek-v4.1-flash[1M]",
  "ANTHROPIC_DEFAULT_FABLE_MODEL": "glm-5.3-flash[1M]",
  "ANTHROPIC_DEFAULT_FABLE_MODEL_NAME": "glm-5.3-flash",
  "ANTHROPIC_DEFAULT_OPUS_MODEL": "claude-opus-5-5[1M]",
  "ANTHROPIC_DEFAULT_OPUS_MODEL_NAME": "claude-opus-5-5",
  "ANTHROPIC_DEFAULT_SONNET_MODEL": "deepseek-v4.1-flash[1M]",
  "ANTHROPIC_DEFAULT_SONNET_MODEL_NAME": "Deepseek V4.1 Flash",
  "ANTHROPIC_DEFAULT_HAIKU_MODEL": "gpt-6-luna",
  "ANTHROPIC_DEFAULT_HAIKU_MODEL_NAME": "gpt-6-luna",
  "CLAUDE_CODE_SUBAGENT_MODEL": "deepseek-v4.1-flash[1M]",
  "CLAUDE_CODE_USE_POWERSHELL_TOOL": "1",
  "CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC": "1"
}
```

> `ANTHROPIC_AUTH_TOKEN` 这一行我们**没有**放进来——它来自用户级环境变量，这里只能说明该示例没有写入这个密钥。

## 1.3 权限（`permissions`）

我们机器上的实际配置：

JSON 片段（合并到对应对象；不能单独保存为完整 JSON）：

```text
"permissions": {
  "allow": ["WebFetch", "WebSearch"],
  "deny": ["EnterPlanMode", "ExitPlanMode", "DesignSync", "NotebookEdit", "SendMessage",
           "PushNotification", "RemoteTrigger", "ReportFindings", "ScheduleWakeup",
           "AskUserQuestion", "CronCreate", "CronDelete", "CronList", "Bash"],
  "defaultMode": "auto"
}
```

**代价说清楚**：这份历史配置只禁用 Bash，没有禁用独立的 PowerShell 工具，不能宣称它只能读改文件。若主要用作编辑器，按 [Claude Code 第 7.1 节](06-claude-code.md#71-权限模式)同时处理两种命令工具，并检查插件/MCP 的执行能力。`AskUserQuestion` 禁用的是结构化提问工具，模型仍可能在普通回复中提出澄清问题。

## 1.4 hooks（进阶，慎抄）

我们挂了一批 hooks，主要做两件事：

1. **拦命令**：`PreToolUse` 里用 `jq` + `grep` 扫描模型要执行的命令，命中 `npm|yarn|npx` 就拒绝并提示改用 `pnpm`，命中 `pip|poetry|conda` 就拒绝并提示改用 `uv`；
2. **联动外部工具**：`SubagentStart`/`SubagentStop`/`PostToolUse`/`PostToolUseFailure`/`PermissionRequest`/`PostCompact`/`SessionEnd` 等事件上，调用一个外部脚本（`~\.orca\agent-hooks\claude-hook.cmd`，用 PowerShell 编码命令包起来）。

⚠️ 注意两点：

- 这些 hook 依赖 `jq` / `grep`；Git Bash 常带 `grep`，`jq` 需另行检查安装。缺依赖的表现由 hook 脚本和客户端决定，应查看日志；
- 第二类是**本机专属**的外部脚本，换机器就要重新写。

**建议**：只抄第 1 类（拦命令）里的思路，自己按需写；不要整段复制。

## 1.5 插件与状态栏

JSON 片段（合并到对应对象；不能单独保存为完整 JSON）：

```text
"enabledPlugins": {
  "context-mode@context-mode": true,
  "pyright-lsp@claude-plugins-official": true
},
"extraKnownMarketplaces": {
  "claude-hud": { "source": { "source": "github", "repo": "jarrodwatts/claude-hud" } },
  "context-mode": { "source": { "source": "github", "repo": "mksglu/context-mode" } }
}
```

`statusLine` 指向 claude-hud 插件（显示上下文/用量），命令里带有本机 Node 路径，**不能直接抄**——照它的说明重新安装插件即可。

## 1.6 其它文件

| 文件 | 内容 |
| --- | --- |
| `%USERPROFILE%\.claude\config.json` | 历史兼容配置，当前适用性待验证，不代替有效凭据，见 Claude Code 第 4.2 节 |
| `%USERPROFILE%\.claude\CLAUDE.md` | 全局提示词（我们这里目前是空的，建议按 [Claude Code](06-claude-code.md) 4.3 写几条） |
| `%USERPROFILE%\.claude\settings.local.json` | 本机私有覆盖（我们只放了 `skillOverrides`） |
| `%USERPROFILE%\.claude\skills\`、`agents\`、`plugins\` | 技能、自定义 agent、插件缓存 |

---

# 2 Codex 完整配置要点

完整文件：`%USERPROFILE%\.codex\config.toml`

## 2.1 我们的运行段

```toml
model = "gpt-6-sol"
model_provider = "newapi"
model_reasoning_effort = "medium"
personality = "pragmatic"
approval_policy = "on-request"
sandbox_mode = "danger-full-access"      # ⚠️ 比教程默认更放开
approvals_reviewer = "auto_review"
web_search = "live"

[model_providers.newapi]
name = "OpenAI"
base_url = "https://newapi.ttxs.site/v1"
wire_api = "responses"
requires_openai_auth = false
env_key = "NEWAPI_KEY"
```

> **为什么是 `env_key`**：只写变量名，密钥留在用户级环境变量里（设置方法见 [统一接入](02-unified-access.md) 第 2 节），配置文件不含密钥。`experimental_bearer_token` 会把密钥字面写进文件，我们不用。
> **代价要清楚**：`env_key` 读的是 **Codex 自己进程**的环境变量。桌面 App、VS Code / Zed 扩展这类长驻入口继承的是它们启动那一刻的环境——**设完或换完 Key 要重启它们一次**，未读取到变量会报缺变量；`Reconnecting` 还可能来自网络、中转和服务端异常，不能仅凭它判定令牌错误。见统一接入第 6 节。
> **两者绝不要同时写**：原教程记录同时存在时优先采用 `env_key`，但未保留精确版本测试日志，“多加一行 env_key 更保险”是陷阱。
> `auth.json` / `OPENAI_API_KEY`（`codex login` 的账号层凭据）不应被当作本教程显式配置的 `env_key` 的替代来源；先确认实际 provider 和凭据引用。
> `openai_base_url = "http://127.0.0.1:57321/v1"` 是**桌面版（现在的 ChatGPT 桌面应用）/computer-use 运行时自动写入的**，不要手动抄。

## 2.2 功能开关与界面

```toml
[features]
hooks = true
memories = false
prevent_idle_sleep = true
chronicle = false
[features.context_management]
experimental_mode = true

[tui]
notifications = ["approval-requested", "agent-turn-complete"]
status_line = ["model-with-reasoning", "current-dir", "git-branch", "context-used",
               "run-state", "permissions", "approval-mode"]
status_line_use_colors = true

[windows]
sandbox = "elevated"        # Windows 沙盒实现；不等于关闭沙盒

[sandbox_workspace_write]
network_access = true
```

本节 Windows 实现选项与沙盒访问级别是两回事：上节 `danger-full-access` 才关闭沙盒限制；`approval_policy` 控制审批，`approvals_reviewer` 控制谁审核。默认仍推荐正文的 `workspace-write` 与 `on-request`。

## 2.3 桌面应用段（ChatGPT 桌面应用）

```toml
[desktop]
appearanceTheme = "dark"
codeFontSize = 13
localeOverride = "zh-CN"
integratedTerminalShell = "powershell"
show-context-window-usage = true
enabled-reasoning-efforts = ["low", "medium", "high", "xhigh", "max"]
```

这些是桌面应用的界面偏好，**可以由应用自己维护**，不建议手抄。

## 2.4 插件 / 技能 / MCP

```toml
[[skills.config]]
path = 'C:\Users\zsj\.codex\skills\screenshot\SKILL.md'
enabled = true

[plugins."documents@openai-primary-runtime"]
enabled = true

[mcp_servers.context7]
command = "npx"
args = ["-y", "@upstash/context7-mcp"]
```

`[marketplaces.*]` 里是**本机缓存路径**（`~\.codex\.tmp\...`、`~\.cache\codex-runtimes\...`），换机器无效；`[hooks.state.*]` 里的 `trusted_hash` 是 Codex 自己对 hooks 文件算的校验值，**不要手写**。

---

# 3 pi 完整配置要点

## 3.1 `settings.json`

```json
{
  "theme": "dark",
  "defaultProvider": "bailian",
  "defaultModel": "deepseek-v4.1-flash",
  "defaultThinkingLevel": "high",
  "defaultProjectTrust": "always",
  "editorPaddingX": 0,
  "showHardwareCursor": true,
  "externalEditor": "code --wait",
  "shellPath": "C:/Program Files/PowerShell/7/pwsh.exe",
  "packages": ["npm:pi-web-access", "npm:pi-subagents", "npm:pi-mcp-adapter", "npm:pi-background-tasks", "npm:context-mode"],
  "enabledModels": [
    "newapi/deepseek-v4.1-flash", "newapi/glm-5.3-flash", "newapi/muse-spark-1.3-contributor",
    "newapi/gpt-6-luna", "newapi/gpt-6-sol", "newapi/gpt-6-astra",
    "bailian/qwen3.8-max", "bailian/deepseek-v4.1-flash"
  ]
}
```

| 项 | 说明 | 建议 |
| --- | --- | --- |
| `defaultProjectTrust: "always"` | 不再询问是否信任项目 | ⚠️ 全局跳过项目信任确认；在自己机器上也应逐个检查仓库来源，不作为推荐默认值 |
| `enabledModels` | 限定 `/model` 与 `Ctrl+P` 循环里的模型 | 抄（按自己的清单改） |
| `packages` | 已装的插件包 | 按需一个个加 |
| `shellPath` | 0.87.x 历史本机配置指向 PowerShell 7 | 需路径真实存在且该版本兼容；本次未复测。新手按正文删去该项走默认 Git Bash，PowerShell 工具按正文 7.1 单独启用 |
| `externalEditor` | `/editor` 用什么打开 | 随意 |

> 我们的 `defaultProvider` 是 `bailian`（另一家供应商、**另一把密钥**），教程里不涉及；你按 [pi](04-pi.md) 配成 `newapi` 即可。

## 3.2 pi 的 `models.json`

我们文件里有三个供应商：`newapi`（教程用这个）、`deepseek`、`bailian`。后两个走各自官方的 Key，与本教程无关。[pi 第 4.1 节](04-pi.md#41-接入-newapiuserprofilepiagentmodelsjson)的首次配置已包含 `thinkingLevelMap`，用于声明可选思考档位及参数映射。新增模型时应核对这些映射；`compat` 兼容开关（如 `thinkingFormat`、`maxTokensField`）则按实际协议需要调整。官方文档：<https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/models.md>

`newapi` 块仅引用环境变量，不在这个配置块中保存真实密钥（和教程一致）：

JSON 片段（合并到对应对象；不能单独保存为完整 JSON）：

```text
"newapi": {
  "baseUrl": "https://newapi.ttxs.site/v1",
  "apiKey": "$NEWAPI_KEY",
  "api": "openai-responses",
  "models": [ … ]
}
```

pi 会优先使用运行时凭据及 `auth.json` 中已存的凭据，再考虑 `models.json` 的 `apiKey` 等来源；具体规则见 [pi 模型配置文档](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/models.md)。旧条目可能覆盖环境变量方案。

仅检查 `newapi` 条目是否存在，不打印密钥或整个文件：

```PowerShell
$authPath = Join-Path $env:USERPROFILE '.pi/agent/auth.json'
if (Test-Path -LiteralPath $authPath -PathType Leaf) {
  try {
    $authData = Get-Content -LiteralPath $authPath -Raw -ErrorAction Stop | ConvertFrom-Json -AsHashtable -ErrorAction Stop
    Write-Output ('存在 newapi 条目：' + $authData.ContainsKey('newapi'))
  } catch {
    Write-Output '凭据文件无法解析；请在本机编辑器检查，不要贴出完整文件。'
  }
} else {
  Write-Output '没有 auth.json 文件'
}
```

若确认旧条目需要移除，先按统一接入第 9 节备份这个文件，再在本机编辑器中仅删除 `newapi` 条目，保留其他供应商。备份也含凭据，不上传。不要用低深度 JSON 重序列化覆盖整个凭据文件。

`pi --list-models` 可检查本地模型注册和凭据解析，但不能验证密钥被服务端接受，也不应要求固定模型数量。错误诊断统一见 [统一接入第 6 节](02-unified-access.md#6-常见报错对照)。

---

# 4 OpenCode 完整配置要点

## 4.1 `~/.config/opencode/` 目录内容

| 文件/目录 | 内容 | 建议 |
| --- | --- | --- |
| `opencode.json` | 主配置（v2 格式：`providers` / `package` / `settings`，顶层 `model` 是默认模型） | 抄教程精简版 |
| ↳ 顶层 `model` 键 | 我们设的是 `newapi/deepseek-v4.1-flash` | 建议显式指定默认模型；历史本机曾回退到 `space-bunny-free`，并非固定结果，实际选择以当前界面为准（见 4.2） |
| `cli.json` | v2 的界面偏好（`session.thinking`、`diffs.wrap`、滚动条等） | 随意 |
| `plugins/` | 本地插件（我们装了 `orca-opencode-status`） | 按需 |
| `node_modules/`、`package-lock.json` | 插件依赖，OpenCode 自己维护 | 不要手动改 |
| `oh-my-openagent.json` | 多智能体编排插件配置 | ⚠️ **我们已弃用**：里面引用的模型名（`kimi-k2.7-code`、`gpt-5.5`、`glm-5.1`、`qwen3.7-plus`、`minimax-m3`、`gpt-5.4-mini`）在原教程中被记录为不可用；本次未重测，需查询当前凭据的实时列表。要用的话必须先把模型名换成 [统一接入](02-unified-access.md) 第 4 节清单里的 |
| `service.json` | 后台服务凭据，自动生成 | 不要改 |

## 4.2 确认实际模型再比较结果

默认模型、提示词、上下文管理与工具权限都会影响结果。pi 和 OpenCode 的正文默认使用相同模型，不能根据“是否生产自家模型”推导客户端能力排名。

先确认实际模型使用 `newapi/` 前缀，必要时用 `--model newapi/deepseek-v4.1-flash` 明确选择。然后固定任务与参数比较客户端；若要比较模型，则固定客户端。未设置默认模型时，客户端可能选择历史模型或内置可用模型，具体选择以界面为准。

## 4.3 思考档位映射：`variants`

首次配置的完整可选档位见 [OpenCode 第 4 节](05-opencode.md#4-配置首次创建已有文件先备份合并)：五个模型均已包含 `variants`，可直接用 `模型#档位` 选择。本节补充历史配置与映射说明；仅添加 `variants` 不会自动选中 `high`。

以下为历史模型条目片段；参数透传、档位、图片输入及上下文/输出上限均需按实际中转协议复测。正文仍采用文本输入声明，不因附录声明 `image` 就认为该链路支持图片。

OpenCode 的档位（命令行写作 `模型#档位`，见 [OpenCode](05-opencode.md) 第 6 节）可以来自模型目录，也可以在模型条目下用 `variants` 数组自行声明或覆盖，把档位翻译成该厂商真正认的参数。不能假定所有模型都有 `low / medium / high / max`。

JSON 片段（合并到对应对象；不能单独保存为完整 JSON）：

```text
"deepseek-v4.1-flash": {
  "name": "deepseek-v4.1-flash",
  "limit": { "context": 500000, "output": 384000 },
  "capabilities": { "tools": true, "input": ["text", "image"], "output": ["text"] },
  "variants": [
    { "id": "low",    "settings": { "thinking": { "type": "disabled" } } },
    { "id": "medium", "settings": { "thinking": { "type": "enabled" } } },
    { "id": "high",   "settings": { "reasoningEffort": "high" } },
    { "id": "max",    "settings": { "reasoningEffort": "max" } }
  ]
}
```

原教程本机快照的 `newapi` 下有 12 个模型，其中 10 个声明了 `variants`，档位集合按厂商能力分三种：

| 档位集合 | 模型 | 特点 |
| --- | --- | --- |
| `low / medium / high / max` | `deepseek-v4.1-flash`、`deepseek-v4-pro` | 低档直接关思考（`thinking.type: disabled`），`high` / `max` 走 `reasoningEffort` |
| `low / medium / high / xhigh` | `gpt-6-sol`、`gpt-6-luna`、`gpt-6-astra` | 最高档叫 `xhigh`，**不是** `max` |
| `low / medium / high` | `glm-5.3`、`glm-5.3-flash`、`qwen3.8-plus`、`mimo-v2.6-flash`、`mimo-v2.6-pro` | 没有更高档 |

> 这和 pi 的 `thinkingLevelMap` 都用于档位映射（见 3.2）：**客户端的档位是抽象的，必须映射到厂商真实参数才生效**。如果模型目录和本地配置都没有提供 `variants`，就没有命名档位可选；仍可通过模型级 `settings` 设置默认参数。
> ⚠️ 剩下两个没声明 `variants` 的模型 `kimi-k2.7-code`、`minimax-m3`，名字也在 4.1 的历史不可用名单里（[统一接入](02-unified-access.md) 第 4 节的清单里同样没有它们）。旧条目仍可能出现在模型选择器里，但本地可选不代表服务端可用；先核对实时列表再清理。表中的 `qwen3.8-plus` 也不在本教程快照里，不要照表新增。

---

# 5 Orca（ADE）配置要点

完整教程见 [Orca](07-orca.md)。这里只记我们机器上的实际形态，以及一个容易踩的坑。

## 5.1 安装与路径（本机）

| 项目 | 我们的值 |
| --- | --- |
| 安装方式 | Scoop 包 `orca-ide`（当前 `1.4.209`），升级用 `scoop update orca-ide` |
| CLI | `orca`（Scoop 的 shim 指向 `<安装目录>\resources\bin\orca.exe`）。用官方安装包装的版本，要在 `Settings → General → Orca CLI` 里注册才会进 PATH |
| 应用数据 | `%APPDATA%\orca`（`orchestration.db`、`logs\`、`profiles\`、`Preferences` 等）：应用自己维护，**不要手改** |
| 托管 hook 脚本 | `%USERPROFILE%\.orca\agent-hooks\`（我们这里有 `claude-hook.cmd`、`codex-hook.cmd`、`copilot-hook.ps1`、`gemini-hook.cmd`） |

## 5.2 1.4 节里那个 `claude-hook.cmd` 是什么

1.4 节第 2 类 hook 调用的 `~\.orca\agent-hooks\claude-hook.cmd`，**是 Orca 自己装的托管脚本**，不是我们写的业务逻辑。它靠 Orca 启动智能体时注入的环境变量工作：`ORCA_AGENT_HOOK_ENDPOINT`、`ORCA_AGENT_HOOK_PORT`、`ORCA_AGENT_HOOK_TOKEN`、`ORCA_PANE_KEY` 等；脚本开头就是"环境变量不全就直接 `exit /b 0`"。所以：

- **它只在"由 Orca 启动的会话"里有意义**（把 working / waiting / done 状态报给 Orca 界面）；
- 在**普通终端**里跑 `claude` 时它一路静默退出，不影响你；
- 它是本机专属路径，换机器大概率不存在，**不属于我们需要维护的配置**。

`Settings → Agents → Agent status hooks` 控制的就是这批托管 hook（关掉会移除、重新打开会恢复，都不需要重启应用）。

## 5.3 用中转时的两条纪律

1. **不要用 `Add account`**。Orca 的 Claude / Codex 账号热切换是给**官方订阅**用的；它管理的额外账号使用独立 home（`~/.local/share/orca/codex-accounts/<id>/home`），**读不到 `~/.codex/config.toml` 里的 NewAPI 配置**。用中转就保持 **system default**。
2. **注意权限预填**。Orca 默认给 Claude / Codex 预填权限绕过参数（`--dangerously-skip-permissions`、`--dangerously-bypass-approvals-and-sandbox`）。历史 Codex 段的 `danger-full-access` 已关闭沙盒；`elevated` 不是另一层权限绕过。要回到教程默认保护，先把 CLI 恢复为 `workspace-write` 与 `on-request`，再在 `Settings → Agents → Agent Permissions` 改成 **Manual**。

## 5.4 OpenCode 插件 `orca-opencode-status`

4.1 表里的 `orca-opencode-status`，就是这套"状态上报"在 OpenCode 上的对应物（由 Orca 自动安装的状态插件）。删掉它不影响 OpenCode 本身，只是 Orca 看不到那个会话的实时状态。

---

# 6 一张"抄配置"检查表

抄完任何一项，请逐条确认：

- [ ] 配置没有真实凭据；不要只搜索 `sk-` 前缀，还需检查其他令牌、URL 参数和插件设置
- [ ] 绝对路径（`C:\Users\zsj\...`）都换成了自己的，或删掉了
- [ ] 模型名都在 [统一接入](02-unified-access.md) 第 4 节清单里
- [ ] 权限相关配置（`deny` / `sandbox_mode`）是你**有意**要的，不是顺手抄的
- [ ] 用 Orca 时确认它启动的是 **system default** 的 Codex / Claude（没用 `Add account` 加额外账号，否则读不到上面的中转配置）
- [ ] 按相应客户端第 5 节完成三层验证；短文本成功只证明基本请求链路，不证明新加的 hook、插件或权限规则生效
