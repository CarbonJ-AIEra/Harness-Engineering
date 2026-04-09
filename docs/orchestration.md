# 编排策略

本文档定义如何协调和管理多个 AI Agent 的工作流程。

## 核心理念

编排是 Agent 的"大脑"，负责：
- 任务分解
- 子 Agent 协调
- 状态管理
- 结果聚合

## 编排模式

### 1. 单 Agent 模式

```
┌─────────────────────────────────────────────────────────────┐
│                   单 Agent 编 排                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   用户请求 ──▶ 单一 Agent ──▶ 执行任务 ──▶ 返回结果           │
│                                                             │
│   适用场景:                                                 │
│   - 简单任务                                                │
│   - 单一领域                                                │
│   - 上下文简单                                              │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 2. 串行编排 (Sequential)

```
┌─────────────────────────────────────────────────────────────┐
│                   串 行 编 排                               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   Agent A ──▶ Agent B ──▶ Agent C ──▶ Agent D             │
│      │           │           │           │                  │
│      ▼           ▼           ▼           ▼                  │
│   结果1       结果2       结果3       结果4                  │
│      │           │           │           │                  │
│      └───────────┴───────────┴───────────┘                  │
│                          │                                   │
│                          ▼                                   │
│                    最终结果                                  │
│                                                             │
│   适用: 有依赖关系的任务                                      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 3. 并行编排 (Parallel)

```
┌─────────────────────────────────────────────────────────────┐
│                   并 行 编 排                               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│                        任务                                  │
│                         │                                   │
│          ┌──────────────┼──────────────┐                   │
│          ▼              ▼              ▼                   │
│     ┌─────────┐    ┌─────────┐    ┌─────────┐              │
│     │Agent A  │    │Agent B  │    │Agent C  │              │
│     │处理模块A│    │处理模块B│    │处理模块C│              │
│     └────┬────┘    └────┬────┘    └────┬────┘              │
│          │              │              │                    │
│          └──────────────┼──────────────┘                   │
│                         │                                   │
│                         ▼                                   │
│                   合并结果                                   │
│                                                             │
│   适用: 独立子任务                                          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 4. 分层编排 (Hierarchical)

```
┌─────────────────────────────────────────────────────────────┐
│                   分 层 编 排                                │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│                 ┌───────────────┐                          │
│                 │ Orchestrator  │  ← 主管 Agent            │
│                 │   (任务规划)    │                          │
│                 └───────┬───────┘                          │
│                         │                                    │
│          ┌──────────────┼──────────────┐                   │
│          ▼              ▼              ▼                     │
│    ┌──────────┐   ┌──────────┐   ┌──────────┐               │
│    │ Supervisor│   │ Supervisor│   │ Supervisor│             │
│    │  Agent A  │   │  Agent B  │   │  Agent C  │             │
│    └────┬─────┘   └────┬─────┘   └────┬─────┘               │
│         │              │              │                      │
│    ┌────┴────┐    ┌────┴────┐    ┌────┴────┐              │
│    │执行 Agent│    │执行 Agent│    │执行 Agent│              │
│    │  A1, A2  │    │  B1, B2  │    │  C1, C2  │              │
│    └─────────┘    └─────────┘    └─────────┘               │
│                                                             │
│   适用: 复杂多领域任务                                       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 5. 循环编排 (Loop)

```
┌─────────────────────────────────────────────────────────────┐
│                   循 环 编 排                                │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│      ┌──────────────────────────────────────┐              │
│      │                                      │              │
│      ▼                                      │              │
│   ┌─────┐      ┌─────┐      ┌─────┐       │              │
│   │执行 │ ───▶ │验证 │ ───▶ │修复 │ ────────┘              │
│   │任务 │      │结果 │      │问题 │                        │
│   └─────┘      └─────┘      └─────┘                        │
│                       │                                     │
│                       ▼                                     │
│                  ┌─────────┐                               │
│                  │  继续?  │                               │
│                  └────┬────┘                               │
│                       │                                     │
│              ┌────────┴────────┐                            │
│              ▼                ▼                            │
│           [是]            [否/达到上限]                      │
│              │                │                            │
│              └────────────────┘                            │
│                       │                                     │
│                       ▼                                     │
│                  完成任务                                   │
│                                                             │
│   适用: 迭代优化任务                                         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## 状态管理

### 1. 状态结构

```yaml
# 任务状态定义
task_state:
  id: "task-001"
  type: "feature_development"
  status: "in_progress"

  # 生命周期状态
  lifecycle:
    - name: "pending"
      description: "任务等待分配"
    - name: "planning"
      description: "任务规划中"
    - name: "executing"
      description: "任务执行中"
    - name: "validating"
      description: "结果验证中"
    - name: "completed"
      description: "任务完成"
    - name: "failed"
      description: "任务失败"

  # 上下文快照
  context:
    goal: "实现用户认证功能"
    progress: 0.6
    critical_files: ["src/auth/login.ts", "src/auth/verify.ts"]
    failing_tests: []

  # 步骤历史
  steps:
    - id: 1
      action: "read_files"
      status: "completed"
      result: "读取了 5 个文件"

    - id: 2
      action: "write_code"
      status: "completed"
      result: "创建了 2 个文件"

    - id: 3
      action: "run_tests"
      status: "failed"
      error: "测试失败: 3/10 通过"
