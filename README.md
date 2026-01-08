# Claude Code 项目配置示例

> 现在大多数软件工程师都严重低估了大型语言模型代理的实力，尤其是 Claude Code 这样的工具。

一旦你配置好 Claude Code，就可以把它指向你的代码库，让它学习团队约定、吸收最佳实践，并持续打磨，直到它像一位超强队友一样工作。**真正的突破在于构建一套稳固的可复用「技能」([skills](#skills---domain-knowledge))，再配上少量专用的「代理」([agents](#agents---specialized-assistants))，用于你最常执行的流程。**

### 实际运作示例

**自定义 UI 库？** 我们准备了一份[技能文档](.claude/skills/core-components/SKILL.md)详细讲解如何使用它。写测试有[技能](.claude/skills/testing-patterns/SKILL.md)，设计 GraphQL 也有[技能](.claude/skills/graphql-schema/SKILL.md)，几乎所有事情都有说明。因此 Claude 生成的代码会天然符合我们的模式和标准。

**自动化质量闸？** 我们通过[钩子](.claude/settings.json)在提交前自动格式化代码、当测试文件变化时运行测试、TypeScript 类型检查，甚至[阻止在 main 分支直接修改](.claude/settings.md)。Claude Code 还生成了多套 ESLint 自动化脚本，自定义规则和 lint 检查可以在代码评审前抓出问题。

**深度代码审查？** 我们有一个[代码审查代理](.claude/agents/code-reviewer.md)，在修改完成后让 Claude 运行。它有一份详尽的检查清单，覆盖 TypeScript 严格模式、错误处理、加载状态、mutation 模式等等。当提交 PR 时，还有一个 [GitHub Action](.github/workflows/pr-claude-code-review.yml) 自动完成整套审查。

**定期维护？** GitHub 工作流代理按计划运行：
- [每月文档同步](.github/workflows/scheduled-claude-code-docs-sync.yml)：读取过去一个月的提交，确认文档仍然准确
- [每周代码质量扫描](.github/workflows/scheduled-claude-code-quality.yml)：随机抽取目录检查并自动修复
- [双周依赖审计](.github/workflows/scheduled-claude-code-dependency-audit.yml)：安全更新依赖并验证测试

**智能技能建议？** 我们构建了一个[技能评估系统](#skill-evaluation-hooks)，分析每一次提示，自动建议应该激活哪些技能，依据关键词、路径以及意图模式。

大量维护和质量工作都可以自动完成，运行得出奇顺畅。

**JIRA/Linear 集成？** 我们通过 [MCP 服务器](.mcp.json)把 Claude Code 连接到工单系统。Claude 可以阅读工单、理解需求、实现功能、更新状态，甚至在发现缺陷时创建新工单。[`/ticket` 命令](.claude/commands/ticket.md)负责整个流程——读取验收标准、实施任务、将 PR 关联回工单。

我们甚至让 Claude Code 负责工单分诊。它会阅读工单、深入代码，并留下建议实现方案的备注。工程师接手时，基本已经完成了一半。

**这里有太多唾手可得的机会，让人难以相信大家还没有广泛采用。**

---

## 目录

- [目录结构](#directory-structure)
- [快速开始](#quick-start)
- [配置参考](#configuration-reference)
  - [CLAUDE.md - 项目记忆](#claudemd---project-memory)
  - [settings.json - 钩子与环境](#settingsjson---hooks--environment)
  - [MCP Servers - 外部集成](#mcp-servers---external-integrations)
  - [LSP Servers - 实时代码智能](#lsp-servers---real-time-code-intelligence)
  - [技能评估钩子](#skill-evaluation-hooks)
  - [Skills - 领域知识](#skills---domain-knowledge)
  - [Agents - 专用助理](#agents---specialized-assistants)
  - [Commands - 斜杠命令](#commands---slash-commands)
- [GitHub Actions 工作流](#github-actions-workflows)
- [最佳实践](#best-practices)
- [仓库中的示例](#examples-in-this-repository)

---

<a id="directory-structure"></a>
## 目录结构

```
your-project/
├── CLAUDE.md                      # 项目记忆（备用路径）
├── .mcp.json                      # MCP 服务器配置（JIRA、GitHub 等）
├── .claude/
│   ├── settings.json              # 钩子、运行环境、权限
│   ├── settings.local.json        # 个人覆盖（已加入 .gitignore）
│   ├── settings.md                # 钩子的人类可读说明
│   ├── .gitignore                 # 忽略本地/个人文件
│   │
│   ├── agents/                    # 自定义 AI 代理
│   │   └── code-reviewer.md       # 主动代码审查代理
│   │
│   ├── commands/                  # 斜杠命令 (/command-name)
│   │   ├── onboard.md             # 深度任务探索
│   │   ├── pr-review.md           # PR 审查流程
│   │   └── ...
│   │
│   ├── hooks/                     # 钩子脚本
│   │   ├── skill-eval.sh          # 提交提示时的技能匹配
│   │   ├── skill-eval.js          # Node.js 技能匹配引擎
│   │   └── skill-rules.json       # 模式匹配配置
│   │
│   ├── skills/                    # 领域知识文档
│   │   ├── README.md              # 技能概览
│   │   ├── testing-patterns/
│   │   │   └── SKILL.md
│   │   ├── graphql-schema/
│   │   │   └── SKILL.md
│   │   └── ...
│   │
│   └── rules/                     # 可选的模块化指令
│       ├── code-style.md
│       └── security.md
│
└── .github/
    └── workflows/
        ├── pr-claude-code-review.yml           # 自动 PR 审查
        ├── scheduled-claude-code-docs-sync.yml # 每月文档同步
        ├── scheduled-claude-code-quality.yml   # 每周质量审查
        └── scheduled-claude-code-dependency-audit.yml
```

---

<a id="quick-start"></a>
## 快速开始

### 1. 创建 `.claude` 目录

```bash
mkdir -p .claude/{agents,commands,hooks,skills}
```

### 2. 添加 CLAUDE.md

在项目根目录创建 `CLAUDE.md`，写入项目关键信息。完整示例见 [CLAUDE.md](CLAUDE.md)。

```markdown
# 项目名称

## 关键信息
- **技术栈**：React、TypeScript、Node.js
- **测试命令**：`npm run test`
- **Lint 命令**：`npm run lint`

## 关键目录
- `src/components/` - React 组件
- `src/api/` - API 层
- `tests/` - 测试文件

## 代码风格
- TypeScript 严格模式
- 优先使用 interface 而非 type
- 禁止 `any`——使用 `unknown`
```

### 3. 使用钩子添加 settings.json

创建 `.claude/settings.json`。完整示例参见 [settings.json](.claude/settings.json)，其中包含自动格式化、测试等配置。

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "[ \"$(git branch --show-current)\" != \"main\" ] || { echo '{\"block\": true, \"message\": \"Cannot edit on main branch\"}' >&2; exit 2; }",
            "timeout": 5
          }
        ]
      }
    ]
  }
}
```

### 4. 添加第一份技能

创建 `.claude/skills/testing-patterns/SKILL.md`。详见 [testing-patterns/SKILL.md](.claude/skills/testing-patterns/SKILL.md)。

```markdown
---
name: testing-patterns
description: 项目中的 Jest 测试范式。当编写测试、创建 mock 或进行 TDD 时使用。
---

# 测试模式

## 测试结构
- 使用 `describe` 进行分组
- 使用 `it` 描述单个测试
- 遵循 AAA：Arrange、Act、Assert

## Mock
- 使用工厂函数：`getMockUser(overrides)`
- mock 外部依赖，而不是内部模块
```

> **提示：** `description` 字段非常关键——Claude 依靠它决定何时激活技能，请写入用户可能提到的关键词。

---

<a id="configuration-reference"></a>
## 配置参考

<a id="claudemd---project-memory"></a>
### CLAUDE.md - 项目记忆

CLAUDE.md 是 Claude 的持久记忆，会在会话启动时自动加载。

**优先级（由高到低）：**
1. `.claude/CLAUDE.md`（项目内 .claude 文件夹）
2. `./CLAUDE.md`（项目根目录）
3. `~/.claude/CLAUDE.md`（用户级，适用于所有项目）

**建议内容：**
- 项目技术栈与架构概览
- 关键命令（测试、构建、部署等）
- 代码风格指南
- 重要目录与用途
- 必须遵守的规则与约束

**📄 示例：** [CLAUDE.md](CLAUDE.md)

---

<a id="settingsjson---hooks--environment"></a>
### settings.json - 钩子与环境

`settings.json` 是钩子、环境变量与权限的总配置。

**位置：** `.claude/settings.json`

**📄 示例：** [settings.json](.claude/settings.json) ｜ [人类可读说明](.claude/settings.md)

#### 钩子事件

| 事件 | 触发时机 | 用途 |
|------|----------|------|
| `PreToolUse` | 工具执行前 | 阻止在 main 分支编辑、校验命令 |
| `PostToolUse` | 工具执行后 | 自动格式化、运行测试、lint |
| `UserPromptSubmit` | 用户提交提示时 | 添加上下文、建议技能 |
| `Stop` | 代理结束时 | 决定是否继续 |

#### 钩子返回格式

```json
{
  "block": true,          // 是否阻止行为（仅 PreToolUse）
  "message": "Reason",   // 展示给用户的原因
  "feedback": "Info",    // 非阻塞反馈
  "suppressOutput": true, // 隐藏命令输出
  "continue": false       // 是否继续执行
}
```

#### 退出码
- `0` - 成功
- `2` - 阻塞错误（仅 PreToolUse，直接阻止）
- 其他 - 非阻塞错误

---

<a id="mcp-servers---external-integrations"></a>
### MCP Servers - 外部集成

MCP（Model Context Protocol）服务器让 Claude Code 连接到 JIRA、GitHub、Slack、数据库等外部工具，实现“读取工单 → 实现功能 → 更新状态”等流程。

**位置：** `.mcp.json`（项目根目录，纳入版本控制）

**📄 示例：** [.mcp.json](.mcp.json)

#### MCP 工作方式

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   Claude Code   │────▶│     MCP 服务器  │────▶│   外部 API       │
│                 │◀────│   (本地桥接进程)│◀────│ (JIRA、GitHub 等)│
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

配置好 JIRA 的 MCP 服务器后，Claude 会获得 `jira_get_issue`、`jira_update_issue`、`jira_create_issue` 等工具。

#### .mcp.json 格式

```json
{
  "mcpServers": {
    "server-name": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-server-name"],
      "env": {
        "API_KEY": "${API_KEY}"
      }
    }
  }
}
```

**字段说明：**

| 字段 | 必填 | 说明 |
|------|------|------|
| `type` | 是 | 服务器类型：`stdio`（本地进程）或 `http`（远程） |
| `command` | stdio 必填 | 可执行文件（如 `npx`、`python`） |
| `args` | 否 | 命令行参数 |
| `env` | 否 | 环境变量（支持 `${VAR}` 展开） |
| `url` | http 必填 | 远程服务器 URL |
| `headers` | http 可选 | 认证所需 HTTP 头 |

#### 示例：JIRA 集成

```json
{
  "mcpServers": {
    "jira": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-jira"],
      "env": {
        "JIRA_HOST": "${JIRA_HOST}",
        "JIRA_EMAIL": "${JIRA_EMAIL}",
        "JIRA_API_TOKEN": "${JIRA_API_TOKEN}"
      }
    }
  }
}
```

**可实现的能力：**
- 读取工单详情、验收标准和评论
- 更新工单状态（To Do/待办 → In Progress/进行中 → In Review/审核中）
- 追加进度评论
- 创建新工单处理发现的缺陷
- 在 PR 中自动关联工单

**结合 [`/ticket` 命令](.claude/commands/ticket.md) 的工作流示例：**
```
你：/ticket PROJ-123

Claude：
1. 正在获取 PROJ-123...
   “Add user profile avatar upload（添加用户头像上传）”

2. 阅读验收标准...
   - Upload button on profile page（在个人资料页提供上传按钮）
   - Support JPG/PNG up to 5MB（支持不超过 5MB 的 JPG/PNG）
   - Show loading state（展示加载状态）

3. 搜索相关代码...
   找到：src/screens/Profile/ProfileScreen.tsx

4. 创建分支：cw/PROJ-123-avatar-upload

5. [实现功能...]

6. 将 JIRA 状态更新为 “In Review（审核中）”
   评论：“PR #456 ready for review（PR #456 已准备好评审）”

7. 创建并关联 PR...
```

#### 常见 MCP 配置

**工单系统：**
```json
{
  "jira": {
    "type": "stdio",
    "command": "npx",
    "args": ["-y", "@anthropic/mcp-jira"],
    "env": {
      "JIRA_HOST": "${JIRA_HOST}",
      "JIRA_EMAIL": "${JIRA_EMAIL}",
      "JIRA_API_TOKEN": "${JIRA_API_TOKEN}"
    }
  },
  "linear": {
    "type": "stdio",
    "command": "npx",
    "args": ["-y", "@anthropic/mcp-linear"],
    "env": { "LINEAR_API_KEY": "${LINEAR_API_KEY}" }
  }
}
```

**代码与 DevOps：**
```json
{
  "github": {
    "type": "stdio",
    "command": "npx",
    "args": ["-y", "@anthropic/mcp-github"],
    "env": { "GITHUB_TOKEN": "${GITHUB_TOKEN}" }
  },
  "sentry": {
    "type": "stdio",
    "command": "npx",
    "args": ["-y", "@anthropic/mcp-sentry"],
    "env": {
      "SENTRY_AUTH_TOKEN": "${SENTRY_AUTH_TOKEN}",
      "SENTRY_ORG": "${SENTRY_ORG}"
    }
  }
}
```

**通信：**
```json
{
  "slack": {
    "type": "stdio",
    "command": "npx",
    "args": ["-y", "@anthropic/mcp-slack"],
    "env": {
      "SLACK_BOT_TOKEN": "${SLACK_BOT_TOKEN}",
      "SLACK_TEAM_ID": "${SLACK_TEAM_ID}"
    }
  }
}
```

**数据库：**
```json
{
  "postgres": {
    "type": "stdio",
    "command": "npx",
    "args": ["-y", "@anthropic/mcp-postgres"],
    "env": { "DATABASE_URL": "${DATABASE_URL}" }
  }
}
```

#### 环境变量

MCP 支持变量展开：
- `${VAR}` - 若环境变量存在则展开，否则报错
- `${VAR:-default}` - 若未设置则使用默认值

在 shell 配置或 `.env` 中设置（切勿提交敏感信息）：
```bash
export JIRA_HOST="https://yourcompany.atlassian.net"
export JIRA_EMAIL="you@company.com"
export JIRA_API_TOKEN="your-api-token"
```

#### MCP 设置项

在 `settings.json` 中开启自动批准：

```json
{
  "enableAllProjectMcpServers": true
}
```

或只允许特定服务器：

```json
{
  "enabledMcpjsonServers": ["jira", "github", "slack"]
}
```

---

<a id="lsp-servers---real-time-code-intelligence"></a>
### LSP Servers - 实时代码智能

LSP（Language Server Protocol）让 Claude 获得实时的类型信息、错误提示、补全与导航能力。编辑 TypeScript 时，Claude 能立即看到类型错误；引用函数时可以跳转定义，大幅提升代码生成质量。

#### 启用 LSP

在 `settings.json` 的 `enabledPlugins` 中加入插件：

```json
{
  "enabledPlugins": {
    "typescript-lsp@claude-plugins-official": true,
    "pyright-lsp@claude-plugins-official": true
  }
}
```

#### LSP 能提供什么

| 功能 | 说明 |
|------|------|
| **Diagnostics** | 每次编辑后实时返回错误和警告 |
| **Type Information** | 悬停信息、函数签名、类型定义 |
| **Code Navigation** | 跳转定义、查找引用 |
| **Completions** | 上下文相关的符号补全 |

#### 可用 LSP 插件

| 插件 | 语言 | 需先安装 |
|------|------|----------|
| `typescript-lsp` | TypeScript/JavaScript | `npm install -g typescript-language-server typescript` |
| `pyright-lsp` | Python | `pip install pyright` |
| `rust-lsp` | Rust | `rustup component add rust-analyzer` |

#### 自定义 LSP 配置

在 `.lsp.json` 中提供高级配置：

```json
{
  "typescript": {
    "command": "typescript-language-server",
    "args": ["--stdio"],
    "extensionToLanguage": {
      ".ts": "typescript",
      ".tsx": "typescriptreact"
    },
    "initializationOptions": {
      "preferences": {
        "quotePreference": "single"
      }
    }
  }
}
```

#### 故障排查

若 LSP 无法工作：

1. **确认二进制已安装：**
   ```bash
   which typescript-language-server
   ```
2. **启用调试日志：**
   ```bash
   claude --enable-lsp-logging
   ```
3. **查看插件状态：**
   ```bash
   claude /plugin  # 检查 Errors 标签
   ```

---

<a id="skill-evaluation-hooks"></a>
### 技能评估钩子

我们最强大的自动化之一是**技能评估系统**。它在每次提交提示时运行，智能建议需要激活的技能。

**📄 文件：** [skill-eval.sh](.claude/hooks/skill-eval.sh) ｜ [skill-eval.js](.claude/hooks/skill-eval.js) ｜ [skill-rules.json](.claude/hooks/skill-rules.json)

#### 工作流程

1. **提示分析** —— 引擎会检查：
   - **关键词**：简单词匹配（`test`、`form`、`graphql` 等）
   - **正则模式**：如 `\btest(?:s|ing)?\b`
   - **文件路径**：提取 `src/components/Button.tsx` 之类
   - **意图**：识别“create test”“fix bug”等意图
2. **目录映射** —— 将路径映射到技能：
   ```json
   {
     "src/components/core": "core-components",
     "src/graphql": "graphql-schema",
     ".github/workflows": "github-actions",
     "src/hooks": "react-ui-patterns"
   }
   ```
3. **置信度评分** —— 每种触发有分值：
   ```json
   {
     "keyword": 2,
     "keywordPattern": 3,
     "pathPattern": 4,
     "directoryMatch": 5,
     "intentPattern": 4
   }
   ```
4. **技能建议** —— 超过阈值的技能会附理由列出。

#### 配置

技能在 [skill-rules.json](.claude/hooks/skill-rules.json) 中定义：

```json
{
  "testing-patterns": {
    "description": "Jest 测试范式与 TDD 工作流",
    "priority": 9,
    "triggers": {
      "keywords": ["test", "jest", "spec", "tdd", "mock"],
      "keywordPatterns": ["\\btest(?:s|ing)?\\b", "\\bspec\\b"],
      "pathPatterns": ["**/*.test.ts", "**/*.test.tsx"],
      "intentPatterns": [
        "(?:write|add|create|fix).*(?:test|spec)",
        "(?:test|spec).*(?:for|of|the)"
      ]
    },
    "excludePatterns": ["e2e", "maestro", "end-to-end"]
  }
}
```

#### 添加到自己的项目

1. 复制钩子脚本：
   ```bash
   cp -r .claude/hooks/ your-project/.claude/hooks/
   ```
2. 在 `settings.json` 中注册钩子：
   ```json
   {
     "hooks": {
       "UserPromptSubmit": [
         {
           "hooks": [
             {
               "type": "command",
               "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/skill-eval.sh",
               "timeout": 5
             }
           ]
         }
       ]
     }
   }
   ```
3. 根据项目实际，自定义 [skill-rules.json](.claude/hooks/skill-rules.json)。

---

<a id="skills---domain-knowledge"></a>
### Skills - 领域知识

技能是教给 Claude 项目特定模式的 Markdown 文档。

**位置：** `.claude/skills/{skill-name}/SKILL.md`

**📄 示例：**
- [testing-patterns](.claude/skills/testing-patterns/SKILL.md)：TDD、工厂函数、mock
- [systematic-debugging](.claude/skills/systematic-debugging/SKILL.md)：四步调试法
- [react-ui-patterns](.claude/skills/react-ui-patterns/SKILL.md)：加载/错误/空状态
- [graphql-schema](.claude/skills/graphql-schema/SKILL.md)：查询、mutation、codegen
- [core-components](.claude/skills/core-components/SKILL.md)：设计系统与 tokens
- [formik-patterns](.claude/skills/formik-patterns/SKILL.md)：表单和校验

#### SKILL.md 前言字段

| 字段 | 必填 | 最大长度 | 说明 |
|------|------|----------|------|
| `name` | **是** | 64 | 小写加连字符，需与目录一致 |
| `description` | **是** | 1024 | 描述用途与触发场景，Claude 通过它匹配技能 |
| `allowed-tools` | 否 | - | 允许使用的工具列表，例如 `Read, Grep, Bash(npm:*)` |
| `model` | 否 | - | 指定模型，例如 `claude-sonnet-4-20250514` |

#### SKILL.md 模板

```markdown
---
name: skill-name
description: 功能与使用时机。
allowed-tools: Read, Grep, Glob
model: claude-sonnet-4-20250514
---

# 技能标题

## 适用场景
- 触发条件 1
- 触发条件 2

## 核心模式

### 模式名称
```typescript
// 示例代码
```

## 反模式

### 不要这样做
```typescript
// 错误示例
```

## 集成
- 相关技能：`other-skill`
```

#### 最佳实践

1. **保持聚焦** —— 控制在 500 行以内，详细内容另拆文件
2. **描述要富含触发词** —— Claude 依赖 description 做语义匹配
3. **包含示例** —— 提供好坏示例
4. **互相引用** —— 展示技能如何协作
5. **确保文件名为 `SKILL.md`** —— 区分大小写

---

<a id="agents---specialized-assistants"></a>
### Agents - 专用助理

代理是拥有独立提示词的 AI 助手。

**位置：** `.claude/agents/{agent-name}.md`

**📄 示例：**
- [code-reviewer.md](.claude/agents/code-reviewer.md)：全面代码审查清单
- [github-workflow.md](.claude/agents/github-workflow.md)：Git 工作流代理

#### 代理格式

```markdown
---
name: code-reviewer
description: 检查代码质量、安全性与规范。写完代码后使用。
model: opus
---

# Agent System Prompt

You are a senior code reviewer...

## 流程
1. 运行 `git diff`
2. 套用审查清单
3. 提供反馈

## 检查清单
- [ ] 禁止 TypeScript `any`
- [ ] 必须有错误处理
- [ ] 包含测试
```

#### 配置字段

| 字段 | 必填 | 说明 |
|------|------|------|
| `name` | 是 | 小写加连字符 |
| `description` | 是 | 1024 字内，描述何时使用 |
| `model` | 否 | `sonnet`、`opus` 或 `haiku` |
| `tools` | 否 | 允许的工具列表 |

---

<a id="commands---slash-commands"></a>
### Commands - 斜杠命令

自定义命令以 `/command-name` 形式调用。

**位置：** `.claude/commands/{command-name}.md`

**📄 示例：**
- [onboard.md](.claude/commands/onboard.md)：深度任务探索
- [pr-review.md](.claude/commands/pr-review.md)：PR 审查流程
- [pr-summary.md](.claude/commands/pr-summary.md)：生成 PR 描述
- [code-quality.md](.claude/commands/code-quality.md)：质量检查
- [docs-sync.md](.claude/commands/docs-sync.md)：文档同步
- [ticket.md](.claude/commands/ticket.md)：工单全流程

#### 命令格式

```markdown
---
description: 命令列表中的简短描述
allowed-tools: Bash(git:*), Read, Grep
---

# 命令说明

你的任务：$ARGUMENTS

## 步骤
1. 先做这个
2. 然后做那个
```

#### 变量

- `$ARGUMENTS` —— 传入的所有参数
- `$1`, `$2`, `$3` —— 单个位置参数

#### 内联 Bash

```markdown
当前分支：!`git branch --show-current`
最近提交：!`git log --oneline -5`
```

---

<a id="github-actions-workflows"></a>
## GitHub Actions 工作流

通过 GitHub Actions 自动化代码审查、质量检查与维护。

**📄 示例：**
- [pr-claude-code-review.yml](.github/workflows/pr-claude-code-review.yml)：自动 PR 审查
- [scheduled-claude-code-docs-sync.yml](.github/workflows/scheduled-claude-code-docs-sync.yml)：每月文档同步
- [scheduled-claude-code-quality.yml](.github/workflows/scheduled-claude-code-quality.yml)：每周质量审查
- [scheduled-claude-code-dependency-audit.yml](.github/workflows/scheduled-claude-code-dependency-audit.yml)：双周依赖更新

### PR 代码审查

自动审查 PR 并响应 `@claude` 提及。

```yaml
name: PR - Claude Code Review
on:
  pull_request:
    types: [opened, synchronize, reopened]
  issue_comment:
    types: [created]

jobs:
  review:
    if: |
      github.event_name == 'pull_request' ||
      (github.event_name == 'issue_comment' &&
       github.event.issue.pull_request &&
       contains(github.event.comment.body, '@claude'))
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: anthropics/claude-code-action@beta
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          model: claude-opus-4-5-20251101
          prompt: |
            Review this PR using .claude/agents/code-reviewer.md standards.
            Run `git diff origin/main...HEAD` to see changes.
```

### 定时工作流

| 工作流 | 频率 | 目的 |
|--------|------|------|
| [Code Quality](.github/workflows/scheduled-claude-code-quality.yml) | 每周（周日） | 随机目录质量审查并自动修复 |
| [Docs Sync](.github/workflows/scheduled-claude-code-docs-sync.yml) | 每月（1 号） | 确认文档与代码同步 |
| [Dependency Audit](.github/workflows/scheduled-claude-code-dependency-audit.yml) | 双周（1、15 号） | 安全更新依赖并运行测试 |

### 必备设置

在仓库 Secrets 中添加 `ANTHROPIC_API_KEY`：Settings → Secrets and variables → Actions → New repository secret。

### 成本预估

| 工作流 | 频率 | 估算成本 |
|--------|------|----------|
| PR 审查 | 每个 PR | ~$0.05 - $0.50 |
| 文档同步 | 每月 | ~$0.50 - $2.00 |
| 依赖审计 | 双周 | ~$0.20 - $1.00 |
| 代码质量 | 每周 | ~$1.00 - $5.00 |

**估算月总成本：** ~$10 - $50（取决于 PR 数量）

---

<a id="best-practices"></a>
## 最佳实践

### 1. 先写好 CLAUDE.md

包含：
- 技术栈概览
- 关键命令
- 强制规则
- 目录结构

### 2. 渐进式构建技能

不要一次写完所有内容：
1. 先记录最常用的模式
2. 随痛点增加再补充
3. 让每个技能聚焦一个领域

### 3. 用钩子自动化

交给钩子处理重复任务：
- 保存时自动格式化
- 测试文件变动时运行测试
- schema 变化时重生成 types
- 阻止在受保护分支上编辑

### 4. 为复杂流程建代理

适用场景：
- 代码审查（结合自定义清单）
- PR 创建与管理
- 调试流程
- 任务调研/上手

### 5. 利用 GitHub Actions

自动化维护：
- PR 审查
- 每周质量巡检
- 每月文档对齐
- 依赖更新

### 6. 将配置纳入版本控制

提交所有文件，除了：
- `settings.local.json`（个人偏好）
- `CLAUDE.local.md`（个人笔记）
- 用户凭据

---

<a id="examples-in-this-repository"></a>
## 仓库中的示例

| 文件 | 说明 |
|------|------|
| [CLAUDE.md](CLAUDE.md) | 项目记忆示例 |
| [.claude/settings.json](.claude/settings.json) | 完整钩子配置 |
| [.claude/settings.md](.claude/settings.md) | 钩子说明文档 |
| [.mcp.json](.mcp.json) | MCP 服务器配置 |
| **Agents** | |
| [.claude/agents/code-reviewer.md](.claude/agents/code-reviewer.md) | 代码审查代理 |
| [.claude/agents/github-workflow.md](.claude/agents/github-workflow.md) | Git 工作流代理 |
| **Commands** | |
| [.claude/commands/onboard.md](.claude/commands/onboard.md) | 深度任务探索 |
| [.claude/commands/ticket.md](.claude/commands/ticket.md) | 工单工作流（读取→实现→更新） |
| [.claude/commands/pr-review.md](.claude/commands/pr-review.md) | PR 审查流程 |
| [.claude/commands/pr-summary.md](.claude/commands/pr-summary.md) | 生成 PR 摘要 |
| [.claude/commands/code-quality.md](.claude/commands/code-quality.md) | 质量检查 |
| [.claude/commands/docs-sync.md](.claude/commands/docs-sync.md) | 文档同步 |
| **Hooks** | |
| [.claude/hooks/skill-eval.sh](.claude/hooks/skill-eval.sh) | 技能评估封装 |
| [.claude/hooks/skill-eval.js](.claude/hooks/skill-eval.js) | 技能匹配引擎 |
| [.claude/hooks/skill-rules.json](.claude/hooks/skill-rules.json) | 模式匹配规则 |
| **Skills** | |
| [.claude/skills/testing-patterns/SKILL.md](.claude/skills/testing-patterns/SKILL.md) | TDD、工厂、mock |
| [.claude/skills/systematic-debugging/SKILL.md](.claude/skills/systematic-debugging/SKILL.md) | 四阶段调试 |
| [.claude/skills/react-ui-patterns/SKILL.md](.claude/skills/react-ui-patterns/SKILL.md) | Loading/Error/Empty |
| [.claude/skills/graphql-schema/SKILL.md](.claude/skills/graphql-schema/SKILL.md) | 查询、mutation、codegen |
| [.claude/skills/core-components/SKILL.md](.claude/skills/core-components/SKILL.md) | 设计系统与 tokens |
| [.claude/skills/formik-patterns/SKILL.md](.claude/skills/formik-patterns/SKILL.md) | 表单与校验 |
| **GitHub Workflows** | |
| [.github/workflows/pr-claude-code-review.yml](.github/workflows/pr-claude-code-review.yml) | 自动 PR 审查 |
| [.github/workflows/scheduled-claude-code-docs-sync.yml](.github/workflows/scheduled-claude-code-docs-sync.yml) | 每月文档同步 |
| [.github/workflows/scheduled-claude-code-quality.yml](.github/workflows/scheduled-claude-code-quality.yml) | 每周质量检查 |
| [.github/workflows/scheduled-claude-code-dependency-audit.yml](.github/workflows/scheduled-claude-code-dependency-audit.yml) | 双周依赖更新 |

---

## 了解更多

- [Claude Code 文档](https://docs.anthropic.com/en/docs/claude-code)
- [Claude Code Action](https://github.com/anthropics/claude-code-action)
- [Anthropic API](https://docs.anthropic.com/en/api)

---

## 许可证

MIT - 欢迎将其作为项目模板使用。
