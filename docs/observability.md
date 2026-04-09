# 可观测性设计

本文档定义 AI Agent 的可观测性设计方案。

## 核心理念

**Agent 的每个操作都必须是可追踪的。**

可观测性是 Harness 的"检测传感器"，用于监控 Agent 行为并提供反馈。

## 可观测性三要素

```
┌─────────────────────────────────────────────────────────────┐
│                   可 观 测 性 三 要 素                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│   │    日志     │  │    指标     │  │    追踪     │        │
│   │  (Logs)     │  │ (Metrics)   │  │ (Traces)    │        │
│   ├─────────────┤  ├─────────────┤  ├─────────────┤        │
│   │ 事件记录    │  │ 聚合数值    │  │ 请求链路    │        │
│   │ 详细描述    │  │ 趋势分析    │  │ 因果关系    │        │
│   │ 时间戳      │  │ 告警触发    │  │ 性能分析    │        │
│   └─────────────┘  └─────────────┘  └─────────────┘        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## 日志设计

### 1. 日志级别

| 级别 | 用途 | 示例 |
|------|------|------|
| DEBUG | 开发调试 | "读取文件: /path/to/file" |
| INFO | 正常操作 | "任务开始: feature-xyz" |
| WARN | 警告信息 | "测试失败，但继续执行" |
| ERROR | 错误信息 | "无法连接数据库" |
| CRITICAL | 严重问题 | "Agent 陷入死循环" |

### 2. 日志格式

```json
{
  "timestamp": "2024-01-15T10:30:00.000Z",
  "level": "INFO",
  "agent_id": "agent-001",
  "session_id": "session-abc123",
  "event": "task_start",
  "data": {
    "task_id": "task-xyz",
    "task_type": "feature_development",
    "target_file": "src/services/user.ts",
    "context_tokens": 2048
  },
  "metadata": {
    "model": "gpt-4o",
    "temperature": 0.2,
    "tools_available": ["read", "write", "bash"]
  }
}
```

### 3. 关键事件日志

```yaml
# 必须记录的事件
required_events:
  - task_received       # 接收任务
  - task_parsed         # 任务解析
  - plan_created        # 计划创建
  - tool_called         # 工具调用
  - tool_completed      # 工具完成
  - tool_failed         # 工具失败
  - decision_made       # 决策点
  - error_occurred      # 错误发生
  - retry_attempted     # 重试
  - task_completed      # 任务完成
  - task_failed         # 任务失败
```

### 4. 日志收集

```yaml
# logging.yaml
logging:
  sinks:
    - name: "stdout"
      format: "json"
      level: "INFO"

    - name: "file"
      path: "logs/agent-{date}.log"
      rotation: "daily"
      retention: 30

    - name: "remote"
      endpoint: "https://logs.example.com"
      buffer_size: 100
      flush_interval: 5

  sampling:
    debug: 0.1   # 10% 采样
    info: 1.0    # 100% 记录
    warn: 1.0
    error: 1.0
```

## 指标设计

### 1. 核心指标

```yaml
# metrics/core.yaml
metrics:
  agent_performance:
    - name: "task_success_rate"
      type: "gauge"
      description: "任务成功率"
      labels: ["task_type", "agent_id"]

    - name: "task_duration_seconds"
      type: "histogram"
      description: "任务执行时间"
      buckets: [10, 30, 60, 120, 300, 600]

    - name: "token_usage_total"
      type: "counter"
      description: "Token 总消耗"
      labels: ["model", "task_type"]

    - name: "tool_call_count"
      type: "counter"
      description: "工具调用次数"
      labels: ["tool_name", "status"]

  harness_health:
    - name: "feedback_latency_ms"
      type: "histogram"
      description: "反馈延迟"

    - name: "constraint_violations"
      type: "counter"
      description: "约束违反次数"
      labels: ["constraint_type"]

    - name: "context_exhaustion_rate"
      type: "gauge"
      description: "上下文耗尽率"
```

### 2. 指标面板

```
┌─────────────────────────────────────────────────────────────┐
│                   Agent Dashboard                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  概览                    效率指标              健康状态      │
│  ┌─────────────┐       ┌─────────────┐     ┌─────────────┐  │
│  │ 任务成功率  │       │ 平均耗时    │     │ 系统状态    │  │
│  │   94.5%    │       │   45s      │     │   正常     │  │
│  └─────────────┘       └─────────────┘     └─────────────┘  │
│                                                             │
│  任务趋势                    工具使用                    错误分布  │
│  ┌────────────────────────┐  ┌────────────────────────┐   │
│  │     📈 Task Trend       │  │     🔧 Tool Usage      │   │
│  │                        │  │                        │   │
│  │  ████                  │  │  read ████████ 45%     │   │
│  │  ████                  │  │  write ████ 25%        │   │
│  │  ████                  │  │  bash ███ 20%          │   │
│  │  ████                  │  │  search ██ 10%         │   │
│  │  ──────────────────    │  │                        │   │
│  │  Mon Tue Wed Thu Fri    │  └────────────────────────┘   │
│  └────────────────────────┘                                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## 追踪设计

### 1. 追踪结构

