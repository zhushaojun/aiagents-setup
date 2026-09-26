# AI Coding 智能体安装配置手册

面向初学者的 **命令行 AI 编程智能体** 安装与配置教程。跟着做，一小时内能让四个命令行工具（外加一个进阶的编排工作台 Orca）在你自己的电脑上跑起来。

**本文档的特点**：所有配置都**复制即用**——配置文件里不出现密钥（密钥放在系统环境变量里），所以你可以直接抄别人的配置，也可以放心把自己的配置发给别人。

---

# 1 先看这一节：装哪几个、按什么顺序

| 优先级 | 工具 | 一句话定位 | 文档 |
| --- | --- | --- | --- |
| ★★★ **主力** | **Codex** | OpenAI 官方，命令行客户端开源；终端 / VS Code 扩展 / ChatGPT 桌面应用三形态，接 GPT 系列工具调用最稳 | [Codex](03-codex.md) |
| ★★ **次主力** | **pi** | 开源轻量、可扩展，接国产模型 + 长上下文最合适 | [pi](04-pi.md) |
| ★ **辅助** | **OpenCode** | 开源通用框架，模型接入灵活；可选用 Console 的限时免费模型 | [OpenCode](05-opencode.md) |
| ★ **辅助** | **Claude Code** | Anthropic 官方，推理细致；接第三方模型时需额外验证兼容性 | [Claude Code](06-claude-code.md) |
| ★★ **进阶（编排层）** | **Orca** | 开源"智能体工作台"（ADE）：一个任务一个 git worktree，多个 CLI 智能体并行赛马，diff 与提交集中处理。**它不接模型、不存密钥**，只是启动上面四个 | [Orca](07-orca.md) |

**建议路径**：先把 **Codex** 用起来（终端与桌面版共用配置，桌面版需单独完成首次认证），再加 **pi**（国产模型、长上下文、插件生态），第三顺位是 **OpenCode**（接入灵活，可按 [OpenCode](05-opencode.md) 第 4.1 节选用 Console 的限时免费模型），最后按需要了解 Claude Code。等所需 CLI 都能单独跑顺、并且开始嫌"一个任务只能串行做"时，再上 **Orca**——它不替代上面四个，而是把它们放进同一个窗口里并行调度。

工具跑顺之后，建议再装一套通用的**技能包**（把资深工程师的做事流程固化成文件，四个工具都能用）：[Matt Skills](09-matt-skills.md)。

---

# 2 五分钟最快路径

```
第 0 步  装前置工具（Node.js / Git for Windows / Windows Terminal+PowerShell 7 / VS Code）
         → 看 01-prerequisites.md，末尾有一键自检脚本

第 1 步  设置两个环境变量（同一个 Key，值一样；设置完必须新开终端）
         NEWAPI_KEY          = sk-你的Key      ← Codex / pi / OpenCode 读它
         ANTHROPIC_AUTH_TOKEN = sk-你的Key      ← Claude Code 读它
         → 看 02-unified-access.md 第 2 节

第 2 步  装主力工具 Codex，复制配置，跑一句验证
         npm install -g @openai/codex
         codex exec "只回复两个字：可用"
         → 看 03-codex.md
```

做完这三步，你已经有主力工具可用了。其余工具按第 1 节的优先级逐个加。

---

# 3 文档地图

教程正文都在仓库根目录（与 `README.md` 同级），截图在 `images/`。（`docs/agents/` 是给本仓库智能体看的工作流规范，读者可跳过。）

**文件名前面的编号就是推荐阅读顺序**，按 `00 → 10` 走即可；只有 `README.md` 和 `AGENTS.md` 不编号。

