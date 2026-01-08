# Claude Code 设置文档

## 环境变量

- `INSIDE_CLAUDE_CODE`: "1" - 表示当前运行在 Claude Code 环境中
- `BASH_DEFAULT_TIMEOUT_MS`: Bash 命令默认超时时间（7 分钟）
- `BASH_MAX_TIMEOUT_MS`: Bash 命令允许的最大超时时间

## 钩子（Hooks）

### UserPromptSubmit

- **技能评估**：分析提示并推荐相关技能
  - **脚本**：`.claude/hooks/skill-eval.sh`
  - **行为**：匹配关键词、文件路径与模式来建议技能

### PreToolUse

- **主分支保护**：阻止在 main 分支上直接修改文件（超时 5 秒）
  - **触发条件**：在 Edit、MultiEdit 或 Write 等工具修改文件前
  - **行为**：若仍在 main 分支则阻止编辑，提示创建特性分支

### PostToolUse

1. **代码格式化**：自动格式化 JS/TS 文件（超时 30 秒）
   - **触发条件**：编辑 `.js`、`.jsx`、`.ts`、`.tsx` 后
   - **命令**：`npx prettier --write`（或 Biome）
   - **行为**：自动整理代码，如遇错误会反馈

2. **NPM 安装**：`package.json` 改动后自动安装依赖（超时 60 秒）
   - **触发条件**：编辑 `package.json`
   - **命令**：`npm install`
   - **行为**：安装依赖，如失败则本次编辑视为失败

3. **测试运行器**：测试文件更新后运行对应测试（超时 90 秒）
   - **触发条件**：编辑 `.test.js`、`.test.jsx`、`.test.ts`、`.test.tsx`
   - **命令**：`npm test -- --findRelatedTests <file> --passWithNoTests`
   - **行为**：运行关联测试，展示结果，不阻塞提交流程

4. **TypeScript 检查**：TS/TSX 文件更新后进行类型检查（超时 30 秒）
   - **触发条件**：编辑 `.ts`、`.tsx`
   - **命令**：`npx tsc --noEmit`
   - **行为**：仅展示首个错误集，不阻塞

## 钩子响应格式

```json
{
  "feedback": "要展示的信息",
  "suppressOutput": true,
  "block": true,
  "continue": false
}
```

## 钩子中可用的环境变量

- `$CLAUDE_TOOL_INPUT_FILE_PATH`: 当前被编辑的文件
- `$CLAUDE_TOOL_NAME`: 正在使用的工具
- `$CLAUDE_PROJECT_DIR`: 项目根目录

## 退出码

- `0`: 成功
- `1`: 非阻塞错误（仅提示反馈）
- `2`: 阻塞错误（仅 PreToolUse，可阻止操作）
