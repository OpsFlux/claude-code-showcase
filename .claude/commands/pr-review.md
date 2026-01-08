---
description: 根据项目标准审查一个 Pull Request
allowed-tools: Read, Glob, Grep, Bash(git:*), Bash(gh:*)
---

# PR Review

请审查以下 PR：$ARGUMENTS

## 指南

1. **获取 PR 信息**：
   - 运行 `gh pr view $ARGUMENTS` 查看详情
   - 运行 `gh pr diff $ARGUMENTS` 浏览改动

2. **阅读审查标准**：
   - 打开 `.claude/agents/code-reviewer.md` 获取检查清单

3. **对所有变更应用清单**：
   - 是否符合 TypeScript 严格模式
   - 错误处理是否到位
   - Loading / Error / Empty 状态是否完整
   - 测试覆盖度
   - 文档是否更新

4. **输出结构化反馈**：
   - **Critical**：合并前必须修复
   - **Warning**：应当修复
   - **Suggestion**：可选优化

5. **使用 `gh pr comment` 发布审查意见**
