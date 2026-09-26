# 统一接入

使用本教程默认的 **NewAPI 方案**时，本篇是**唯一需要填写密钥的地方**。后面 Claude Code / Codex / pi / OpenCode 四篇的默认配置都可以直接复制；另接其他供应商（如 OpenCode Console）时，需按对应文档单独配置密钥。

一句话原理：把密钥放进**系统环境变量**，各工具自己去读。这样配置文件里只出现变量名，不出现密钥，谁都能复制、也不怕误传到网上。

---

# 1 你需要准备的东西

| 项目 | 值 | 说明 |
| --- | --- | --- |
| 中转站地址 | `https://newapi.ttxs.site` | 我们统一的模型入口（NewAPI） |
| 密钥 | `sk-你的Key` | **找朱老师要**，禁止分享给任何人 |
| 环境变量名（通用） | `NEWAPI_KEY` | Codex / pi / OpenCode 读这个 |
| 环境变量名（Claude Code 专用） | `ANTHROPIC_AUTH_TOKEN` | Claude Code 只认这个名字 |

> 为什么不让 Claude Code 也读 `NEWAPI_KEY`？因为 Claude Code 的 `settings.json` 不支持 `${变量}` 展开（官方 [issue #4276](https://github.com/anthropics/claude-code/issues/4276) 一直没做），它只认官方约定的 `ANTHROPIC_AUTH_TOKEN` 环境变量。所以 Claude Code 用第二个变量名，值是**同一个 Key**。

## 1.1 变量名的约定

这两个名字是全套文档的约定，四个工具都按这两个名字去读，**名字不要改**（改了各工具的配置文件里的引用也要跟着改）。

各工具怎么引用它们：

| 工具 | 在配置里怎么写 |
| --- | --- |
| Codex | `env_key = "NEWAPI_KEY"` |
| pi | `"apiKey": "$NEWAPI_KEY"` |
| OpenCode | `"apiKey": "{env:NEWAPI_KEY}"` |
| Claude Code | 不写——它自动读 `ANTHROPIC_AUTH_TOKEN` |

---

# 2 设置环境变量

## 2.1 Windows（推荐：永久写入用户变量，只需做一次）

打开 **PowerShell 7**，把下面的 `sk-你的Key` 换成真 Key，**两行一起执行**：

```PowerShell
[Environment]::SetEnvironmentVariable("NEWAPI_KEY", "sk-你的Key", "User")
[Environment]::SetEnvironmentVariable("ANTHROPIC_AUTH_TOKEN", "sk-你的Key", "User")
```

**执行后必须新开一个终端**（旧终端读不到新变量）。验证：

```PowerShell
$env:NEWAPI_KEY.Length
$env:ANTHROPIC_AUTH_TOKEN.Length
```

预期输出：两个数字（我们的 Key 是 51 位）。如果输出为空，说明没新开终端，或者 Key 填错了。

也可以图形化设置：`Win + R` → `sysdm.cpl` → 高级 → 环境变量 → 用户变量 → 新建。

## 2.2 临时设置（只对当前终端窗口有效，适合测试）

```PowerShell
$env:NEWAPI_KEY = "sk-你的Key"
$env:ANTHROPIC_AUTH_TOKEN = "sk-你的Key"
```

## 2.3 远程 Linux 服务器

把下面两行追加到 `~/.bashrc`（注意替换 Key），然后 `source ~/.bashrc`：

```Bash
export NEWAPI_KEY="sk-你的Key"
export ANTHROPIC_AUTH_TOKEN="sk-你的Key"
```

验证：

```Bash
echo ${#NEWAPI_KEY} ${#ANTHROPIC_AUTH_TOKEN}
```

预期输出：`51 51`。

> 服务器上**不需要**装 Claude Code 的图形界面，四个工具都有命令行版本。

---

# 3 地址与协议：同一个中转，两种接口

NewAPI 同时提供两种协议接口，不同工具用不同的那个，**不要互相混用**：

| 工具 | 用哪个协议 | 配置里填的地址 |
| --- | --- | --- |
| Claude Code | Anthropic Messages（`/v1/messages`） | `https://newapi.ttxs.site`（**不带** `/v1`） |
| Codex | OpenAI Responses | `https://newapi.ttxs.site/v1` |
| pi | OpenAI Responses / Completions | `https://newapi.ttxs.site/v1` |
| OpenCode | OpenAI 兼容 | `https://newapi.ttxs.site/v1` |

常见的坑：

- **Claude Code 不能接 OpenAI 协议端点**。它的 `ANTHROPIC_BASE_URL` 必须是说过 Anthropic Messages 协议的地址，填了 OpenAI 兼容地址会直接报 400/404。我们的中转两种都提供，所以按上表填即可。
- 地址**末尾不要多加斜杠**（`.../v1/` 有时会 404）。

---

# 4 当前可用的模型清单

> ## 🔎 实时清单与价格看这里
> **<https://newapi.ttxs.site/pricing>**
>
> 这是中转后台的计费页，**随时随地打开就是最新的**：当期有哪些模型可用、每个模型输入/输出每 1M tokens 多少钱，都在这一页。模型上架、下架、调价都在这里反映。
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
| Codex | `gpt-6-sol` | `gpt-6-luna`、`gpt-5.6-sol` | Codex 与 GPT 系列同源，工具调用最稳（贵一些，追求效果就用它） |
| pi | `deepseek-v4.1-flash` | `glm-5.3-flash`、`gpt-6-sol` | 长上下文 + 工具调用稳；想更省更快就换 `glm-5.3-flash` |
| Claude Code | `deepseek-v4.1-flash` | `glm-5.3-flash`、`qwen3.8-max` | 日常写代码够用，成本低 |
| OpenCode | `deepseek-v4.1-flash` | `glm-5.3-flash`、`gpt-6-sol`、`deepseek-v4-pro` | 通用性好，插件生态用它最省心 |

想省钱的时候：先到 [pricing 页](https://newapi.ttxs.site/pricing) 比一下单价，把"主力"换成便宜的 flash 档（`glm-5.3-flash`、`deepseek-v4-flash` 这类）即可，改一处配置就生效。

## 4.2 模型名不要乱写

- **不要写空格、不要写显示名**。写 `gpt-6-sol`，不要写 `GPT-6 Sol`。
- **`[1m]` 后缀只对 Claude Code 有意义**：它是 Claude Code 客户端自己的写法（表示开启 1M 上下文），Claude Code 会把后缀处理掉再发请求。直接把 `deepseek-v4.1-flash[1m]` 发给中转会返回 `model_not_found`，**所以别把这个后缀抄到 Codex / pi / OpenCode 里**。
- 模型名写错的表现：`503 model_not_found` / `No available channel for model xxx`。

---

# 5 四个工具怎么读这个 Key（速查表）

| 工具 | 配置文件 | 写法 |
| --- | --- | --- |
| Claude Code | `%USERPROFILE%\.claude\settings.json` | 不写 Key，直接读环境变量 `ANTHROPIC_AUTH_TOKEN` |
| Codex | `%USERPROFILE%\.codex\config.toml` | `env_key = "NEWAPI_KEY"` |
| pi | `%USERPROFILE%\.pi\agent\models.json` | `"apiKey": "$NEWAPI_KEY"` |
| OpenCode | `%USERPROFILE%\.config\opencode\opencode.json` | `"apiKey": "{env:NEWAPI_KEY}"` |

四家的写法各不相同，**别抄错**：Claude Code 什么都不写（直接读环境变量）；Codex 只写变量名（不加 `$`）；pi 用 `$变量名`；OpenCode 用 `{env:变量名}`。

---

# 6 常见报错对照

## 6.1 密钥没生效：四家报错长得完全不一样

四家都不会说“你的环境变量没设”这种人话，签名各不相同（**下表均为实测**）。看到其中任意一条，先按 2.1 重设变量、**新开一个终端**，再用 `$env:NEWAPI_KEY.Length` 确认输出 `51`。

| 工具 | 报错原文 | 出现时机 | 别踩的坑 |
| --- | --- | --- | --- |
| Claude Code | `Not logged in · Please run /login` | 立即 | 别真去 `/login`——那是 Anthropic 官方登录流程，和中转无关 |
| Codex | `ERROR: Missing environment variable: NEWAPI_KEY.` | **约 1 秒** | 反复 `ERROR: Reconnecting... 1/5` 是另一回事：变量读到了，**令牌本身不对** |
| pi | `No API key found for newapi.` | 约 4 秒 | 它会提示你 `/login`，**照做就把明文密钥写进 `auth.json`**，而 `auth.json` 优先级高于 `models.json` 的 `apiKey`，之后改环境变量都不再生效。清理命令见[附录](10-appendix-full-config.md) 3.2 |
| OpenCode | `Error: Invalid token` | 立即 | 也可能是把 `{env:NEWAPI_KEY}` 误写成了 `$NEWAPI_KEY`（那是 pi 的语法） |

> **关键是分清“变量没读到”和“令牌是错的”**：前者秒级失败、且明说缺什么；后者会反复重试或直接 401。两者修法完全不同。
> pi 还有个好用探针：`pi --list-models`。**有密钥时列出 newapi 的模型，没密钥时一个都不列**，并对 `settings.json` 的 `enabledModels` 逐条报 `Warning: No models match pattern "newapi/..."`。

## 6.2 其他常见报错

| 现象 | 原因 | 解决 |
| --- | --- | --- |
| `401 Invalid token` / `Authentication failed` | Key 错了、过期了，或额度用完（**不是**没生效，那种情况见 6.1） | 找朱老师核对 Key；`$env:NEWAPI_KEY.Length` 应为 `51`，注意有没有多余空格或引号 |
| `503 model_not_found` / `No available channel for model` | 模型名写错，或该模型当期不在清单里 | 先在 [pricing 页](https://newapi.ttxs.site/pricing) 确认模型还在不在，再按第 4 节核对名字 |
| 请求发出去但一直转圈 | 地址填错（多 `/v1`、少 `/v1`、多了斜杠） | 按第 3 节表格核对 |
| Claude Code 里 `[claude-code:unrecognized_model]` 警告 | Claude Code 不认识第三方模型，**属于正常现象** | 忽略，能正常回答就行 |
| `HTTP 400` 且提示协议相关 | 拿 OpenAI 协议地址喂给了 Claude Code | 见第 3 节 |

---

# 7 安全

- Key 等于你的额度，**不要**贴进聊天群、截图、GitHub、飞书公开文档。
- 配置文件里只写变量名（本篇教程所有示例都如此），所以配置文件本身可以随便分享。
- 万一 Key 泄露：找朱老师换一把，然后按 2.1 重新设置两个环境变量即可。

---

# 8 常用链接

| 用途 | 链接 |
| --- | --- |
| **实时查看可用模型与价格** | <https://newapi.ttxs.site/pricing> |
| 模型列表接口（只给名字，不给价格） | `https://newapi.ttxs.site/v1/models` |
| 中转首页 | <https://newapi.ttxs.site> |
