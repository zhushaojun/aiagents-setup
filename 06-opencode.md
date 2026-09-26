# OpenCode

**辅助工具。** 开源、通用的终端 AI 编程智能体：几乎什么模型都能接，终端 / IDE / 桌面端都有入口。它已经发展成为一个能真正干活的独立产品（v2），**虽然能力上限不如 Codex 和 pi（它不是由自家最强模型驱动的），但完全可用，值得动手试一试。**

<!-- TODO 截图：OpenCode v2 的终端界面（TUI） -->

---

# 1 它是什么、适合谁

- **定位**：社区驱动的开源 AI 编程智能体。它本身**不绑定模型**，用"供应商"的方式接任意模型（我们接的是 NewAPI 中转）。
- **适合**：想用完全开源的工具、想一套配置到处跑（终端 / IDE / 桌面）、想接各种国产模型对比效果的人。
- **优势**：开源可审计；供应商配置灵活；界面现代化，默认信息密度高。
- **需要知道的两件事**：
  1. **它能力不强，主要原因是模型**：它不像 Codex/Claude Code 那样由自家最强模型驱动，所以同样的任务，出来的结果通常不如前两者稳定；
  2. **它是 v2 了**，配置格式与网上大量 v1 教程**不兼容**（`provider` 变 `providers`、`npm` 变 `package`、`options` 变 `settings`）。本篇按 **v2** 写。

---

# 2 前置要求

| 项目 | 要求 | 检查方法 |
| --- | --- | --- |
| Node.js | ≥ 20（v2 通过 npm 安装） | `node -v` |
| 环境变量 | `NEWAPI_KEY` 已设置 | `$env:NEWAPI_KEY.Length` → `51` |
| Git for Windows | 建议装（模型要执行命令时会用到） | `bash --version` |

> 密钥设置见 **[统一接入](02-unified-access.md)** 第 2 节。若模型执行命令时报找不到 bash，装 Git for Windows，或用环境变量 `OPENCODE_GIT_BASH_PATH` 指向 `bash.exe`。

---

# 3 安装

## 3.1 Windows（npm 安装 v2，推荐）

```PowerShell
npm install -g @opencode/cli
opencode --version
```

预期输出：`2.0.16` 这类 v2 版本号。

> ⚠️ **别装错包**：v2 的 npm 包是 **`@opencode/cli`**；网上老教程里的 `opencode-ai` 是 **v1** 旧线（版本还是 1.18.x），两者配置格式完全不同。教程只会用 `@opencode/cli`。

## 3.2 官方脚本（可选）

官方一键脚本会自动装最新版：

```Bash
curl -fsSL https://opencode.ai/install | bash
```

## 3.3 远程 Linux 服务器

```Bash
npm install -g @opencode/cli
mkdir -p ~/.config/opencode
# 配置文件写法见第 4 节，把 opencode.json 放到 ~/.config/opencode/
```

再把 `export NEWAPI_KEY="sk-你的Key"` 写进 `~/.bashrc` 并 `source ~/.bashrc`。

---

# 4 配置（复制即用）

配置文件位置：`%USERPROFILE%\.config\opencode\opencode.json`（Linux 上是 `~/.config/opencode/opencode.json`）。

```PowerShell
New-Item -ItemType Directory -Force "$env:USERPROFILE\.config\opencode" | Out-Null
@'
{
  "$schema": "https://opencode.ai/config.json",
  "providers": {
    "newapi": {
      "package": "aisdk:@ai-sdk/openai-compatible",
      "settings": {
        "apiKey": "{env:NEWAPI_KEY}",
        "baseURL": "https://newapi.ttxs.site/v1"
      },
      "models": {
        "glm-5.3-flash": {
          "name": "GLM-5.3 Flash",
          "limit": { "context": 500000, "output": 128000 },
          "capabilities": { "tools": true, "input": ["text"], "output": ["text"] }
        },
        "deepseek-v4.1-flash": {
          "name": "DeepSeek V4.1 Flash",
          "limit": { "context": 500000, "output": 384000 },
          "capabilities": { "tools": true, "input": ["text"], "output": ["text"] }
        },
        "gpt-6-sol": {
          "name": "GPT-6 Sol",
          "limit": { "context": 258000, "output": 32768 },
          "capabilities": { "tools": true, "input": ["text", "image"], "output": ["text"] }
        }
      }
    }
  }
}
'@ | Set-Content -Encoding utf8 "$env:USERPROFILE\.config\opencode\opencode.json"
```

**v2 格式的四个要点**（对照老教程看容易踩坑）：

