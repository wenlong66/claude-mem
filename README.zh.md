# Claude-Mem 中文说明

Claude-Mem 是用于在编码会话之间保留上下文的持久化记忆插件。本文只保留 Claude Code 和 Codex CLI 的安装过程与使用说明。

## 安装过程

### 系统要求

- Node.js 20.0.0 或更高版本
- Bun 1.0 或更高版本（如缺失，`npx claude-mem install` 会自动安装）
- uv（如缺失会自动安装，用于 Chroma 的嵌入服务）
- Claude Code 或 Codex CLI
- SQLite 3（通过 `bun:sqlite` 内置）

### 方式一：使用 npx 安装（推荐）

```bash
npx claude-mem install
```

交互式安装器会执行以下操作：

- 检查运行时依赖，缺失时自动安装 Bun 和 uv
- 检测已安装的 Claude Code 和 Codex CLI，并让你选择要接入的工具
- 如果未检测到 Claude Code，会提示是否安装 Claude Code
- 提示选择用于压缩观察记录的 LLM Provider
- 使用 Claude Provider 时，提示选择用于压缩观察记录的 Claude 模型（Haiku / Sonnet / Opus）
- 复制插件文件到 marketplace 目录并注册插件
- 自动启动 worker 服务

安装时只选择 Claude Code 和/或 Codex CLI 即可。

### 方式二：在 Claude Code 插件市场安装

如果只需要接入 Claude Code，也可以在 Claude Code 中执行：

```bash
/plugin marketplace add thedotmack/claude-mem
/plugin install claude-mem
```

该方式会自动配置 Claude Code hooks 并启动 worker 服务。安装完成后，开启新的 Claude Code 会话，即可自动加载之前会话中的相关上下文。

> 注意：不要使用 `npm install -g claude-mem` 作为插件安装方式。该命令只安装 SDK/库，不会注册插件 hooks，也不会启动 worker 服务。请使用 `npx claude-mem install` 或 Claude Code 的 `/plugin` 命令安装。

### 方式三：本地源码安装

如果需要从本地源码安装或调试，可以先克隆仓库并构建：

```bash
git clone https://github.com/thedotmack/claude-mem.git
cd claude-mem
# Install dependencies
npm install

# Build hooks and worker service
npm run build

# Worker service will auto-start on first Claude Code session
# Or manually start with:
npm run worker:start

# Verify worker is running
npm run worker:status
```

### 安装后检查

依赖会由 `npx claude-mem install` 自动安装。之后如果外部升级导致版本标记不一致，启动时会提示运行：

```bash
npx claude-mem repair
```

Claude-Mem 的数据默认保存在：

```text
~/.claude-mem/
```

常见文件包括：

- 数据库：`~/.claude-mem/claude-mem.db`
- Worker PID：`~/.claude-mem/.worker.pid`
- Worker 端口：`~/.claude-mem/.worker.port`
- 日志：`~/.claude-mem/logs/worker-YYYY-MM-DD.log`
- 配置：`~/.claude-mem/settings.json`

## 使用说明

### Claude Code 中使用

安装并重启 Claude Code 后，Claude-Mem 会自动记录会话中的工具使用、项目观察和摘要信息。通常不需要手动操作，新会话会自动注入相关上下文。

可以直接用自然语言询问 Claude，例如：

```text
上次会话我们做了什么？
之前修复过这个问题吗？
这个文件最近改过哪些内容？
帮我找一下和认证相关的历史记录。
```

Claude 会通过 Claude-Mem 的搜索能力查找相关记忆。

### Codex CLI 中使用

使用 `npx claude-mem install` 安装时选择 Codex CLI 后，Claude-Mem 会为 Codex CLI 注册插件并启用相关 hooks。之后在 Codex CLI 会话中工作时，Claude-Mem 会记录会话上下文，并在后续会话中提供相关记忆。

如需同时接入 Claude Code 和 Codex CLI，请在安装器的工具选择步骤中同时选择这两个选项。

### 查看记忆

Claude-Mem 提供本地 Web 查看器：

```text
http://localhost:37777
```

可以在该页面查看实时记忆流、历史观察和相关上下文。

### 排除敏感内容

如果不希望某些内容被保存，可使用 `<private>` 标签包裹：

```text
<private>
这里是敏感信息，不应写入记忆。
</private>
```

### 中文模式

如需让 Claude-Mem 生成中文观察，可编辑配置文件：

```text
~/.claude-mem/settings.json
```

设置：

```json
{
  "CLAUDE_MEM_MODE": "code--zh"
}
```

修改后重启 Claude Code 或重新打开 Codex CLI 会话生效。
