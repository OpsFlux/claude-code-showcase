---
description: 端到端处理一张 JIRA/Linear 工单
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(git:*), Bash(gh:*), Bash(npm:*), mcp__jira__*, mcp__github__*, mcp__linear__*
---

# 工单工作流

目标工单：$ARGUMENTS

## 操作指南

### 1. 阅读工单

首先拉取并理解工单内容：

```
使用 JIRA/Linear MCP 工具：
- 获取工单标题、描述、验收标准
- 查看关联的工单或 epic
- 查阅评论或附件
```

请总结：
- 要完成的事项
- 验收标准
- 任何阻塞或依赖

### 2. 探索代码库

动手前：
- 搜索相关代码位置
- 理解当前实现
- 确认需要修改的文件

### 3. 创建分支

```bash
git checkout -b {缩写}/{ticket-id}-{简要描述}
```

### 4. 实现改动

- 遵循项目既有模式（查阅相关技能）
- 先写测试（TDD）
- 分批提交

### 5. 更新工单

在进行过程中：
- 添加进度评论
- 及时更新状态（In Progress → In Review）
- 记录任何阻塞或问题

### 6. 创建 PR 并关联

准备就绪时：
- 使用 `gh pr create` 创建 PR
- 将 PR 关联到工单
- 在 PR 标题中加入工单号：`feat(PROJ-123): description`

### 7. 如果发现其他缺陷

若发现无关的 bug：
1. 新建工单描述细节
2. 如相关，链接到当前工单
3. 在 PR 描述中备注
4. 继续原任务

## 示例工作流

```
我：/ticket PROJ-123

Claude：
1. 正在从 JIRA 获取 PROJ-123...
   标题：Add user profile avatar upload（添加用户头像上传）
   描述：Users should be able to upload a profile picture...
   验收标准：
   - [ ] 个人资料页提供上传按钮
   - [ ] 支持大小不超过 5MB 的 JPG/PNG
   - [ ] 上传过程中展示加载状态

2. 搜索与 profile 相关的代码...
   找到：src/screens/Profile/ProfileScreen.tsx
   找到：src/components/Avatar/Avatar.tsx

3. 创建分支：cw/PROJ-123-avatar-upload

4. [按照 TDD 流程实现功能]

5. 将 JIRA 状态更新为 “In Review”...
   评论：“Implementation complete, PR ready for review（实现完成，PR 待评审）”

6. 创建 PR 并关联到 PROJ-123...
   已建 PR #456：feat(PROJ-123): add avatar upload to profile
```
