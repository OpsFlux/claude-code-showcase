# 项目名称

> 这是一个示例 CLAUDE.md，用于展示如何为项目配置 Claude Code。

## 关键信息

- **技术栈**：React、TypeScript、Node.js
- **测试命令**：`npm test`
- **Lint 命令**：`npm run lint`
- **构建命令**：`npm run build`

## 关键目录

- `src/components/` - React 组件
- `src/hooks/` - 自定义 React hooks
- `src/utils/` - 工具函数
- `src/api/` - API 客户端代码
- `tests/` - 测试文件

## 代码风格

- 启用 TypeScript 严格模式
- 更倾向 `interface` 而不是 `type`（联合/交叉除外）
- 禁止 `any`，请使用 `unknown`
- 优先使用提前返回，避免嵌套条件
- 倡导组合优先于继承

## Git 约定

- **分支命名**：`{缩写}/{描述}`（例如 `jd/fix-login`）
- **提交格式**：Conventional Commits（`feat:`、`fix:`、`docs:` 等）
- **PR 标题**：沿用提交格式

## 关键规则

### 错误处理
- 切勿默默吞掉错误
- 必须向用户展示错误反馈
- 记录日志以便调试

### UI 状态
- 必须处理 loading、error、empty、success 状态
- 仅在没有数据时显示全屏 loading
- 每个列表都需要空状态

### Mutations
- 异步操作期间禁用按钮
- 在按钮上显示加载指示
- 必须提供 onError，并给出用户反馈

## 测试

- 先写失败的测试（TDD）
- 使用工厂模式：`getMockX(overrides)`
- 测试行为而不是实现细节
- 提交前运行全部测试

## 技能激活

在实现任何任务之前，先确认相关技能：

- 编写测试 → `testing-patterns`
- 构建表单 → `formik-patterns`
- GraphQL 操作 → `graphql-schema`
- 调试问题 → `systematic-debugging`
- UI 组件 → `react-ui-patterns`

## 常用命令

```bash
# 开发
npm run dev          # 启动开发服务器
npm test             # 运行测试
npm run lint         # 执行 lint
npm run typecheck    # 类型检查

# Git
npm run commit       # 交互式提交
gh pr create         # 创建 PR
```
