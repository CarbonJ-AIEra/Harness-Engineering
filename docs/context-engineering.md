# 上下文工程

本文档定义如何管理 AI Agent 的上下文窗口，确保 Agent 获得正确的信息。

## 核心理念

**仓库必须是单一事实来源。**

上下文窗口应作为"工作内存预算"而非"垃圾场"来管理。

## 上下文管理策略

### 1. 上下文分层

```
┌─────────────────────────────────────────────────────────────┐
│                    上 下 文 分 层 结 构                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ L1: 系统级上下文 (System Prompt)                      │   │
│  │ - 项目概述                                            │   │
│  │ - 技术栈                                              │   │
│  │ - 通用规范                                            │   │
│  └─────────────────────────────────────────────────────┘   │
│                          ▼                                  │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ L2: 仓库级上下文 (AGENTS.md)                           │   │
│  │ - 架构约束                                            │   │
│  │ - 代码规范                                            │   │
│  │ - 工作流程                                            │   │
│  └─────────────────────────────────────────────────────┘   │
│                          ▼                                  │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ L3: 任务级上下文 (Task Prompt)                        │   │
│  │ - 具体任务描述                                         │   │
│  │ - 预期结果                                            │   │
│  │ - 约束条件                                            │   │
│  └─────────────────────────────────────────────────────┘   │
│                          ▼                                  │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ L4: 会话级上下文 (Conversation)                        │   │
│  │ - 历史交互                                            │   │
│  │ - 当前状态                                            │   │
│  │ - 临时数据                                            │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 2. 上下文注入策略

#### 静态注入 (Static)

项目配置和文档在初始化时加载：

```yaml
# configs/context-injection.yaml
static_context:
  files:
    - path: AGENTS.md
      priority: high
      always_included: true

    - path: ARCHITECTURE.md
      priority: high
      always_included: true

    - path: README.md
      priority: medium
      always_included: false

    - path: package.json
      priority: medium
      always_included: true
```

#### 动态注入 (Dynamic)

根据任务类型和上下文自动加载：

```yaml
dynamic_context:
  rules:
    - trigger: "type:frontend"
      load:
        - FRONTEND.md
        - components/**/*.md

    - trigger: "type:backend"
      load:
        - API.md
        - services/**/*.md

    - trigger: "task:test"
      load:
        - TESTING.md
        - tests/**/*.md
```

### 3. 上下文压缩策略

当上下文接近限制时，按优先级保留：

```yaml
context_compression:
  priority_order:
    - system_instructions  # 最高
    - current_task
    - relevant_files
    - architecture_rules
    - conversation_history  # 最低

  strategies:
    - name: "truncate_history"
      when: "tokens > 80%"
      action: "summarize_old_messages"

    - name: "drop_low_priority"
      when: "tokens > 90%"
      action: "remove_decorative_docs"

    - name: "smart_chunk"
      when: "file_too_large"
      action: "load_relevant_sections"
```

### 4. 工作内存预算

```
┌─────────────────────────────────────────────────────────────┐
│                   工作内存预算分配                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   总预算: 128K tokens                                        │
│                                                             │
│   ┌───────────────────────────────────────┐                │
│   │ 系统指令         │ ████████████████████ │ 20%           │
│   ├───────────────────────────────────────┤                │
│   │ 任务描述         │ ████████████████████ │ 15%           │
│   ├───────────────────────────────────────┤                │
│   │ 相关代码         │ ████████████████████ │ 40%           │
│   ├───────────────────────────────────────┤                │
│   │ 历史对话         │ ████████████████████ │ 15%           │
│   ├───────────────────────────────────────┤                │
│   │ 工具输出/中间结果 │ ████████████████████ │ 10%           │
│   └───────────────────────────────────────┘                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## AGENTS.md 最佳实践

### 结构模板

```markdown
# AGENTS.md

## 项目概述
[简短描述项目的目的和功能]

## 技术栈
- **语言**: [列表]
- **框架**: [列表]
- **主要库**: [列表]
- **构建工具**: [列表]

## 项目结构
```
src/
├── components/    # UI 组件
├── services/      # 业务逻辑
├── utils/         # 工具函数
└── types/        # 类型定义
```

## 架构约束

### 允许的依赖方向
- `components` → `services` → `utils`
- 禁止反向依赖

### 关键规则
- 所有 API 调用必须通过 `services/api.ts`
- 组件必须是函数式组件
- 使用 CSS Modules 进行样式隔离

## 代码规范

### 命名
- 组件: PascalCase
- 函数: camelCase
- 常量: SCREAMING_SNAKE_CASE

### 必须
- Props 必须有类型定义
- 错误必须被捕获和处理
- 所有副作用必须注明

### 禁止
- 禁止使用 `any` 类型
- 禁止直接操作 DOM
- 禁止在组件中直接使用 fetch

## 任务模板

### 功能开发
```markdown
## 功能: [名称]
### 需求
[描述]
### 验收标准
- [ ] [标准 1]
- [ ] [标准 2]
### 相关文件
- [文件路径]
```

## 工具使用规则

| 工具 | 用途 | 限制 |
|------|------|------|
| read_file | 读取代码 | 仅限 src/ 目录 |
| write_file | 写入代码 | 需要 backup |
| run_command | 执行命令 | 仅限 npm 脚本 |
| search_code | 搜索代码 | 无限制 |

## 常见问题

### Q: 如何添加新依赖?
A: 必须先在 `docs/dependency-policy.md` 备案

### Q: 组件太复杂怎么办?
A: 拆分为更小的子组件，单个组件不超过 200 行
```

## 上下文质量标准

### 高质量上下文特征

| 特征 | 描述 | 评估 |
|------|------|------|
| 相关性 | 内容与当前任务高度相关 | 相关文件 vs 噪声 |
| 完整性 | 包含完成任务所需的全部信息 | 缺失信息率 |
| 准确性 | 信息准确且最新 | 过期信息率 |
| 可操作性 | 信息可直接用于行动 | 行动成功率 |
| 简洁性 | 无冗余和不必要内容 | Token 效率 |

### 文档质量检查清单

- [ ] README.md 存在且最新
- [ ] 每个模块有文档
- [ ] API 有完整文档
- [ ] 重要决策有记录 (ADR)
- [ ] 依赖关系有图示
- [ ] 示例代码可运行

## 工具支持

### 上下文管理工具

```yaml
# tools/context-manager.yaml
tools:
  - name: "read_relevant_files"
    description: "读取与任务相关的文件"
    config:
      max_files: 10
      max_lines_per_file: 500

  - name: "search_context"
    description: "搜索相关上下文"
    config:
      search_depth: 2
      include_docs: true

  - name: "summarize_history"
    description: "总结对话历史"
    config:
      max_tokens: 1000
      keep_recent: 5
```

## 相关文档

- [HARNESS.md](../HARNESS.md) - 总体规范
- [AGENTS.md](../AGENTS.md) - Agent 配置
- [core-principles.md](./core-principles.md) - 核心原则
- [observability.md](./observability.md) - 可观测性
