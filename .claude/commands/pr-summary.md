---
description: 为当前分支的改动生成摘要
allowed-tools: Bash(git:*)
---

# PR Summary

为当前分支的改动生成一份 PR 摘要。

## 指南

1. **分析改动**：
   ```bash
   git log main..HEAD --oneline
   git diff main...HEAD --stat
   ```

2. **撰写摘要**，包含：
   - 简述改动内容
   - 修改的关键文件列表
   - 若有，说明破坏性变更
   - 测试说明

3. **按 PR 模板排版**：
   ```markdown
   ## Summary
   [1-3 条概述]

   ## Changes
   - [关键改动列表]

   ## Test Plan
   - [ ] [测试检查项]
   ```
