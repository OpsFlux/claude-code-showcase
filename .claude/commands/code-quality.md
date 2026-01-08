---
description: 对指定目录执行代码质量检查
allowed-tools: Read, Glob, Grep, Bash(npm:*), Bash(npx:*)
---

# Code Quality Review

请审查目录：$ARGUMENTS

## 指南

1. **确定需要检查的文件**：
   - 查找目录中的所有 `.ts`、`.tsx` 文件
   - 排除测试文件和自动生成文件

2. **运行自动化检查**：
   ```bash
   npm run lint -- $ARGUMENTS
   npm run typecheck
   ```

3. **人工检查清单**：
   - [ ] 不出现 TypeScript `any`
   - [ ] 错误处理完善
   - [ ] 正确处理加载状态
   - [ ] 列表具备空状态
   - [ ] Mutations 具有 onError 处理
   - [ ] 异步过程中按钮已禁用

4. **按严重级别输出问题**：
   - Critical（必须修复）
   - Warning（应该修复）
   - Suggestion（改进建议）