| 字段 | v2 写法（本篇） | 老教程（v1，无效） |
| --- | --- | --- |
| 供应商容器 | `"providers"`（复数） | `"provider"` |
| 驱动包 | `"package": "aisdk:@ai-sdk/openai-compatible"` | `"npm": "@ai-sdk/openai-compatible"` |
| 连接与密钥 | `"settings": { "apiKey": "{env:NEWAPI_KEY}", "baseURL": "..." }` | `"options": { ... }` |
| 密钥插值 | `{env:变量名}` | 同名，但写在 `options` 里 |

`models` 里的 `limit.context` / `limit.output` 是给模型声明的上下文与输出上限，**写小了会让它无法处理长文件**，照抄上面的值即可（数值来自中转的模型信息）。

---

# 5 验证

用非交互模式直接验证密钥链路：

```PowerShell
opencode run --standalone --model newapi/gpt-6-sol "只回复两个字：可用"
```

预期输出里出现 `可用`。

如果报 `Invalid token`，说明 `{env:NEWAPI_KEY}` 没解析出来（环境变量没生效，或你把它写成了 `$NEWAPI_KEY`）。

再进交互界面干活：

```PowerShell
cd D:\codes\some-project
opencode
```

界面里可以用 `/models` 切换模型（应能看到 `newapi/glm-5.3-flash` 等）。

<!-- TODO 截图：OpenCode 里模型选择列表（能看到 newapi 下的模型） -->

---

# 6 日常用法

| 你想做的事 | 怎么做 |
| --- | --- |
| 进入交互界面 | 在项目目录运行 `opencode` |
| 一次性执行一条指令 | `opencode run "把 xxx 改成 yyy"` |
| 指定模型 | `opencode run --model newapi/glm-5.3-flash "..."`（`provider/model`，可用 `#` 加档位） |
| 继续上次会话 | `opencode run -c "..."` 或界面里继续 |
| 让它读某个文件 | `opencode run --file 路径 "解释这个文件"` |
| 自动批准权限 | 加 `--auto`（放心再开） |
| 看会话用量 | `opencode stats` |
| 管理供应商凭据 | `opencode auth` |

---

# 7 进阶

- **插件**：`opencode plugin` 管理插件；配置里也可以直接列插件包名。
- **多智能体编排**：社区有 `oh-my-openagent` 这类插件，把不同任务分给不同模型。**本篇不展开、也不推荐初学者上**——它需要先熟悉基础用法，而且插件里的模型名要自己跟中转清单对齐，很容易写出失效配置。
- **Web / 服务模式**：`opencode serve` 起一个本地服务，`opencode serve` + 浏览器可当轻量 Web 版用；`opencode acp` 供 IDE 接入 Agent Client Protocol。
- **桌面端与 IDE 扩展**：OpenCode 有桌面应用与编辑器扩展，适合不喜欢终端的人（本篇只讲终端，其余入口按需自选）。

---

# 8 常见问题

| 现象 | 原因 / 解决 |
| --- | --- |
| 装了但不认识配置 | 装成了 v1：`npm uninstall -g opencode-ai`，改 `npm install -g @opencode/cli` |
| `Invalid token` | `{env:NEWAPI_KEY}` 没解析：检查环境变量是否新开终端生效；写法必须是 `{env:NEWAPI_KEY}` |
| 模型不在列表里 | `providers`（复数）写成了 `provider`，或模型 `id` 不在中转清单里（见 [统一接入](02-unified-access.md) 第 4 节） |
| 模型执行命令报找不到 bash | 装 Git for Windows；或用 `OPENCODE_GIT_BASH_PATH` 指向 `bash.exe` |
| 后台服务起不来 | 加 `--standalone` 用私有服务跑：`opencode run --standalone "..."` |
| `model_not_found` | 模型名写错，或该模型当期已下架（看 [pricing 页](https://newapi.ttxs.site/pricing)）；注意别把 Claude Code 的 `[1M]` 后缀抄进来 |

---

# 9 升级与卸载

```PowerShell
opencode upgrade                          # 就地升级（官方推荐）
npm install -g @opencode/cli@latest       # 或用 npm 升级
npm uninstall -g @opencode/cli            # 卸载
opencode uninstall                        # 官方卸载命令（会清理相关文件）
```

配置在 `%USERPROFILE%\.config\opencode`，卸载重装不丢（但用 `opencode uninstall` 时会提示是否清理）。

---

# 10 参考资料

1. 官方文档：<https://opencode.ai/docs/zh-cn/>
2. 下载页（终端 / 桌面 / IDE 扩展）：<https://opencode.ai/zh/download>
3. 官网：<https://opencode.ai/zh>
4. 统一接入与模型清单：见本仓库 [统一接入](02-unified-access.md)
5. **实时查看可用模型与价格**：<https://newapi.ttxs.site/pricing>