```json
{
  "trace_id": "abc123-def456",
  "span_id": "span-001",
  "parent_id": null,
  "operation": "feature_development",
  "start_time": "2024-01-15T10:30:00.000Z",
  "end_time": "2024-01-15T10:35:00.000Z",
  "duration_ms": 300000,
  "attributes": {
    "agent_id": "agent-001",
    "task_type": "feature",
    "files_modified": 3,
    "tests_added": 5
  },
  "events": [
    {
      "timestamp": "...",
      "name": "plan_created",
      "attributes": {"steps": 5}
    },
    {
      "timestamp": "...",
      "name": "tool_call",
      "attributes": {"tool": "read_file", "file": "src/a.ts"}
    }
  ],
  "children": [
    {"span_id": "span-002", "operation": "read_files"},
    {"span_id": "span-003", "operation": "write_code"},
    {"span_id": "span-004", "operation": "run_tests"}
  ]
}
```

### 2. 追踪采样

```yaml
# tracing.yaml
tracing:
  sampler:
    type: "probabilistic"
    rate: 0.1  # 10% 采样

  storage:
    backend: "jaeger"
    endpoint: "http://jaeger:14268/api/traces"

  instrumentation:
    automatic: true
    manual_spans: true

  retention:
    days: 7
```

## 可视化分析

### 1. Agent 行为图

```
┌─────────────────────────────────────────────────────────────┐
│                   任 务 执 行 轨 迹                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   [任务开始]                                                  │
│       │                                                     │
│       ▼                                                     │
│   [读取上下文] ──▶ [分析需求] ──▶ [制定计划]                   │
│       │                 │               │                   │
│       │                 │               ▼                   │
│       │                 │         [创建分支]                 │
│       │                 │               │                   │
│       │                 │               ▼                   │
│       │                 │         [实现功能] ◀──┐            │
│       │                 │               │      │            │
│       │                 │               ▼      │            │
│       │                 │         [编写测试]    │            │
│       │                 │               │      │            │
│       │                 │               ▼      │            │
│       │                 │         [运行测试] ───┴──▶ [失败]   │
│       │                 │               │                   │
│       │                 │               ▼                   │
│       │                 │         [调试修复] ──▶ [通过]       │
│       │                 │               │                   │
│       │                 │               ▼                   │
│       ▼                 ▼         [提交代码]                 │
│   [任务完成]                                                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 2. 错误分析

```
┌─────────────────────────────────────────────────────────────┐
│                   错 误 分 布 分 析                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   错误类型                    频率        根本原因           │
│   ┌─────────────────────┬───────────┬─────────────────┐    │
│   │ 文件不存在           │   45%     │ 路径引用错误     │    │
│   │ 测试失败             │   30%     │ 边界条件未处理   │    │
│   │ 依赖冲突             │   15%     │ 版本不兼容       │    │
│   │ 权限不足             │   10%     │ 操作范围限制     │    │
│   └─────────────────────┴───────────┴─────────────────┘    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## 告警规则

```yaml
# alerts.yaml
alerts:
  - name: "high_failure_rate"
    condition: "task_success_rate < 0.85"
    severity: "critical"
    window: "5m"
    action: "slack_notify"

  - name: "slow_task_completion"
    condition: "task_duration_p95 > 600"
    severity: "warning"
    window: "15m"
    action: "log_analysis"

  - name: "context_exhaustion"
    condition: "context_exhaustion_rate > 0.1"
    severity: "warning"
    window: "10m"
    action: "optimize_harness"

  - name: "tool_failure_spike"
    condition: "tool_error_rate > 0.05"
    severity: "critical"
    window: "5m"
    action: "investigate_tool"
```

## 实现示例

```typescript
// observability/agent-tracker.ts
import { trace, metrics, logs } from '@opentelemetry/sdk-node';

class AgentTracker {
  private tracer = trace.getTracer('agent');
  private meter = metrics.getMeter('agent');
  private logger = logs.getLogger('agent');

  // 计数器
  private taskCounter = this.meter.createCounter('tasks_total', {
    description: 'Total number of tasks'
  });

  // 直方图
  private taskDuration = this.meter.createHistogram('task_duration_seconds', {
    description: 'Task duration in seconds'
  });

  // 追踪任务
  async trackTask<T>(taskId: string, fn: () => Promise<T>): Promise<T> {
    return this.tracer.startActiveSpan('task', async (span) => {
      try {
        this.logger.info('Task started', { taskId });
        this.taskCounter.add(1, { status: 'started' });

        const start = Date.now();
        const result = await fn();
        const duration = (Date.now() - start) / 1000;

        this.taskDuration.record(duration);
        span.setStatus({ code: SpanStatusCode.OK });
        this.logger.info('Task completed', { taskId, duration });

        return result;
      } catch (error) {
        span.recordException(error as Error);
        span.setStatus({ code: SpanStatusCode.ERROR });
        this.taskCounter.add(1, { status: 'failed' });
        this.logger.error('Task failed', { taskId, error });
        throw error;
      } finally {
        span.end();
      }
    });
  }
}
```

## 相关文档

- [HARNESS.md](../HARNESS.md) - 总体规范
- [AGENTS.md](../AGENTS.md) - Agent 配置
- [evaluation.md](./evaluation.md) - 评估体系
- [orchestration.md](./orchestration.md) - 编排策略
