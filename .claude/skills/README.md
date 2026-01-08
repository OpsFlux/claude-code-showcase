# Claude Code 技能

此目录存放项目专属技能，帮助 Claude 掌握代码库的领域知识与最佳实践。

## 分类索引

### 代码质量与模式
| Skill | 描述 |
|-------|------|
| [testing-patterns](./testing-patterns/SKILL.md) | Jest 测试、工厂函数、mock 策略、TDD 流程 |
| [systematic-debugging](./systematic-debugging/SKILL.md) | 四阶段调试方法、根因分析 |

### React 与 UI
| Skill | 描述 |
|-------|------|
| [react-ui-patterns](./react-ui-patterns/SKILL.md) | React 模式、加载状态、错误处理、GraphQL hooks |
| [core-components](./core-components/SKILL.md) | 设计系统组件、设计令牌、组件库 |
| [formik-patterns](./formik-patterns/SKILL.md) | 表单处理、校验、提交模式 |

### 数据与 API
| Skill | 描述 |
|-------|------|
| [graphql-schema](./graphql-schema/SKILL.md) | GraphQL 查询、Mutation、代码生成 |

## 常见任务的技能组合

### 构建新功能
1. **react-ui-patterns** —— Loading/Error/Empty 状态
2. **graphql-schema** —— 创建查询与 Mutation
3. **core-components** —— UI 实现
4. **testing-patterns** —— 编写测试（TDD）

### 构建表单
1. **formik-patterns** —— 表单结构与校验
2. **graphql-schema** —— 提交时的 Mutation
3. **react-ui-patterns** —— Loading / Error 处理

### 调试问题
1. **systematic-debugging** —— 根因分析
2. **testing-patterns** —— 先写失败测试

## 技能工作方式

技能会在 Claude 检测到相关上下文时自动启用。每个技能提供：

- **适用场景** —— 何时触发
- **核心模式** —— 最佳实践与示例
- **反模式** —— 需要避免的做法
- **集成** —— 与其他技能如何协作

## 新增技能流程

1. 创建目录：`.claude/skills/skill-name/`
2. 添加 `SKILL.md`（大小写敏感），并写入 YAML 前言：
   ```yaml
   ---
   # 必填
   name: skill-name              # 小写、连字符，<=64 字符
   description: 功能与使用场景，包含触发关键词。  # <=1024 字符

   # 可选
   allowed-tools: Read, Grep, Glob
   model: claude-sonnet-4-20250514
   ---
   ```
3. 包含标准章节：When to Use、Core Patterns、Anti-Patterns、Integration
4. 在本 README 中登记
5. 在 `.claude/hooks/skill-rules.json` 中添加触发规则

**重要：** `description` 字段至关重要，Claude 依靠其语义匹配自动激活技能，请写入用户常提到的关键词。

## 维护建议

- 模式变化时及时更新技能
- 移除过期内容
- 新模式出现时补充
- 确保示例与代码库保持同步
