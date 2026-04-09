# Harness Engineering 规范体系

> **核心信念**: 模型是商品，Harness 是护城河。

基于 OpenAI [Harness Engineering](https://openai.com/index/harness-engineering/) 理念构建的 AI Agent 工程规范体系。

## 概述

Harness Engineering 是一种新兴的工程学科，专注于设计和管理 AI Agent 的控制系统。根据 LangChain 的定义：**Agent = Model + Harness**

它涵盖了上下文工程、评估、可观测性、编排、安全自治和软件架构的交叉领域。

## 核心原则

| 原则 | 描述 |
|------|------|
| **双向控制闭环** | 同时部署引导指令和检测传感器 |
| **混合执行模式** | 结合计算型控制（快速、确定）和推断型控制（语义理解） |
| **质量前置策略** | 验证按成本与时效分层，越早发现越好 |
| **多样性约束定律** | 通过固化标准架构压缩 Agent 可能生成的代码范围 |

## 目录结构

```
Harness Engineering/
├── HARNESS.md                    # 总体规范
├── AGENTS.md                     # Agent 配置和使用指南
├── ARCHITECTURE.md               # 架构约束定义
│
├── docs/                         # 详细指南
│   ├── core-principles.md       # 核心原则详解
│   ├── context-engineering.md   # 上下文工程实践
│   ├── evaluation.md            # 评估体系
│   ├── observability.md         # 可观测性设计
│   ├── orchestration.md         # 编排策略
│   ├── security.md              # 安全规范
│   └── best-practices.md       # 最佳实践
│
├── configs/                      # 配置文件
│   ├── agent-config.yaml        # Agent 配置
│   ├── constraints.yaml         # 约束规则
│   ├── context-injection.yaml   # 上下文注入
│   ├── logging.yaml             # 日志配置
│   └── observability.yaml       # 可观测性配置
│
├── tools/                        # 工具定义
│   └── index.md                 # 标准工具集
│
├── templates/                    # 模板
│   ├── agent-prompt.md          # Agent 提示词模板
│   └── task-spec.md             # 任务规范模板
│
└── evals/                        # 评估相关
    └── README.md                # 评估指南
```

## 快速开始

### 1. 创建 AGENTS.md

在项目根目录创建 `AGENTS.md`，定义 Agent 的行为约束：

```markdown
# AGENTS.md

## 项目概述
[项目简介]

## 技术栈
- 语言: TypeScript
- 框架: React, Node.js

## 架构约束
[架构规则]

## 代码规范
[编码标准]
```

### 2. 配置 Agent

编辑 `configs/agent-config.yaml`：

```yaml
agents:
  coding:
    model: "gpt-4o"
    temperature: 0.2
    tools:
      enabled:
        - read_file
        - write_file
        - run_command
        - run_test
```

### 3. 设置约束

在 `configs/constraints.yaml` 中定义架构约束：

```yaml
architecture:
  layers:
    - name: "presentation"
      can_access: ["application"]
    - name: "application"
      can_access: ["domain", "infrastructure"]
```

## 三大支柱

### 1. Context Engineering (上下文工程)

确保 Agent 获得正确的信息：
- 仓库是单一事实来源
- 管理上下文窗口作为工作内存预算
- 创建持久化的仓库本地指令

### 2. Architectural Constraints (架构约束)

使用确定性工具强制执行代码结构：
- Linter 规则
- CI 验证
- 模块边界检查

### 3. Entropy Management (熵管理)

部署调度任务来：
- 清理死代码
- 修复文档漂移
- 维护依赖健康

## 评估基准

| 领域 | 基准 | 描述 |
|------|------|------|
| 软件工程 | SWE-bench | 真实 GitHub Issue 修复 |
| 终端任务 | Terminal-Bench | CLI 操作 |
| Web 自动化 | WebArena | 浏览器任务 |
| 工具使用 | MCP Bench | MCP 工具使用 |

## 安全控制

- **边界安全**: 提示注入防护、输入验证
- **隔离安全**: 沙箱执行、网络隔离
- **策略安全**: RBAC 权限、操作审批
- **审计安全**: 完整操作审计、事后分析

## 相关资源

- [OpenAI Harness Engineering](https://openai.com/index/harness-engineering/)
- [Martin Fowler - Harness Engineering](https://martinfowler.com/articles/harness-engineering.html)
- [Awesome Harness Engineering](https://github.com/walkinglabs/awesome-harness-engineering)

## 许可证

MIT
