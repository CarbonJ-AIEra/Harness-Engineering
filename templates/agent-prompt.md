# Agent 提示词模板

本文档提供标准化的 Agent 提示词模板。

## 1. 系统提示词模板

```markdown
# 系统提示词模板

## 项目概述

{project_overview}

## 技术栈

- **语言**: {languages}
- **框架**: {frameworks}
- **工具**: {tools}

## 架构原则

{architecture_principles}

## 代码规范

### 命名约定
- 组件/类: PascalCase
- 函数/方法: camelCase
- 常量: SCREAMING_SNAKE_CASE
- 文件: kebab-case

### 必需实践
- 所有函数必须有类型定义
- 所有公共 API 必须有文档注释
- 所有错误必须被处理
- 所有副作用必须明确标注

### 禁止实践
- 禁止使用 any 类型
- 禁止使用 var
- 禁止 console.log 用于生产代码
- 禁止直接操作 DOM

## 工具使用规则

| 工具 | 用途 | 限制 |
|------|------|------|
| read_file | 读取代码 | 无限制 |
| write_file | 创建文件 | 需要 backup |
| edit_file | 修改代码 | 必须精确匹配 |
| run_command | 执行命令 | 仅允许列表中的命令 |

## 工作流程

1. 理解任务需求
2. 制定执行计划
3. 逐步执行并验证
4. 提交前运行检查
5. 报告完成状态

## 输出格式

完成每个任务后，报告：
- 创建/修改的文件
- 通过的测试
- 遇到的问题
- 建议的后续步骤
```

## 2. 任务执行提示词

```markdown
## 任务: {task_name}

### 描述
{task_description}

### 验收标准
- [ ] {acceptance_criteria_1}
- [ ] {acceptance_criteria_2}
- [ ] {acceptance_criteria_3}

### 约束条件
- 时间限制: {time_limit}
- 范围限制: {scope_constraints}

### 相关上下文
{relevant_context}

### 预期产出
{expected_output}

---

开始执行任务。

1. **计划阶段**: 分解任务为可执行的步骤
2. **执行阶段**: 按计划执行，记录每个步骤
3. **验证阶段**: 验证产出符合验收标准
4. **报告阶段**: 汇报结果和建议
```

## 3. 代码审查提示词

```markdown
## 代码审查任务

### 待审查代码
```typescript
{code_to_review}
```

### 审查维度

#### 正确性
- 代码逻辑是否正确？
- 边界情况是否处理？
- 错误处理是否完善？

#### 可读性
- 命名是否清晰？
- 复杂度是否适中？
- 注释是否必要且准确？

#### 性能
- 是否存在性能问题？
- 是否有优化空间？

#### 安全性
- 是否存在安全漏洞？
- 输入是否验证？

#### 最佳实践
- 是否符合项目规范？
- 是否遵循 SOLID 原则？

---

请提供详细的审查报告：

```json
{
  "issues": [
    {
      "severity": "critical|major|minor",
      "location": "文件:行号",
      "description": "问题描述",
      "suggestion": "修复建议"
    }
  ],
  "summary": "总体评价",
  "approval": "approved|needs_changes|rejected"
}
```
```

## 4. 调试任务提示词

```markdown
## 调试任务

### 问题描述
{problem_description}

### 错误信息
```
{error_message}
```

### 相关代码
```typescript
{related_code}
```

### 复现步骤
1. {step_1}
2. {step_2}
3. {step_3}

### 环境信息
- 语言版本: {language_version}
- 依赖版本: {dependency_versions}
- 运行环境: {runtime}

---

调试步骤：

1. **分析错误**: 理解错误类型和位置
2. **定位根因**: 追踪错误来源
3. **制定方案**: 设计修复策略
4. **实施修复**: 执行修复
5. **验证修复**: 确认问题解决
6. **防止回归**: 添加测试
```

## 5. 测试编写提示词

```markdown
## 测试编写任务

### 待测试代码
```typescript
{code_to_test}
```

### 测试要求

#### 测试覆盖
- 正常路径测试
- 边界条件测试
- 错误处理测试
- 性能测试（如适用）

#### 测试风格
- 使用 Given-When-Then 格式
- 测试名称清晰描述意图
- 每个测试只验证一个行为

### 示例测试结构
```typescript
describe('功能名称', () => {
  describe('正常路径', () => {
    it('应该 [期望行为]', () => {
      // Given
      const input = ...;

      // When
      const result = ...;

      // Then
      expect(result).toBe(...);
    });
  });

  describe('边界条件', () => {
    it('应该在输入为空时 [处理行为]', () => {
      // ...
    });
  });

  describe('错误处理', () => {
    it('应该在输入无效时抛出 [错误类型]', () => {
      // ...
    });
  });
});
```
```

## 6. 架构约束提示词

```markdown
## 架构约束提醒

### 当前文件位置
`{file_path}`

### 模块层级
`{layer}` (presentation/application/domain/infrastructure)

### 允许的依赖
- 可依赖: {allowed_dependencies}
- 禁止依赖: {forbidden_dependencies}

### 架构规则
{architecture_rules}

---

在修改代码前，请确认：
1. 不会引入违规的依赖关系
2. 遵守模块边界
3. 使用正确的导入路径

如需修改架构，请先讨论并更新 ARCHITECTURE.md。
```

## 7. 反馈生成提示词

```markdown
## 任务完成报告

### 任务概要
- 任务名称: {task_name}
- 执行时间: {duration}
- Token 消耗: {tokens_used}

### 变更清单

**创建的文件:**
- {new_files}

**修改的文件:**
- {modified_files}

**删除的文件:**
- {deleted_files}

### 测试结果
- 通过: {tests_passed}
- 失败: {tests_failed}
- 覆盖率: {coverage}%

### 验证清单
- [ ] 编译通过
- [ ] 测试通过
- [ ] Lint 通过
- [ ] 类型检查通过
- [ ] 功能符合预期

### 遇到的问题
{issues_encountered}

### 后续建议
{follow_up_suggestions}

---

**状态**: {status}
- ✅ 成功完成
- ⚠️ 部分完成
- ❌ 失败
```

## 相关文档

- [AGENTS.md](../AGENTS.md) - Agent 配置
- [ARCHITECTURE.md](../ARCHITECTURE.md) - 架构约束
- [docs/context-engineering.md](../docs/context-engineering.md) - 上下文工程
