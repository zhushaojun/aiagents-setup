# Claude Code

# 1 Visual Studio Code

## 1\.1 安装扩展

注意安装2\.1\.153版本，并取消自动更新

![image\.png](图片和附件/image.png)



## 1\.2 配置

打开用户目录 %USERPROFILE%， 创建 `.claude` 目录。在其中创建3个文件

```JSON
{
  "env": {
    "ANTHROPIC_AUTH_TOKEN": "sk-api-key这里填写你的apikey",
    "ANTHROPIC_BASE_URL": "https://newapi.ttxs.site",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "kimi-k2.6",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "deepseek-v4-flash[1m]",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "deepseek-v4-pro[1m]",
    "ENABLE_TOOL_SEARCH": "true",
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1",
    "CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC": "1",
    "CLAUDE_CODE_DISABLE_NONSTREAMING_FALLBACK": "1"
  },
  "language": "简体中文",
  "alwaysThinkingEnabled": true,
  "effortLevel": "high",
  "autoUpdatesChannel": "stable",
  "skipAutoPermissionPrompt": true,
  "defaultPermissionMode": "auto"
}
```

```JSON
{
  "primaryApiKey": "any"
}
```

```Markdown
# 提示

* 用中文
* 复杂任务先做Plan
* 优先使用Agent高效率
* 当前系统是win11
* 当工具调用涉及网络搜索或mcp时，用haiku agent执行
```

# 2 服务器上通过Visual Studio Code使用

## 2\.1 服务器中安装扩展

首先确保本地使用没问题，然后vscode连接远程服务器，然后点击在远程服务器中安装claude扩展

![image\.png](图片和附件/image%203.png)

## 2\.2 服务器中配置

在命令行中执行以下代码，注意替换序列号

```Bash
mkdir ~/.claude

cat > ~/.claude/settings.json << EOF
{
  "env": {
    "ANTHROPIC_AUTH_TOKEN": "sk-api-key这里填写你的apikey",
    "ANTHROPIC_BASE_URL": "https://newapi.ttxs.site",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "kimi-k2.6",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "deepseek-v4-flash[1m]",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "deepseek-v4-pro[1m]",
    "ENABLE_TOOL_SEARCH": "true",
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1",
    "CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC": "1",
    "CLAUDE_CODE_DISABLE_NONSTREAMING_FALLBACK": "1"
  },
  "language": "简体中文",
  "alwaysThinkingEnabled": true,
  "effortLevel": "high",
  "autoUpdatesChannel": "stable",
  "skipAutoPermissionPrompt": true,
  "defaultPermissionMode": "auto"
}
EOF 

cat > ~/.claude/config.json << EOF
{
  "primaryApiKey": "any"
}
EOF 

cat > ~/.claude/CLAUDE.md << EOF
# 提示

* 用中文
* 复杂任务先做Plan
* 优先使用Agent高效率
* 当工具调用涉及网络搜索或mcp时，用haiku agent执行
EOF 
```

# 3 命令行安装使用

```Bash
npm install -g @anthropic-ai/claude-code@2.1.153
claude
```

进入claude以后输 /model 应该能看到以下模型。

![image\.png](图片和附件/image%201.png)



# 4 Claude for Windows

参考这篇文章安装配置：https://zhuanlan\.zhihu\.com/p/2032968840011834298

Base URL：https://newapi\.ttxs\.site          apikey填写上面那个一样的

使用cc\-switch模型设置参考：



![image\.png](图片和附件/image%202.png)



![image\.png](图片和附件/image%204.png)



