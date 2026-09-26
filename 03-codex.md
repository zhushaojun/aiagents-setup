# Codex

**这是我们当前的主力工具。** 由 OpenAI 出品，命令行、VS Code 扩展、ChatGPT 桌面应用（Codex 的桌面形态现已并入 ChatGPT）三种形态齐全，在我们的日常任务与当前配置中优先使用。**它的命令行客户端还是开源的**（可审计、可自改，见第 10 节）。

![VS Code 里的 Codex 聊天面板：右侧列出历史 Chats 与 Do anything 输入框，底部显示当前模型 6 Sol High 和 Work locally](images/codex-vscode-chat.png)

---

# 1 它是什么、适合谁

- **定位**：OpenAI 官方的命令行编程智能体，客户端表现应在相同模型与任务条件下比较；三种使用形态（终端 / VS Code 扩展 / ChatGPT 桌面应用）共用同一套配置。
- **适合**：绝大多数日常任务——读代码、改 bug、写脚本、跑命令、查文档。
- **优势**：提供读文件、执行命令和修改代码等工具；**命令行客户端开源**；配置文件简单，CLI 不登录 OpenAI 账号也能使用我们的中转。
- **长任务**：先检索相关文件并分解任务；是否换客户端或模型应结合实际上下文限制与压缩表现判断；Claude Code 的选型与兼容性注意事项见 [Claude Code](06-claude-code.md) 第 8 节。

---

# 2 前置要求

| 项目 | 要求 | 检查方法 |
| --- | --- | --- |
| Node.js | ≥ 22.19.0（本教程统一基线） | `node -v` |
| npm | 随 Node 自带，按网络情况选择镜像 | `npm -v` |
| 环境变量 | `NEWAPI_KEY` 已设置 | `-not [string]::IsNullOrWhiteSpace($env:NEWAPI_KEY)` → `True` |
| VS Code（可选） | 最新版 | `code --version` |

> `NEWAPI_KEY` 怎么设置见 **[统一接入](02-unified-access.md)** 第 2 节。缺变量与服务端拒绝凭据是不同问题，排查见该篇第 6 节。

---

# 3 安装

## 3.1 命令行版（Windows，必装）

```PowerShell
npm install -g @openai/codex
codex --version
```