| 文件 | 内容 | 什么时候看 |
| --- | --- | --- |
| [00-ai-coding.md](00-ai-coding.md) | 总览与选型：命令行主线、桌面入口与四个工具的顺位 | **可选**，想先了解背景时看；想直接动手就从 01 开始 |
| [01-prerequisites.md](01-prerequisites.md) | Node.js、Git for Windows、Windows Terminal + PowerShell 7、VS Code、npm 镜像源、一键自检 | **最先看**，只做一次 |
| [02-unified-access.md](02-unified-access.md) | 默认 NewAPI 方案填写密钥的地方：环境变量怎么设、地址与协议、**可用模型清单与实时价格页链接**、四家写法对照、报错对照 | 装任何工具前先看 |
| [03-codex.md](03-codex.md) | 主力工具：终端 / VS Code 扩展 / ChatGPT 桌面应用 / 远程服务器，完整配置与权限取舍 | 必看 |
| [04-pi.md](04-pi.md) | 次主力：安装（含 Git Bash 依赖）、`models.json` 接 NewAPI、默认模型与插件包 | 必看 |
| [05-opencode.md](05-opencode.md) | 第三顺位：**v2 配置格式**（与网上老教程不兼容）、Console 限时免费模型的可选接入、验证与升级 | 按需 |
| [06-claude-code.md](06-claude-code.md) | 末位：客户端配置、模型档位映射、事实性回答与第三方模型兼容性排查 | 按需 |
| [07-orca.md](07-orca.md) | 进阶：**智能体工作台（ADE）**——worktree 并行、多智能体赛马、共享依赖与 `.env`、Orca CLI | 前四个用顺了再看 |
| [08-cc-switch.md](08-cc-switch.md) | 供应商统一管理/本地路由工具，进阶用 | 用熟了再看 |
| [09-matt-skills.md](09-matt-skills.md) | 进阶：第三方技能包（25 个工程流程技能），装完主力工具后再看 | 想固化"做事流程"时看 |
| [10-appendix-full-config.md](10-appendix-full-config.md) | 我们机器上的进阶配置（脱敏）：hooks、插件、权限、桌面段等 | 想深度定制时看 |
| `images/` | 教程用到的截图，文件名对应工具与章节（如 `codex-vscode-extension.png`） | — |

---

# 4 环境变量总表（默认 NewAPI 方案）

设置方法见 [统一接入](02-unified-access.md) 第 2 节。设置完**必须新开一个终端**才生效。

| 变量名 | 谁读它 | 值 |
| --- | --- | --- |
| `NEWAPI_KEY` | Codex、pi、OpenCode | `sk-你的Key`（找朱老师要） |
| `ANTHROPIC_AUTH_TOKEN` | Claude Code | 同上，**值一样** |

> 这两个名字是全套文档的约定，四个工具都按它们读取密钥。
> 为什么 Claude Code 要单独一个变量名？因为它的配置文件不支持 `${变量}` 展开，只认官方约定的 `ANTHROPIC_AUTH_TOKEN`。

---

# 5 版本校验表

本文档记录的是下表版本上的配置写法。工具更新很快，如果你那边行为不一样，请以官方文档为准，并顺手回来更新这张表。

| 工具 | 本文档对应的版本 | 记录日期 | 配置文件位置 | 启动命令 |
| --- | --- | --- | --- | --- |
| Codex | 0.156.x | 2026-09-25 | `%USERPROFILE%\.codex\config.toml` | `codex` |
| pi | 0.87.x | 2026-09-25 | `%USERPROFILE%\.pi\agent\models.json`、`settings.json` | `pi` |
| Claude Code | 2.1.x | 2026-09-25 | `%USERPROFILE%\.claude\settings.json` | `claude` |
| OpenCode | `@opencode/cli` 2.0.x | 2026-09-25 | `%USERPROFILE%\.config\opencode\opencode.json` | `opencode` |
| Orca | 1.4.x | 2026-09-25 | `%APPDATA%\orca`（应用数据） | `orca`（桌面应用自带 CLI，需在设置里注册） |

装好之后，用这几条命令自己确认一下（能正常回答就说明配置生效了）：

| 工具 | 自检命令 |
| --- | --- |
| Codex | `codex exec "只回复两个字：可用"` |
| pi | `pi -p "只回复两个字：可用"` |
| Claude Code | `claude -p "只回复两个字：可用"` |
| OpenCode | `opencode run --standalone "只回复两个字：可用"` |
| Orca | `orca status --json`（看到 `"state": "ready"` 即 CLI 通了；再在界面里让智能体回一句验证整链） |

> 模型单价本文档不写（会变），**实时价看 <https://newapi.ttxs.site/pricing>**。

---

# 6 四家速查卡

