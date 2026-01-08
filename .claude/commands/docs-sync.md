---
description: 检查文档是否与代码保持同步
allowed-tools: Read, Glob, Grep, Bash(git:*)
---

# Documentation Sync

确认文档与当前代码状态一致。

## 指南

1. **查找近期代码改动**：
   ```bash
   git log --since="30 days ago" --name-only --pretty=format: -- "*.ts" "*.tsx" | sort -u
   ```

2. **定位相关文档**：
   - 在 `/docs/` 中搜索提到相关代码的文件
   - 检查变更附近的 README
   - 查看改动文件中的 TSDoc 注释

3. **核对文档准确性**：
   - 示例代码还能运行吗？
   - API 签名是否正确？
   - Prop 类型是否更新？

4. **只报告真实问题**：
   - 文档是持续演进的
   - 只标记错误内容，而不是“缺少”
   - 不要为了写文档而写

5. **输出需要更新的文档清单**
