# 统一接入

使用本教程默认的 **NewAPI 方案**时，Codex / OpenCode / Claude Code 在本篇设置密钥，**pi 另按 [pi 第 4.1 节](04-pi.md#41-接入-newapiuserprofilepiagentmodelsjson)将密钥写入 `auth.json`**。后面 Claude Code / Codex / pi / OpenCode 四篇的默认配置可用于首次创建，已有文件先备份合并；另接其他供应商（如 OpenCode Console）时，需按对应文档单独配置密钥。

密钥来源：Codex / OpenCode / Claude Code 读取**用户级环境变量**；pi 使用 **`auth.json` 中的真实 API Key**。当前使用环境中，pi 原教程的环境变量方案未生效，因此按现用的文件凭据方案配置。

---

# 1 你需要准备的东西

| 项目 | 值 | 说明 |
| --- | --- | --- |
| 中转站地址 | `https://newapi.ttxs.site` | 我们统一的模型入口（NewAPI） |
| 密钥 | `sk-你的Key` | 使用你自己的有效密钥，禁止分享；服务入口：[NewAPI](https://newapi.ttxs.site) |
| 环境变量名（通用） | `NEWAPI_KEY` | Codex / OpenCode 读这个；pi 使用 `auth.json` |
| 环境变量名（Claude Code 专用） | `ANTHROPIC_AUTH_TOKEN` | 本教程选用的 Bearer 鉴权变量 |

> 本教程为 Claude Code 选用 `ANTHROPIC_AUTH_TOKEN`，由客户端生成 Bearer 请求头。它也支持 `ANTHROPIC_API_KEY`（`X-Api-Key` 请求头）等官方鉴权方式；选用哪种取决于服务端。本教程不依赖 `settings.json` 中的 `${变量}` 展开，避免把真实密钥写入配置。参见 [官方环境变量说明](https://code.claude.com/docs/en/env-vars)。

## 1.1 变量名的约定

这两个名字是全套文档的约定，Codex / OpenCode / Claude Code 按这两个名字去读；pi 不使用这两个变量，**名字不要改**（改了各工具的配置文件里的引用也要跟着改）。

各工具怎么引用它们：

| 工具 | 在配置里怎么写 |
| --- | --- |
| Codex | `env_key = "NEWAPI_KEY"` |
| pi | `auth.json`：`"newapi": { "type": "api_key", "key": "sk-你的Key" }` |
| OpenCode | `"apiKey": "{env:NEWAPI_KEY}"` |
| Claude Code | 不写——它自动读 `ANTHROPIC_AUTH_TOKEN` |

---

# 2 设置环境变量

本节用于 Codex / OpenCode / Claude Code；只使用 pi 时，直接按 [pi 第 4.1 节](04-pi.md#41-接入-newapiuserprofilepiagentmodelsjson)填写 `auth.json`。

## 2.1 Windows（推荐：永久写入用户变量，只需做一次）

打开 **PowerShell 7**，把下面的 `sk-你的Key` 换成真 Key，**两行一起执行**：

```PowerShell
[Environment]::SetEnvironmentVariable("NEWAPI_KEY", "sk-你的Key", "User")
[Environment]::SetEnvironmentVariable("ANTHROPIC_AUTH_TOKEN", "sk-你的Key", "User")
```

用户级设置只影响之后继承到新环境的进程，**不会修改当前终端的 `$env:`**。新标签页也可能继承旧父进程的环境。请完整退出并重启 Windows Terminal、VS Code、桌面应用或 Orca，再验证；只想测试当前终端可以使用 2.2。

```PowerShell
-not [string]::IsNullOrWhiteSpace($env:NEWAPI_KEY)
-not [string]::IsNullOrWhiteSpace($env:ANTHROPIC_AUTH_TOKEN)
```

预期为两个 `True`，仅表示变量非空，**不能证明密钥有效**。密钥长度不作为判定标准。若为 `False`，检查用户变量是否保存、变量名是否一致，以及启动程序的父进程是否已重启。

也可以图形化设置：`Win + R` → `sysdm.cpl` → 高级 → 环境变量 → 用户变量 → 新建。

## 2.2 临时设置（只对当前终端窗口有效，适合测试）

```PowerShell
$env:NEWAPI_KEY = "sk-你的Key"
$env:ANTHROPIC_AUTH_TOKEN = "sk-你的Key"
```

## 2.3 远程 Linux 服务器

先按第 9 节备份 `~/.bashrc`，用编辑器检查并设置下面两项（已有同名变量就更新，不重复追加；注意替换 Key），然后 `source ~/.bashrc`：

```Bash
export NEWAPI_KEY="sk-你的Key"
export ANTHROPIC_AUTH_TOKEN="sk-你的Key"
```

验证：

```Bash
for name in NEWAPI_KEY ANTHROPIC_AUTH_TOKEN; do
  if [ -n "${!name}" ]; then printf '%s 已设置\n' "$name"; else printf '%s 未设置\n' "$name"; fi
done
```

预期两项均为“已设置”；这里只检查非空，不验证凭据。上面的读取方式使用 Bash。编辑 `~/.bashrc` 前先备份，已有同名变量应替换旧设置而不是不断追加；该文件含真实密钥，不要分享。

> 服务器上**不需要**装 Claude Code 的图形界面，四个工具都有命令行版本。

---

# 3 地址与协议：同一个中转，不同接口

本教程使用该 NewAPI 服务的 Messages、Responses 和 Chat Completions 接口，不同工具按配置选择，**不要互相混用**：

| 工具 | 用哪个协议 | 配置里填的地址 |
| --- | --- | --- |
| Claude Code | Anthropic Messages（`/v1/messages`） | `https://newapi.ttxs.site`（**不带** `/v1`） |
| Codex | OpenAI Responses | `https://newapi.ttxs.site/v1` |
| pi | OpenAI Responses / Completions | `https://newapi.ttxs.site/v1` |
| OpenCode | OpenAI 兼容 | `https://newapi.ttxs.site/v1` |

常见的坑：

- **Claude Code 不能接 OpenAI 协议端点**。它的 `ANTHROPIC_BASE_URL` 必须是说过 Anthropic Messages 协议的地址，不能只因地址相同就假定兼容。按上表填写，并以响应确认协议；400/404 也可能来自其他路径或参数问题。
- 地址**末尾不要多加斜杠**（`.../v1/` 有时会 404）。

---

# 4 当前可用的模型清单

> ## 🔎 实时清单与价格看这里
> **<https://newapi.ttxs.site/pricing>**
>
> 这是中转的计费查询入口。价格、计费单位和模型展示以页面当前说明为准；页面展示不保证你的凭据有权限，也不保证对应渠道此刻可用。
>
> **清单与价格随时可能变，以这一页为准。** 本文下面那份清单只是快照，用来对照名字、离线看。
> （这是网页应用，需要浏览器打开；看不到内容时先登录中转账号。）

下面这份清单是**快照**（用我们的 Key 查中转的模型列表接口得到，2026-09）：

```
claude-fable-5-1            claude-opus-5               claude-opus-5-5
codex-auto-review           deepseek-v4-flash           deepseek-v4-pro
deepseek-v4.1-flash         glm-5.3                     glm-5.3-flash
gpt-5.6-luna                gpt-5.6-sol                 gpt-5.6-terra
gpt-6-astra                 gpt-6-luna                  gpt-6-sol
gpt-image-2                 gpt-image-2.5-flare         gpt-image-2.5-sunburst
grok-4.7                    mimo-v2.6-flash             mimo-v2.6-pro
muse-spark-1.3-contributor  openrouter/free             qwen3.8-flash
qwen3.8-max                 space-bunny-free
```

自己查一遍（随时确认，模型会变；**注意这个接口只给模型名，不给价格**）：

```PowerShell
(Invoke-RestMethod -Uri "https://newapi.ttxs.site/v1/models" -Headers @{ Authorization = "Bearer $env:NEWAPI_KEY" }).data.id |
  Sort-Object
```

## 4.1 各工具主力模型推荐

| 工具 | 主力 | 备选 | 理由 |
| --- | --- | --- | --- |
| Codex | `gpt-6-sol` | `gpt-6-luna`、`gpt-5.6-sol` | 本教程的默认搭配；效果与费用需结合任务验证 |
| pi | `deepseek-v4.1-flash` | `glm-5.3-flash`、`gpt-6-sol` | 用于试用不同模型；长上下文和工具调用需分别验证 |
| Claude Code | `deepseek-v4.1-flash` | `glm-5.3-flash`、`qwen3.8-max` | 我们的日常配置；价格以当前计费页为准 |
| OpenCode | `deepseek-v4.1-flash` | `glm-5.3-flash`、`gpt-6-sol`、`deepseek-v4-pro` | 保持与 pi 相同的默认模型，便于对照客户端行为 |

想控制费用时，先在 [pricing 页](https://newapi.ttxs.site/pricing) 比较实际单价及计费单位；不要只根据 flash 名称判断价格。切换模型后核对客户端注册、模型选择及兼容性，再执行短文本验证。

默认值以上表为准，各工具示例是它的可执行展开。模型列表只返回名称，不能证明上下文、图片、工具调用或输出上限。pi / OpenCode 示例中的上限来自旧配置记录，本次未取得可复核的中转能力元数据，均作为**待验证的配置值**；在长任务前向服务维护者确认该模型、协议和渠道实际限制。[1M] 后缀也不会让服务端自动获得更大窗口。

## 4.2 模型名不要乱写

- **不要写空格、不要写显示名**。写 `gpt-6-sol`，不要写 `GPT-6 Sol`。
- **`[1m]` 后缀只对 Claude Code 有意义**：它是 Claude Code 客户端自己的写法（用于声明 1M 上下文），原教程记录客户端会处理后缀后再请求，具体版本与中转组合仍需复测。该后缀不在本教程的中转模型快照中，直接作为服务端模型名可能返回 `model_not_found`，**所以别把这个后缀抄到 Codex / pi / OpenCode 里**。
- 模型名错误可能出现 `model_not_found`；`No available channel for model` 也可能是渠道或权限问题，按第 6 节分开排查。

---

# 5 四个工具怎么读这个 Key（速查表）

| 工具 | 配置文件 | 写法 |
| --- | --- | --- |
| Claude Code | `%USERPROFILE%\.claude\settings.json` | 不写 Key，直接读环境变量 `ANTHROPIC_AUTH_TOKEN` |
| Codex | `%USERPROFILE%\.codex\config.toml` | `env_key = "NEWAPI_KEY"` |
| pi | `%USERPROFILE%\.pi\agent\auth.json` | `"newapi": { "type": "api_key", "key": "sk-你的Key" }` |
| OpenCode | `%USERPROFILE%\.config\opencode\opencode.json` | `"apiKey": "{env:NEWAPI_KEY}"` |

四家的写法各不相同，**别抄错**：Claude Code 什么都不写（直接读环境变量）；Codex 只写变量名（不加 `$`）；pi 将真实 Key 写入 `auth.json` 的 `newapi.key`；OpenCode 用 `{env:变量名}`。

---

# 6 常见报错对照

先记录客户端版本、实际供应商、模型、HTTP 状态码和脱敏错误信息。本节整理原教程基线版本（见 [README 第 4 节](README.md#4-版本基线与核验状态)）的观察与排查建议；未保留逐条测试日志，因此报错字样和等待时长不是通用判据。

## 6.1 密钥与环境变量

| 现象 | 可能原因 | 检查顺序 | 下一步 |
| --- | --- | --- | --- |
| Codex：`Missing environment variable: NEWAPI_KEY` | 当前进程未读到变量，或 `env_key` 写错 | 核对变量名 → 非空检查 → 重启父应用 | 按第 2 节重新加载环境，不必先更换密钥 |
| pi：`No API key found for newapi.` | 配置未加载或文件凭据缺失 | 核对目录和 JSON → `auth.json` 的 `newapi.type` 与 `newapi.key` | 按附录 3.2 检查格式和非空状态，按 pi 第 4.1 节补齐凭据 |
| Claude Code：要求登录 | 当前鉴权未生效、配置冲突或入口认证要求 | 确认 `ANTHROPIC_AUTH_TOKEN` → 地址 → CLI 与扩展分别验证 | 查当前版本鉴权文档，不把所有登录提示都当成密钥错误 |
| `401` / `Invalid token` / `Authentication failed` | 凭据错误、失效、被旧凭据覆盖，或引用未解析 | 核对变量与引用语法 → 凭据优先级 → 服务端响应 | 请服务维护者核对密钥状态；额度问题按响应及后台确认 |
| pi 的模型列表没有 `newapi` | 配置目录不对、格式错误、过滤规则或凭据不可用 | 核对配置路径 → `providers` → `enabledModels` → 凭据 | `--list-models` 是本地检查，不证明服务端接受密钥 |

不要打印密钥，也不要将完整凭据文件贴进聊天。分享错误信息时删掉请求头、令牌和私人路径。

## 6.2 请求、模型与工具能力

| 现象 | 可能原因 | 检查顺序 | 下一步 |
| --- | --- | --- | --- |
| `Reconnecting`、超时、一直等待 | 网络、代理、中转异常、协议不兼容或服务端拒绝请求 | 看完整错误与状态码 → 地址/协议 → 服务状态 | 保存脱敏日志；服务入口：[NewAPI](https://newapi.ttxs.site)。不能仅凭重连判定密钥错误 |
| `model_not_found` | 模型名错误、下架或当前凭据无权限 | 对照第 4 节实时列表 → 检查实际请求模型 | 更正模型名或询问维护者 |
| `No available channel for model` | 模型存在但当前渠道不可用、分组权限或服务端故障 | 模型列表 → 凭据权限 → 后台渠道状态 | 由服务维护者确认，不能只靠改模型拼写 |
| 客户端选择器找不到模型 | 客户端未注册、配置未加载或被过滤 | 配置目录 → 模型条目 → 客户端过滤规则 | 修正客户端配置；与服务端缺模型分开处理 |
| `400` / `404` | 路径、协议、参数或输入类型不被支持 | 第 3 节地址 → 错误响应 → 模型能力 | 用最短文本请求定位，再逐项恢复参数 |
| Claude Code 未识别第三方模型警告 | 客户端对上下文等能力采用默认假设 | 确认模型能力 → 长会话是否异常 | 见 Claude Code 第 8.2 节，不一律忽略 |
| 能聊天但不能执行命令 | Shell 路径、权限、工具调用兼容性或运行环境问题 | 检查 Shell → 工具权限 → 只读命令验证 | 见前置工具及对应客户端第 5 节 |

---

# 7 安全

- Key 等于你的额度，**不要**贴进聊天群、截图、GitHub、飞书公开文档。
- pi 的 `auth.json` 保存明文密钥，不能分享；其他工具的示例引用环境变量。你自己的文件还可能含其他服务凭据、私有路径、插件配置和历史密钥，分享前必须检查。环境变量也不等于加密存储，能读取进程环境的程序仍可能取得它。
- 万一 Key 泄露：撤销泄露的密钥并换新，按 2.1 更新两个变量，同时更新 pi 的 `auth.json` 中的 `newapi.key`，再重启相关程序。服务入口：[NewAPI](https://newapi.ttxs.site)。

---

# 8 常用链接

| 用途 | 链接 |
| --- | --- |
| **实时查看可用模型与价格** | <https://newapi.ttxs.site/pricing> |
| 模型列表接口（只给名字，不给价格） | `https://newapi.ttxs.site/v1/models` |
| 中转首页 | <https://newapi.ttxs.site> |


# 9 已有配置的备份与合并

后续四篇的首次配置命令遇到已有文件会停止，不会覆盖。此时先备份，再在编辑器里合并相应字段，保留其他供应商、插件、权限和提示词。**JSON 不能直接拼接两个对象，TOML 不应重复声明同一个表。**

PowerShell 备份示例（将目标替换为要编辑的那个文件；提示词文件也适用）：

```PowerShell
$configPath = Join-Path $env:USERPROFILE '.codex/config.toml'
if (-not (Test-Path -LiteralPath $configPath -PathType Leaf)) { throw '目标文件不存在，请使用首次配置步骤。' }
$backupPath = $configPath + '.' + (Get-Date -Format 'yyyyMMdd-HHmmss-fffffff') + '.bak'
Copy-Item -LiteralPath $configPath -Destination $backupPath -ErrorAction Stop
Write-Output "已备份到 $backupPath；请手动合并后验证。"
```

Linux Bash 示例：

```Bash
config_path="$HOME/.codex/config.toml"  # 改为实际目标
if [ -f "$config_path" ]; then
  backup_path="$(mktemp "${config_path}.$(date +%Y%m%d-%H%M%S).XXXXXX.bak")" &&
    cp -p -- "$config_path" "$backup_path" && printf '已备份到 %s\n' "$backup_path"
else
  printf '目标文件不存在，请使用首次配置步骤。\n'
fi
```

备份可能含密钥，按原文件同样保管，不提交、不分享。先核对备份确实成功，再编辑；如需回退，关闭使用该配置的程序，确认回退会放弃本次修改后恢复备份。

验证按三层进行：本地环境和配置 → 指定模型的短文本请求 → 练习目录的读文件与只读命令。短文本成功不代表图片、长上下文或全部工具已经兼容。

**JSON 合并示例：pi 的 `models.json`。** 下例只演示供应商合并，`models: []` 表示未展示模型条目，不是可直接运行的完整接入方案。实际编辑时保留已有模型列表；首次添加 NewAPI 时使用 [pi 第 4.1 节](04-pi.md#41-接入-newapiuserprofilepiagentmodelsjson)的完整模型条目。`other` 是演示用供应商，已有文件中保留自己的真实条目，不要新增这个示例供应商。

合并前（已有其他供应商，以及需要更新的同名 NewAPI）：

```json
{
  "providers": {
    "other": {
      "baseUrl": "https://example.invalid/v1",
      "api": "openai-responses",
      "models": []
    },
    "newapi": {
      "baseUrl": "https://old.example.invalid/v1",
      "api": "openai-responses",
      "models": []
    }
  }
}
```

合并后（仅更新 `newapi` 的地址，保留 `other` 及已有模型；密钥单独写入 `auth.json`）：

```json
{
  "providers": {
    "other": {
      "baseUrl": "https://example.invalid/v1",
      "api": "openai-responses",
      "models": []
    },
    "newapi": {
      "baseUrl": "https://newapi.ttxs.site/v1",
      "api": "openai-responses",
      "models": []
    }
  }
}
```

本例省略其他供应商的认证配置，合并时保留其原有字段。若 `providers.newapi` 仍有旧的 `apiKey` 环境变量引用，移除该字段，并按 pi 第 4.1 节配置 `auth.json`。

原来没有 `newapi` 时，在现有 `providers` 对象内添加一次；已经有时直接修改该对象中的同名字段，不再粘贴第二个 `providers` 或 `newapi`。不要把 `models.json` 的合并结果写进 `settings.json`：默认供应商和模型仍按 pi 第 4.2 节设置，本例不改变默认项。

**TOML 合并示例：Codex 的 `config.toml`。** 下例演示切换默认模型并更新同名供应商；其他权限、插件等配置继续保留。`other` 同样仅供演示。

合并前：

```toml
model = "existing-model"
model_provider = "other"
approval_policy = "on-request"
sandbox_mode = "workspace-write"

[model_providers.other]
name = "Other"
base_url = "https://example.invalid/v1"
wire_api = "responses"
env_key = "OTHER_KEY"

[model_providers.newapi]
name = "NewAPI"
base_url = "https://old.example.invalid/v1"
wire_api = "responses"
env_key = "OLD_NEWAPI_KEY"
```

合并后：

```toml
model = "gpt-6-sol"
model_provider = "newapi"
approval_policy = "on-request"
sandbox_mode = "workspace-write"

[model_providers.other]
name = "Other"
base_url = "https://example.invalid/v1"
wire_api = "responses"
env_key = "OTHER_KEY"

[model_providers.newapi]
name = "NewAPI"
base_url = "https://newapi.ttxs.site/v1"
wire_api = "responses"
env_key = "NEWAPI_KEY"
requires_openai_auth = false
```

顶层 `model`、`model_provider` 在第一个表之前修改，不要追加到 `[model_providers.newapi]` 里面。没有该供应商表时才新增；已有时原地更新，避免重复表或同名键。其他默认项按 [Codex 第 4.1 节](03-codex.md#41-主配置userprofilecodexconfigtoml)核对合并。

**恢复指定备份。** 先关闭使用配置的客户端，确认目标文件和要恢复的备份路径。下面会先保存当前文件，再恢复指定备份；不自动选择最新备份。将示例备份文件名替换为第 9 节备份命令实际输出的路径。

PowerShell 7：

```PowerShell
$configPath = Join-Path $env:USERPROFILE '.codex/config.toml'
$restorePath = Join-Path $env:USERPROFILE '.codex/config.toml.REPLACE-WITH-BACKUP.bak'
if (-not (Test-Path -LiteralPath $configPath -PathType Leaf)) { throw '当前配置不存在，请先核对目标路径。' }
if (-not (Test-Path -LiteralPath $restorePath -PathType Leaf)) { throw '指定备份不存在，请填写实际备份路径。' }
if ((Resolve-Path -LiteralPath $configPath).Path -eq (Resolve-Path -LiteralPath $restorePath).Path) { throw '备份不能是当前配置本身。' }
$beforeRestorePath = $configPath + '.before-restore-' + [guid]::NewGuid().ToString('N') + '.bak'
Copy-Item -LiteralPath $configPath -Destination $beforeRestorePath -ErrorAction Stop
Copy-Item -LiteralPath $restorePath -Destination $configPath -Force -ErrorAction Stop
Write-Output "已恢复指定备份；恢复前的配置保存在 $beforeRestorePath"
```

Linux Bash：

```Bash
(
set -e
config_path="$HOME/.codex/config.toml"
restore_path="$HOME/.codex/config.toml.REPLACE-WITH-BACKUP.bak"
if [ ! -f "$config_path" ] || [ ! -f "$restore_path" ]; then
  printf '当前配置或指定备份不存在，请核对路径。\n' >&2
  exit 1
fi
if [ "$config_path" -ef "$restore_path" ]; then
  printf '备份不能是当前配置本身。\n' >&2
  exit 1
fi
before_restore_path="$(mktemp "${config_path}.before-restore.XXXXXX.bak")"
cp -p -- "$config_path" "$before_restore_path"
cp -p -- "$restore_path" "$config_path"
printf '已恢复指定备份；恢复前的配置保存在 %s\n' "$before_restore_path"
)
```

恢复后先检查配置语法及本地加载，再重启客户端，按对应教程第 5 节重做验证。恢复配置不等于恢复用户级环境变量；如果旧配置引用不同变量，也要核对其是否可用。两份备份均按原文件同样保管，不提交、不分享。
