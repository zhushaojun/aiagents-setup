# 附：完整配置（我们机器上的进阶配置）

**这份文档是"抄作业参考"，不是必读内容。** 主文档给的是**精简版**；这里收录我们机器上实际在用的进阶配置，供你按需增补。

## ⚠️ 抄之前先看这三条

1. **绝对路径不能抄**：文中的 `C:\Users\zsj\...`、插件市场路径、hook 校验哈希都是本机专属，换台机器必须自己重新生成。
2. **有些配置会明显削弱或放开能力**，抄错了会很难排查（例如 `deny: ["Bash"]` 会让模型不能执行命令）。
3. **建议增量抄**：每次只加一项，加完立刻验证（`codex exec "只回复两个字：可用"` 这类命令），坏了能马上定位。

---

# 1 Claude Code 完整配置要点

完整文件：`%USERPROFILE%\.claude\settings.json`（**不含密钥**：`env` 里不放 `ANTHROPIC_AUTH_TOKEN`，它来自系统环境变量——所以这份文件可以直接分享）

## 1.1 顶层开关（除 `env` 外）

| 配置项 | 我们的值 | 作用 | 建议 |
| --- | --- | --- | --- |
| `language` | `简体中文` | 界面与回复语言 | 抄 |
| `model` | `haiku` | 默认档位；`haiku` 这一档在我们映射里是 `gpt-6-luna`。注意截图里 `/model` 选中的是 `Custom model`（值就是 `ANTHROPIC_MODEL`），实际生效的是它 | 抄 |
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

```json
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

> `ANTHROPIC_AUTH_TOKEN` 这一行我们**没有**放进来——它来自系统环境变量，这样配置文件可以安全分享。

## 1.3 权限（`permissions`）

我们机器上的实际配置：

```json
"permissions": {
  "allow": ["WebFetch", "WebSearch"],
  "deny": ["EnterPlanMode", "ExitPlanMode", "DesignSync", "NotebookEdit", "SendMessage",
           "PushNotification", "RemoteTrigger", "ReportFindings", "ScheduleWakeup",
           "AskUserQuestion", "CronCreate", "CronDelete", "CronList", "Bash"],
  "defaultMode": "auto"
}
```

**代价说清楚**：`deny` 里的 `Bash` 意味着**模型不能执行任何命令**（只能读/改文件、搜网页），`AskUserQuestion` 也禁了（它不会反问你要澄清）。这是"把它当高级编辑器用"的配置，**不是新手配置**。

## 1.4 hooks（进阶，慎抄）

我们挂了一批 hooks，主要做两件事：

1. **拦命令**：`PreToolUse` 里用 `jq` + `grep` 扫描模型要执行的命令，命中 `npm|yarn|npx` 就拒绝并提示改用 `pnpm`，命中 `pip|poetry|conda` 就拒绝并提示改用 `uv`；
2. **联动外部工具**：`SubagentStart`/`SubagentStop`/`PostToolUse`/`PostToolUseFailure`/`PermissionRequest`/`PostCompact`/`SessionEnd` 等事件上，调用一个外部脚本（`~\.orca\agent-hooks\claude-hook.cmd`，用 PowerShell 编码命令包起来）。

⚠️ 注意两点：

- 这些 hook 依赖 `jq` / `grep`（来自 **Git for Windows**），没装就静默失败；
- 第二类是**本机专属**的外部脚本，换机器就要重新写。

**建议**：只抄第 1 类（拦命令）里的思路，自己按需写；不要整段复制。

## 1.5 插件与状态栏

```json
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
| `%USERPROFILE%\.claude\config.json` | `{"primaryApiKey": "any"}`，用于跳过登录要求 |
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
experimental_bearer_token = "sk-你的Key"  # ⚠️ 字面密钥！换成 env_key 更安全
```

> **改动建议**：把 `experimental_bearer_token` 换成 `env_key = "NEWAPI_KEY"`（教程就是这样写的），这样配置文件里就不含密钥了。
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
sandbox = "elevated"        # ⚠️ 与 danger-full-access 配套，等于不限制

[sandbox_workspace_write]
network_access = true
```

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
| `defaultProjectTrust: "always"` | 不再询问是否信任项目 | ⚠️ 只在自己机器上用；别人的仓库不要这么设 |
| `enabledModels` | 限定 `/model` 与 `Ctrl+P` 循环里的模型 | 抄（按自己的清单改） |
| `packages` | 已装的插件包 | 按需一个个加 |
| `shellPath` | 指向 PowerShell 7 | 路径不同就改或删掉 |
| `externalEditor` | `/editor` 用什么打开 | 随意 |