```

### 2. 状态持久化

```yaml
# state-persistence.yaml
persistence:
  backend: "redis"
  ttl: 3600  # 1小时

  snapshots:
    interval: 300  # 每5分钟
    storage: "s3://agent-states/"

  checkpoints:
    on_tool_call: true
    on_error: true
    on_completion: true
```

## 任务分解

### 1. 分解策略

```yaml
task_decomposition:
  strategies:
    - name: "linear"
      description: "顺序分解"
     适用: "有明确顺序的任务"

    - name: "parallel"
      description: "并行分解"
     适用: "独立子任务"

    - name: "tree"
      description: "树状分解"
      适用: "复杂多层次任务"

  rules:
    max_depth: 5
    max_tasks_per_level: 10
    min_task_complexity: "需要至少2个工具调用"
```

### 2. 分解示例

```
原始任务: "实现用户注册功能"

├── 分解为:
│
├── 1. 数据库设计
│   ├── 创建用户表
│   └── 编写迁移脚本
│
├── 2. 后端 API
│   ├── POST /register
│   ├── POST /verify-email
│   └── 单元测试
│
├── 3. 前端表单
│   ├── 注册表单组件
│   ├── 表单验证
│   └── 错误处理
│
└── 4. 集成
    ├── API 集成测试
    └── E2E 测试
```

## 通信协议

### 1. Agent 间消息格式

```json
{
  "message_id": "msg-001",
  "from": "orchestrator",
  "to": "agent-coder",
  "type": "task_assignment",
  "payload": {
    "task_id": "task-002",
    "description": "实现登录功能",
    "constraints": ["使用 JWT", "60分钟内完成"],
    "context": {
      "related_files": ["src/auth/types.ts"],
      "test_examples": ["tests/auth/login.spec.ts"]
    }
  },
  "reply_to": "msg-001-response"
}
```

### 2. 响应格式

```json
{
  "in_response_to": "msg-001",
  "status": "completed|partial|failed",
  "result": {
    "files_created": [],
    "files_modified": [],
    "tests_passed": true,
    "artifacts": []
  },
  "metadata": {
    "duration_ms": 45000,
    "tokens_used": 8000,
    "steps": 12
  }
}
```

## 容错机制

### 1. 重试策略

```yaml
retry:
  max_attempts: 3
  backoff:
    type: "exponential"
    initial_delay: 1000  # ms
    multiplier: 2
    max_delay: 30000

  retryable_errors:
    - "network_timeout"
    - "rate_limit"
    - "temporary_failure"

  non_retryable_errors:
    - "invalid_input"
    - "permission_denied"
    - "resource_not_found"
```

### 2. 降级策略

```yaml
degradation:
  strategies:
    - trigger: "primary_agent_timeout"
      action: "fallback_to_simpler_agent"

    - trigger: "context_exhaustion"
      action: "reduce_task_scope"

    - trigger: "tool_unavailable"
      action: "use_alternative_tool"
```

## 工作流配置

```yaml
# workflow.yaml
workflows:
  feature_development:
    type: "hierarchical"
    orchestrator:
      model: "gpt-4o"
      role: "task_planner"

    supervisors:
      - domain: "backend"
        model: "gpt-4o"
        agents: 3

      - domain: "frontend"
        model: "gpt-4o"
        agents: 2

    execution:
      mode: "sequential_with_parallel"
      checkpoint_interval: 60

    termination:
      conditions:
        - "all_tasks_completed"
        - "max_duration: 3600"
        - "failure_threshold: 0.3"

  bug_fix:
    type: "loop"
    max_iterations: 5

    steps:
      - name: "analyze"
        agent: "researcher"
      - name: "fix"
        agent: "coder"
      - name: "verify"
        agent: "tester"
      - name: "review"
        agent: "reviewer"

    exit_conditions:
      - "all_tests_pass"
      - "no_new_errors"
      - "iteration_limit"
```

## 工具支持

### 编排框架

| 框架 | 特点 | 适用场景 |
|------|------|----------|
| LangChain | 链式组合 | 简单工作流 |
| CrewAI | 多 Agent 协作 | 团队协作 |
| AutoGen | 对话式编排 | 复杂交互 |
| Temporal | 工作流引擎 | 生产级 |
| Prefect | 数据流水线 | 数据处理 |

## 相关文档

- [HARNESS.md](../HARNESS.md) - 总体规范
- [AGENTS.md](../AGENTS.md) - Agent 配置
- [context-engineering.md](./context-engineering.md) - 上下文工程
- [observability.md](./observability.md) - 可观测性
