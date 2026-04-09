# AI Agent 配置和使用指南

本文档定义项目中 AI Agent 的配置标准、使用规范和最佳实践。

## Agent 定义

根据 LangChain 的定义：**Agent = Model + Harness**

```
Agent = LLM (大脑) + Tools (四肢) + Middleware (神经系统) + Orchestration (控制系统)
```

## Agent 类型

### 1. Coding Agent (编码 Agent)

**职责**: 辅助代码编写、重构、调试

**配置要求**:
- 强类型语言优先（如 TypeScript、Rust、Go）
- 清晰的模块边界
- 完整的类型定义

**工具集**:
- 文件读写
- 代码执行
- Git 操作
- 测试运行

### 2. Research Agent (研究 Agent)

**职责**: 代码分析、文档生成、知识检索

**配置要求**:
- 访问完整代码库
- 文档检索能力
- 引用溯源

**工具集**:
- 代码搜索
- 文档读取
- 引用追踪

### 3. Review Agent (评审 Agent)

**职责**: 代码审查、质量评估、安全扫描

**配置要求**:
- 只读访问为主
- 明确的评审标准
- 结构化输出格式

**工具集**:
- 代码分析
- 静态检查
- 安全扫描

### 4. Orchestration Agent (编排 Agent)

**职责**: 任务分解、Agent 协调、工作流管理

**配置要求**:
- 任务规划能力
- 子 Agent 管理
- 状态追踪

## Agent 配置文件

每个 Agent 需要在 `configs/` 目录配置：

```yaml
# configs/agent-config.yaml
agents:
  coding:
    model: gpt-4o
    temperature: 0.2
    max_tokens: 4096
    tools:
      - read_file
      - write_file
      - run_command
      - search_code
    constraints:
      - no_delete_without_approval
      - require_test_for_new_code
      - max_file_size: 10000

  research:
    model: gpt-4o
    temperature: 0.3
    max_tokens: 8192
    tools:
      - search_code
      - read_file
      - list_directory
    constraints:
      - read_only_mode
      - no_code_modification
```

## Agent 指令文件

### AGENTS.md 格式

每个仓库必须有 `AGENTS.md` 文件，作为 Agent 的主要指令来源：

```markdown
# AGENTS.md - Agent Instructions

## 项目概述
[项目简介]

## 技术栈
- 语言: [语言列表]
- 框架: [框架列表]
- 工具: [工具列表]

## 架构约束
[架构规则]

## 代码规范
[编码标准]

## 禁止行为
- 不要修改 [具体限制]
- 不要删除 [保护内容]
- 不要绕过 [安全检查]

## 必须遵循
- 所有新代码必须包含测试
- 所有提交必须通过 lint 检查
- 所有 PR 必须有描述

## 工具使用限制
[工具使用规则]
```

## 生命周期管理

### 1. 初始化 (Init)

```
[Init Agent]
├── 读取 AGENTS.md
├── 分析项目结构
├── 加载工具集
└── 验证配置
```

### 2. 任务执行 (Execute)

```
[Agent Loop]
├── 理解任务
├── 规划步骤
├── 执行操作
├── 验证结果
└── 循环直到完成
```

### 3. 交接 (Handoff)

```
[Handoff]
├── 生成任务摘要
├── 记录完成状态
├── 保留关键上下文
└── 传递工件
```

### 4. 暂停/恢复 (Pause/Resume)

- 状态必须可序列化
- 上下文必须持久化
- 目标必须清晰记录

## 12 Factor Agents 原则

生产级 Agent 应遵循：

1. **显式提示**: 所有行为通过 prompt 显式控制
2. **状态所有权**: 明确状态存储位置
3. **清洁暂停**: 支持中断和恢复
4. **可观测输出**: 所有操作可追踪
5. **确定性行为**: 相同输入产生相同输出
6. **资源边界**: 内存和 Token 使用受限
7. **错误处理**: 优雅降级和重试
8. **安全验证**: 操作前权限检查
9. **版本控制**: 配置可追溯
10. **测试覆盖**: 行为可测试
11. **文档化**: 工具和流程有文档
12. **可配置性**: 参数可调

## 输出规范

### 标准输出格式

```json
{
  "status": "success|error|partial",
  "message": "操作描述",
  "artifacts": [
    {
      "type": "file|test|doc",
      "path": "文件路径",
      "action": "created|modified|deleted"
    }
  ],
  "metrics": {
    "tokens_used": 1234,
    "duration_ms": 5000,
    "tools_called": 10
  }
}
```

### 错误处理

```json
{
  "status": "error",
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "描述",
    "recoverable": true,
    "suggestion": "修复建议"
  }
}
```

## 监控指标

| 指标 | 描述 | 告警阈值 |
|------|------|----------|
| success_rate | 任务成功率 | < 90% |
| avg_duration | 平均执行时间 | > 预期 2x |
| token_usage | Token 消耗 | > 日均 150% |
| error_rate | 错误率 | > 5% |
| tool_errors | 工具调用错误 | > 10% |

## 相关文档

- [HARNESS.md](./HARNESS.md) - 总体规范
- [ARCHITECTURE.md](./ARCHITECTURE.md) - 架构约束
- [docs/context-engineering.md](./docs/context-engineering.md) - 上下文工程
- [docs/evaluation.md](./docs/evaluation.md) - 评估体系
