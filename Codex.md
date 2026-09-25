# Codex

# 1 Codex应用安装

Codex 安装链接，https://apps\.microsoft\.com/detail/9plm9xgg6vks?hl=zh\-cn\&gl=US

# 2 配置

创建或编辑第一个文件。这里的experimental\_bearer\_token找朱老师要，禁止分享给任何人。

```TOML
model_provider = "newapi"
personality = "pragmatic"
sandbox_mode = "workspace-write"

[model_providers.newapi]
name = "newapi"
base_url = "https://newapi.ttxs.site/v1"
wire_api = "responses"

[windows]
sandbox = "unelevated"

[features]
js_repl = true
multi_agent = true

[tui]
status_line = ["model-with-reasoning", "context-remaining", "current-dir", "model-name", "project-root", "git-branch", "context-used"]

[sandbox_workspace_write]
network_access = true

```

创建或编辑第二个文件

```JSON
{
  "OPENAI_API_KEY": "你的序列号"
}
```

# 3 命令行安装

```PowerShell
npm i -g @openai/codex
```

# 4 VSCode 插件安装

![images\.png](图片和附件/images.png)

# 5 服务器使用



# 参考资料

1. VSCode Codex插件, https://marketplace\.visualstudio\.com/items?itemName=openai\.chatgpt

2. Codex 文档, https://developers\.openai\.com/codex

3. 入门教程，https://www\.bilibili\.com/video/BV1oJAoz2Emf/

4. Codex视频教程：https://www\.bilibili\.com/video/BV1Z9Vz61ESo

