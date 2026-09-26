# cc-switch（进阶：供应商统一管理）

**什么时候看这篇**：当你已经把 [Codex](03-codex.md) / [pi](04-pi.md) / [OpenCode](05-opencode.md) / [Claude Code](06-claude-code.md) 用起来了，开始遇到下面这些需求时——

- 手上有**多个**模型供应商（我们的 NewAPI + 别的中转 + 官方订阅），要频繁切换；
- 想知道**每个工具到底花了多少额度**；
- 想让"主供应商挂了自动切备用"；
- 想在本机开一个**统一入口**，让所有工具都往同一个地址发请求。

**如果以上都不需要**（只用我们的 NewAPI 一家），那就**不用装它**——本文档的主线方案（环境变量 + 复制配置）已经够用，少一个工具就少一个故障点。

---

# 1 它是什么

CC Switch 是一个开源的**桌面应用**，把"AI 编程工具的供应商配置"从每个项目/终端里抽出来，集中管理：

| 能力 | 说明 |
| --- | --- |
| 供应商切换 | 一个界面管理多个工具的供应商配置，可点"启用"写回对应工具的配置文件；Claude Code 支持热切换 |
| 本地路由 | 在本机起一个可控端点，各工具统一往它发请求；内置熔断、健康监控与故障转移队列 |
| 协议转换 | 自动把不同 API 格式转换成目标工具要求的格式 |
| 用量统计 | 统一查看各供应商/各工具的消耗 |
| 扩展管理 | MCP 服务器、Skills、Prompt 模板、会话配置的集中管理 |

支持的工具有 Claude Code、Codex、OpenCode、Gemini CLI 等（具体清单以官网当前版本为准）。

![CC Switch 主界面：顶部是各工具的图标分组与本地路由开关，下方是供应商卡片（newapi，标注“需要路由”）](images/cc-switch-main.png)

---

# 2 安装

官网（Windows 版下载）：<https://ccswitch.io/zh/>

> 网上存在多个同名的"CC Switch 官网"站点，认准官网页面里的 GitHub 仓库链接与版本号；下载后建议核对一下文件名与版本。

---

# 3 基本用法（概念层面）

1. **添加供应商**：点 `+`，可以选预设供应商，也可以手动填 OpenAI 兼容接口（以下示例用于 OpenAI 协议；Claude Code 直连 Messages 的地址不带 `/v1`，本地路由则按界面要求配置）：
   - 名称：`newapi`
   - 接口地址：`https://newapi.ttxs.site/v1`
   - 密钥：`sk-你的Key`
2. **启用**：在供应商卡片上点"启用"，CC Switch 会去改写对应工具的配置文件。
3. **托盘切换**：v3.13 之后托盘菜单按应用分组，右键托盘图标就能切到某个工具的某个供应商。

**添加/编辑供应商的界面长这样**（Claude Code 得把 Sonnet / Opus / Haiku 三档映射到实际模型，这也是下面第 4.2 节那个坑的根源）：

![CC Switch 编辑供应商界面：API Key、请求地址、“需要模型映射”开关，以及 Sonnet / Opus / Haiku 三档的菜单显示名与实际请求模型](images/cc-switch-model-mapping.png)

---

# 4 ⚠️ 两个必须知道的坑

## 4.1 它会**改你的配置文件**

CC Switch 的工作原理就是替你写 `~/.claude/settings.json`、`~/.codex/config.toml` 这类文件。所以：

- **用之前先备份**：按 [统一接入第 9 节](02-unified-access.md#9-已有配置的备份与合并)备份将被改写的文件；若备份整个配置目录，其中可能有凭据与会话，应私下保管；
- 用完之后**回头检查**：我们教程里那些配置项（尤其 `ANTHROPIC_BASE_URL`、模型档位映射、`env_key`）是否还在。

## 4.2 开启"本地路由"可能会删掉 `ANTHROPIC_MODEL`

问题报告 [#6889](https://github.com/farion1231/cc-switch/issues/6889) 描述了开启路由后 `ANTHROPIC_MODEL` 被删除、请求使用 `default[1M]` 失败的情形。报告环境为 CC Switch v3.20.0、Claude Code 2.1.247、Windows 11；2026-09-26 查阅时页面仍为 Open。这不是所有版本的必然行为，本次没有本地复现；使用前仍需查看最新处理状态并记录自己的版本。

**处理办法**：开完路由后打开 `%USERPROFILE%\.claude\settings.json`，检查模型映射是否仍符合预期。下面是 `env` 内的 **JSON 片段，不能单独保存为完整配置文件**：

```text
"ANTHROPIC_MODEL": "deepseek-v4.1-flash[1M]",
"ANTHROPIC_DEFAULT_FABLE_MODEL": "glm-5.3-flash[1M]",
"ANTHROPIC_DEFAULT_OPUS_MODEL": "claude-opus-5-5[1M]",
"ANTHROPIC_DEFAULT_SONNET_MODEL": "deepseek-v4.1-flash[1M]",
"ANTHROPIC_DEFAULT_HAIKU_MODEL": "gpt-6-luna"
```

先查看路由界面的模型映射及实际请求模型；路由工具可能有意管理这些字段。确认当前版本确实需要补回时，再按 [Claude Code](06-claude-code.md) 第 4 节合并，不要与路由工具反复争写配置。改完重做该客户端第 5 节的三层验证。

---

# 5 和本文档主线方案的关系

| | 主线方案（本文档） | cc-switch |
| --- | --- | --- |
| 配置方式 | 手写配置文件 + 环境变量，一次性配好 | 图形界面点选，自动改写配置文件 |
| 供应商数量 | 一家（NewAPI） | 多家，随时切换 |
| 依赖 | 无（只有四个 CLI） | 多一个常驻桌面应用 |
| 适合谁 | 初学者、单一供应商 | 老手、多供应商、要看用量 |

**建议**：先把实际需要的客户端跑顺，有多供应商需求后再使用 CC Switch。

---

# 6 参考资料

1. CC Switch 官网：<https://ccswitch.io/zh/>
2. 供应商切换教程：<https://cc-switch.cc/tutorials/provider-switching>
3. 历史问题报告（本地路由删除 ANTHROPIC_MODEL）：<https://github.com/farion1231/cc-switch/issues/6889>
4. 本仓库主线方案：[统一接入](02-unified-access.md)