| | Codex | pi | Claude Code | OpenCode |
| --- | --- | --- | --- | --- |
| 配置位置 | `~/.codex/config.toml` | `~/.pi/agent/models.json` | `~/.claude/settings.json` | `~/.config/opencode/opencode.json` |
| 密钥写法 | `env_key = "NEWAPI_KEY"` | `"apiKey": "$NEWAPI_KEY"` | 不写（读 `ANTHROPIC_AUTH_TOKEN`） | `"apiKey": "{env:NEWAPI_KEY}"` |
| 接口地址 | `https://newapi.ttxs.site/v1` | 同左 | `https://newapi.ttxs.site`（**不带 `/v1`**） | `https://newapi.ttxs.site/v1` |
| 默认模型 | `gpt-6-sol` | `deepseek-v4.1-flash` | `deepseek-v4.1-flash[1M]` | `deepseek-v4.1-flash` |
| 说中文 | `~/.codex/AGENTS.md` | `~/.pi/agent/AGENTS.md` | `~/.claude/CLAUDE.md` | 会话里直接说 |
| 非交互验证 | `codex exec "只回复两个字：可用"` | `pi -p "只回复两个字：可用"` | `claude -p "只回复两个字：可用"` | `opencode run --standalone "只回复两个字：可用"` |
| 升级 | `npm i -g @openai/codex` | `pi update` | `npm i -g @anthropic-ai/claude-code@latest` | `opencode upgrade` |

> **Orca 不在这张表里**：它不接模型、不存密钥，只负责启动上面这四个 CLI（所以密钥与地址的写法照抄它们各自的配置）。见 [Orca](07-orca.md) 第 4 节。

---

# 7 常见问题（先看这里）

1. **报 `401` / `Invalid token`** → 环境变量没生效。**新开一个终端**，用 `$env:NEWAPI_KEY.Length` 确认输出 `51`。见 [统一接入](02-unified-access.md) 第 2 节。
2. **报 `503 model_not_found`** → 模型名不在中转清单里。先去 [pricing 页](https://newapi.ttxs.site/pricing) 看这个模型还在不在，再对照 [统一接入](02-unified-access.md) 第 4 节。注意别把 Claude Code 的 `[1M]` 后缀抄到别的工具里。
3. **工具能聊天但不能跑命令（pi / OpenCode）** → 没装 Git for Windows。见 [前置工具](01-prerequisites.md) 第 2 节。
4. **中文乱码** → 用 Windows Terminal + PowerShell 7，不要用老的 cmd 窗口。
5. **Claude Code 在第三方模型上功能异常（图片、工具调用等）** → 记录客户端版本和模型，用同一模型在其他客户端复现，再检查配置与中转链路。见 [Claude Code](06-claude-code.md) 第 8.2、8.3 节。
6. **事实性回答不可靠** → 确认当前模型，核对资料来源，必要时换模型比较。见 [Claude Code](06-claude-code.md) 第 8.1 节。
7. **OpenCode 配置不生效** → 你装成了 v1（`opencode-ai`）。v2 的包是 `@opencode/cli`，配置格式也不同。见 [OpenCode](05-opencode.md) 第 3 节。
8. **Orca 里智能体起不来** → 先在**普通终端**里手动跑那个 CLI：手动都不行就是 CLI 自己的问题（按第 1～7 条查），手动能跑就检查 `Settings → Agents` 有没有识别到它。见 [Orca](07-orca.md) 第 8 节。
9. **Orca 新建 worktree 后没有 `node_modules` / `.env`** → 这是正常的（worktree 是干净检出）。按 [Orca](07-orca.md) 7.1 配共享目录与 `.worktreeinclude`。

---

# 8 安全与维护

- **密钥就是你的额度**：不要贴到聊天群、截图、公开仓库、公开飞书文档里。本仓库所有文档里的配置都只写变量名，所以可以安全分享。
- **配置文件可以分享，环境变量不要分享**。
- **定期核对版本**：工具更新很快（尤其 Claude Code 和 OpenCode），如果某个工具行为变了，先看它的官方文档，再回来更新本文档第 5 节的版本表。
- 想统一管理供应商、看用量、做故障转移：[cc-switch](08-cc-switch.md)。

---

# 9 参考资料

- Codex：<https://developers.openai.com/codex>
- pi：<https://pi.dev/> ｜ 源码 <https://github.com/earendil-works/pi>
- Claude Code：<https://code.claude.com/docs/zh-CN/>
- OpenCode：<https://opencode.ai/docs/zh-cn/>
- Orca：<https://www.onorca.dev/> ｜ 源码 <https://github.com/stablyai/orca>
- Matt Skills（技能包）：<https://www.aihero.dev/skills> ｜ 源码 <https://github.com/mattpocock/skills>
- cc-switch：<https://ccswitch.io/zh/>