版本输出示例：`codex-cli 0.157.0`，不是已验证最新版声明；统一记录见 [README 第 4 节](README.md#4-版本基线与核验状态)。

## 3.2 VS Code 扩展

在 VS Code 扩展商店搜索 **Codex**，安装 OpenAI 官方那个（扩展 ID `openai.chatgpt`）：

![VS Code 扩展商店里的 Codex 扩展页面，发布者 OpenAI、标识符 openai.chatgpt](images/codex-vscode-extension.png)

装完后左侧会出现 Codex 图标，点开即可对话。**扩展和命令行共用 `~/.codex/config.toml`**，仍需重启 VS Code 以刷新环境变量，并在扩展中检查认证、模型和权限。

## 3.3 桌面应用（现已改名 ChatGPT）

从微软商店安装 **ChatGPT** 桌面应用：<https://apps.microsoft.com/detail/9plm9xgg6vks>

![Codex 桌面应用主界面：左侧是 New chat / Scheduled / Plugins / Codex++ 和 Projects 列表，中间提示 What should we build in dl-course?，底部显示模型 6 Sol High](images/codex-desktop-app.png)

**注意**：Codex 原来的桌面版现在位于 **ChatGPT 桌面应用**中，不要去找一个叫"Codex 桌面版"的独立安装包。首次启动按应用提示认证：可登录 ChatGPT 账号，也可通过 API key 使用 Codex，但[部分功能可能不可用](https://learn.chatgpt.com/docs/quickstart)。本教程的 `NEWAPI_KEY` 用于中转模型请求，不能代替桌面应用的首次认证。进入应用后选择 **Codex**，它会读取同一份 `~/.codex/config.toml`；我们的电脑里它还提供"computer use"（让模型操作浏览器/桌面）能力。

## 3.4 远程 Linux 服务器

```Bash
npm install -g @openai/codex
mkdir -p ~/.codex

(
set -e
set -o noclobber
if [ -e ~/.codex/config.toml ] || [ -L ~/.codex/config.toml ]; then
  printf '文件已存在，请按统一接入第 9 节备份并手动合并。\n' >&2
  exit 1
fi
cat > ~/.codex/config.toml << 'EOF'
model = "gpt-6-sol"
model_provider = "newapi"
model_reasoning_effort = "medium"
approval_policy = "on-request"
sandbox_mode = "workspace-write"

[model_providers.newapi]
name = "NewAPI"
base_url = "https://newapi.ttxs.site/v1"
wire_api = "responses"
env_key = "NEWAPI_KEY"
requires_openai_auth = false

[sandbox_workspace_write]
network_access = true
EOF
)
```

凭据设置与已有 `~/.bashrc` 的处理见 [统一接入第 2.3 节](02-unified-access.md#23-远程-linux-服务器)，不要重复追加同名变量。

---

# 4 配置（首次创建，已有文件先备份合并）

## 4.1 主配置：`%USERPROFILE%\.codex\config.toml`

以下首次创建步骤遇到已有文件会停止。已有配置先按 [统一接入第 9 节](02-unified-access.md#9-已有配置的备份与合并)备份，再手动合并供应商、模型与权限字段；全局提示词也按此规则处理。

```PowerShell
$configPath = "$env:USERPROFILE\.codex\config.toml"
if (Test-Path -LiteralPath $configPath) {
  throw '文件已存在：请按统一接入第 9 节备份后手动合并，不要整份覆盖。'
}
New-Item -ItemType Directory -Force (Split-Path -Parent $configPath) -ErrorAction Stop | Out-Null
@'
# ===== 模型 =====
model = "gpt-6-sol"
model_provider = "newapi"
model_reasoning_effort = "medium"
personality = "pragmatic"

# ===== 权限 =====
approval_policy = "on-request"
sandbox_mode = "workspace-write"
web_search = "live"

# ===== 走我们的中转（密钥从 NEWAPI_KEY 环境变量读，配置文件里没有 Key）=====
[model_providers.newapi]
name = "NewAPI"
base_url = "https://newapi.ttxs.site/v1"
wire_api = "responses"
env_key = "NEWAPI_KEY"
requires_openai_auth = false

# 沙箱内命令允许联网；web_search 由上面的独立选项控制
[sandbox_workspace_write]
network_access = true
'@ | Set-Content -LiteralPath $configPath -Encoding utf8 -ErrorAction Stop
```

**关键项说明**：

| 配置项 | 作用 | 可改成 |
| --- | --- | --- |
| `model` | 用哪个模型 | `gpt-6-luna`、`gpt-5.6-sol`；清单见 [统一接入](02-unified-access.md) 第 4 节 |
| `model_provider = "newapi"` | 用下面定义的 `[model_providers.newapi]` | 不要删 |
| `model_reasoning_effort` | 思考强度 | `low`（快）/ `medium`（默认）/ `high`、`xhigh`、`max`（慢而强） |
| `approval_policy = "on-request"` | 模型需要额外权限时问你 | `never`=从不问（**非交互式跑用这个**） |
| `sandbox_mode = "workspace-write"` | 限制写入到工作区及获准路径，部分配置路径另受保护 | 见第 7 节"进阶：权限" |
| `env_key = "NEWAPI_KEY"` | **密钥来源** | 只写变量名，不要加 `$` |
| `requires_openai_auth = false` | 不使用 OpenAI 认证，默认值也是 `false` | 本例显式填写以表达自定义供应商的认证意图 |

> 用 `env_key = "NEWAPI_KEY"` 这种写法时，**不需要** `codex login`，也不要把 Key 硬写进配置文件。

`requires_openai_auth` 的默认值见 [官方配置参考](https://learn.chatgpt.com/docs/config-file/config-reference)。省略这个字段本身不会开启 OpenAI 认证；桌面入口的首次认证另见第 3.3 节。

> ⚠️ **千万别写 `approval_policy = "untrusted"`**：官方从 **0.149.0** 起移除了这个值，而且是硬失败——配置里留着它 Codex **直接拒绝启动**：
> `Error loading configuration: approval_policy = "untrusted" is no longer supported; remove this setting`
> `on-failure` 也一并废弃了。**记住这两句：交互式用 `on-request`，非交互式（脚本、CI）用 `never`。**

## 4.2 让它说中文：`%USERPROFILE%\.codex\AGENTS.md`

```PowerShell
$configPath = "$env:USERPROFILE\.codex\AGENTS.md"
if (Test-Path -LiteralPath $configPath) {
  throw '文件已存在：请按统一接入第 9 节备份后手动合并，不要整份覆盖。'
}
New-Item -ItemType Directory -Force (Split-Path -Parent $configPath) -ErrorAction Stop | Out-Null
@'
- 始终用中文回答
- 修改代码前先说明要改哪些文件
- 复杂任务先给出计划，得到确认后再动手
- 当前系统是 Windows 11，命令行优先用 PowerShell
'@ | Set-Content -LiteralPath $configPath -Encoding utf8 -ErrorAction Stop
```

`AGENTS.md` 是"随身的提示词"，Codex 每次干活都会读。**项目根目录**下也可以放一个 `AGENTS.md`，只对那个项目生效（推荐写项目的技术栈、运行命令、代码规范）。

---

# 5 验证（第一次使用必须做）

## 5.1 本地环境与配置

```PowerShell
codex --version
Test-Path -LiteralPath (Join-Path $env:USERPROFILE '.codex/config.toml') -PathType Leaf
```

版本应正常返回，文件检查应为 `True`。凭据非空检查见 [统一接入第 2 节](02-unified-access.md#2-设置环境变量)，版本基线见 [README 第 4 节](README.md#4-版本基线与核验状态)。

## 5.2 请求链路

若已按 README 创建练习仓库和样例文件，直接复用；否则先按 [前置工具第 6.1 节](01-prerequisites.md#61-创建独立练习目录)准备，保持终端位于该目录。Codex 的非交互调用默认要求 Git 仓库；无需提交文件。

```PowerShell
codex exec -c 'model_provider="newapi"' --model gpt-6-sol "只回复两个字：可用"
```

预期收到正常回答。它只验证所选模型的基本请求链路；确认实际供应商和模型符合命令与配置，失败按 [统一接入第 6 节](02-unified-access.md#6-常见报错对照)排查。

## 5.3 文件读取与只读命令

仍在上述练习目录启动交互模式：

```PowerShell
codex -c 'model_provider="newapi"' --model gpt-6-sol
```

输入以下提示（不要提前告诉模型文件里的随机文本）：

```text
读取当前目录的 agent-check.txt，原样报告其中的文本；实际执行 git status --short 并报告输出。不要创建、修改或删除任何文件。
```

成功标准：能在工具调用记录中看到读取文件及执行命令，读出的文本与自己准备的随机文本一致，Git 输出包含未跟踪的 `agent-check.txt`。需要权限时先核对命令和目标目录再确认。仅凭模型口头说“已执行”不算通过。

这一步不验证图片、写文件、长上下文或最大输出。进入真实项目时，把示例路径 `D:\codes\some-project` 换成自己的路径；没有 D 盘可继续使用用户目录下的练习目录。

---

# 6 日常用法

| 你想做的事 | 怎么做 |
| --- | --- |
| 进入交互界面 | 在项目目录里运行 `codex` |
| 继续上次对话 | `codex resume` |
| 一次性跑完不问 | `codex exec "把 xxx 改成 yyy"` |
| 让它在只读模式分析（不改文件） | `codex exec --sandbox read-only "分析这个 bug"` |
| 切换模型 | 界面里 `/model` |
| 换思考强度 | 界面里 `/reasoning`（或改配置里的 `model_reasoning_effort`） |
| 看当前会话用量 | 界面里 `/status` |
| 退出 | `/quit` 或 `Ctrl+C` |

**有效提问的三个习惯**（对四个工具都适用）：

1. **给判断标准**："改成用 pandas 读取，要求能处理缺失值"；
2. **要它先给计划**："先列出你要改的文件，我同意后再改"；
3. **一次一件事**，别把十个需求塞进一句话。

---

# 7 进阶

## 7.1 权限与沙箱：为什么默认不用最大权限

教程默认 `sandbox_mode = "workspace-write"` + `approval_policy = "on-request"`：前者限制文件及网络访问边界，后者决定何时请求额外权限。工作区之外还可能包含临时目录或显式授权路径，`.git`、`.codex` 等路径可能额外受保护。实际边界以当前运行环境为准。

我们自己的机器上用的是更激进的配置：

```toml
sandbox_mode = "danger-full-access"
[windows]
sandbox = "elevated"
```

`danger-full-access` 关闭沙盒限制，但仍受操作系统权限与组织策略约束；是否审批由独立的审批设置决定。`[windows] sandbox = "elevated"` 选择 Windows 原生沙盒实现，本身不表示关闭沙盒，也不是自动批准所有命令。见 [官方配置说明](https://learn.chatgpt.com/docs/config-file/config-basic)。

![Codex 的权限确认弹窗：标题栏显示 Action Required，正文给出 Environment: local、Reason（目标路径在工作区外）和它要执行的命令，底部是 1. Yes, proceed (y) 与 2. No, and tell Codex what to do differently (esc)](images/codex-permission-prompt.png)

**建议**：保留教程默认权限；确需最大权限时，只在另有独立隔离与恢复条件的环境使用。Git 不能保护仓库外文件、未跟踪文件或凭据，不能作为关闭沙盒的理由。

## 7.2 常用可选配置

```toml
# 状态栏显示什么（可选）
[tui]
status_line = ["model-with-reasoning", "current-dir", "git-branch", "context-used", "approval-mode"]

# 多套配置档：$CODEX_HOME/<名字>.config.toml，用 codex --profile <名字> 切换
# 例：把"写文档"和"改代码"分成两套模型/权限
```

## 7.3 接入 MCP / 技能 / 插件

Codex 支持 MCP 服务器、`skills`、`plugins`。技能用 `/skills` 挑选或 `$技能名` 显式调用（**不支持 `/技能名` 这种斜杠写法**）；现成的第三方技能包与装法见 [Matt Skills](09-matt-skills.md)。

```toml
[mcp_servers.context7]
command = "npx"
args = ["-y", "@upstash/context7-mcp"]
```

配置项较多，建议在**确认过基础用法**之后再折腾。我们自己的完整配置（含桌面段、插件市场、MCP）见 [附-完整配置](10-appendix-full-config.md)。

---

# 8 常见问题

| 现象 | 可能原因、检查顺序与下一步 |
| --- | --- |
| `401 Invalid token` | 可能是凭据错误、过期或配置冲突；先核对变量与引用，再按 [统一接入第 6 节](02-unified-access.md#6-常见报错对照)检查服务端响应 |
| 一直提示登录 OpenAI | 先确认实际 `model_provider` 为 `newapi`、加载的是预期用户级配置及 `env_key`，再核对所用入口的首次认证要求（桌面入口见第 3.3 节）；不能归因于漏写默认值为 `false` 的 `requires_openai_auth` |
| `model_not_found` / `No available channel for model` | 先核对实际模型名与当前凭据的模型列表，再检查渠道和权限；两类错误不等价，见 [统一接入第 6 节](02-unified-access.md#6-常见报错对照) |
| 想让它在别的盘干活 | 先 `cd` 到那个目录再运行 `codex`；具体可写范围见第 7.1 节 |
| VS Code 扩展里报错 | 扩展与命令行共用配置；先在命令行确认 `codex exec "只回复两个字：可用"` 能正常回答 |
| 中文乱码（Windows） | 用 **Windows Terminal + PowerShell 7**，不要用老的 cmd 窗口 |
| 回答太慢 | `model_reasoning_effort` 降到 `low`，或换 `gpt-6-luna` |
| 上下文不够用 | 换上下文更大的模型，或用 `/compact` 压缩会话 |

---

# 9 升级与卸载

```PowerShell
npm install -g @openai/codex     # 升级到最新版
npm uninstall -g @openai/codex   # 卸载（配置目录 ~/.codex 不会被删）
```

VS Code 扩展和 ChatGPT 桌面应用各自在应用内升级。配置、会话、`AGENTS.md` 都保存在 `%USERPROFILE%\.codex`，卸载重装不会丢。

---

# 10 参考资料

1. Codex 官方文档：<https://developers.openai.com/codex>
2. 源码（命令行客户端开源）：<https://github.com/openai/codex>
3. VS Code 扩展（扩展 ID `openai.chatgpt`）：<https://marketplace.visualstudio.com/items?itemName=openai.chatgpt>
4. 桌面应用（微软商店，装 ChatGPT，Codex 的桌面形态已并入其中）：<https://apps.microsoft.com/detail/9plm9xgg6vks>
5. 配置项完整参考：<https://developers.openai.com/codex/config>
6. 统一接入与模型清单：见本仓库 [统一接入](02-unified-access.md)
7. **实时查看可用模型与价格**：<https://newapi.ttxs.site/pricing>