> 我们的 `defaultProvider` 是 `bailian`（另一家供应商、**另一把密钥**），教程里不涉及；你按 [pi](04-pi.md) 配成 `newapi` 即可。

## 3.2 pi 的 `models.json`

我们文件里有三个供应商：`newapi`（教程用这个）、`deepseek`、`bailian`。后两个走各自官方的 Key，与本教程无关。进阶项包括 `compat` 兼容开关（如 `thinkingFormat`、`maxTokensField`）与 `thinkingLevelMap`（把 pi 的思考档位映射到厂商实际支持的档位）——**这些只在模型行为异常时才需要调**，官方文档：<https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/models.md>

---

# 4 OpenCode 完整配置要点

## 4.1 `~/.config/opencode/` 目录内容

| 文件/目录 | 内容 | 建议 |
| --- | --- | --- |
| `opencode.json` | 主配置（v2 格式：`providers` / `package` / `settings`，顶层 `model` 是默认模型） | 抄教程精简版 |
| ↳ 顶层 `model` 键 | 我们设的是 `newapi/deepseek-v4.1-flash` | **必须抄**：不设时 OpenCode 会自己挑，实测会落到内置免费模型 `space-bunny-free`（见 4.2） |
| `cli.json` | v2 的界面偏好（`session.thinking`、`diffs.wrap`、滚动条等） | 随意 |
| `plugins/` | 本地插件（我们装了 `orca-opencode-status`） | 按需 |
| `node_modules/`、`package-lock.json` | 插件依赖，OpenCode 自己维护 | 不要手动改 |
| `oh-my-openagent.json` | 多智能体编排插件配置 | ⚠️ **我们已弃用**：里面引用的模型名（`kimi-k2.7-code`、`gpt-5.5`、`glm-5.1`、`qwen3.7-plus`、`minimax-m3`、`gpt-5.4-mini`）在中转上**已全部失效**（请求会返回 `model_not_found`）。要用的话必须先把模型名换成 [统一接入](02-unified-access.md) 第 4 节清单里的 |
| `service.json` | 后台服务凭据，自动生成 | 不要改 |

## 4.2 关于"能力不强"

OpenCode 的结果不如 Codex / pi，**主要原因是模型**：它本身不产出模型，用我们中转的国产模型时，同样的任务质量天然差一档。想让它表现更好：

1. **先确认真的用上了我们的模型**：`opencode.json` 顶层要有 `model`，且 `opencode run --standalone` 状态行里的模型名要走 `newapi/`。不设 `model` 时它可能悄悄用内置免费模型 `space-bunny-free`——那不是“差一档”，是压根没接上中转；
2. 优先用 `gpt-6-sol` / `claude-opus-5` 这类较强模型；
3. 任务拆小，一次一件事；
4. 让它先给计划再动手（和别的工具一样）。

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
2. **注意权限预填**。Orca 默认给 Claude / Codex 预填权限绕过参数（`--dangerously-skip-permissions`、`--dangerously-bypass-approvals-and-sandbox`）。我们的 Codex 段本身已经是 `danger-full-access` + `[windows] sandbox = "elevated"`，再叠一层就彻底没有边界了。想收着点：`Settings → Agents → Agent Permissions` 改成 **Manual**。

## 5.4 OpenCode 插件 `orca-opencode-status`

4.1 表里的 `orca-opencode-status`，就是这套"状态上报"在 OpenCode 上的对应物（由 Orca 自动安装的状态插件）。删掉它不影响 OpenCode 本身，只是 Orca 看不到那个会话的实时状态。

---

# 6 一张"抄配置"检查表

抄完任何一项，请逐条确认：

- [ ] 密钥还留在环境变量里（配置文件里没有 `sk-` 开头的字符串）
- [ ] 绝对路径（`C:\Users\zsj\...`）都换成了自己的，或删掉了
- [ ] 模型名都在 [统一接入](02-unified-access.md) 第 4 节清单里
- [ ] 权限相关配置（`deny` / `sandbox_mode`）是你**有意**要的，不是顺手抄的
- [ ] 用 Orca 时确认它启动的是 **system default** 的 Codex / Claude（没用 `Add account` 加额外账号，否则读不到上面的中转配置）
- [ ] 抄完跑一次验证命令，确认没坏：`codex exec "只回复两个字：可用"` / `pi -p "只回复两个字：可用"` / `claude -p "只回复两个字：可用"` / `opencode run --standalone "只回复两个字：可用"`
