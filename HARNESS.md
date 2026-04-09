# Harness Engineering 规范

> **核心信念**: 模型是商品，Harness 是护城河。

## 概述

Harness Engineering 是一种新兴的工程学科，专注于设计和管理 AI Agent 的控制系统。根据 LangChain 的定义：**Agent = Model + Harness**。它涵盖了上下文工程、评估、可观测性、编排、安全自治和软件架构的交叉领域。

**目标**: 塑造环境，使 AI Agent 在长期运行的编码/研究任务中可靠工作。

**关键洞察**: 编码 Agent 表现不佳通常是 Harness 问题，而非模型问题。

## 核心原则

### 1. 双向控制闭环

系统必须同时部署：
- **引导指令** (行动前干预): 通过规范、模板、约束进行前馈控制
- **检测传感器** (行动后观察): 通过评估、可观测性、反馈机制进行后馈控制

缺少任一环节都会导致 Agent 重复犯错或盲目执行。

### 2. 混合执行模式

| 类型 | 特点 | 示例 |
|------|------|------|
| 计算型控制 | 快速、确定性 | 类型检查、Linter、静态分析 |
| 推断型控制 | 成本较高、语义判断 | AI 审查、代码评审 |

### 3. 质量前置策略

验证步骤按成本与时效分层：
- **高频快检**: 代码提交前触发（pre-commit hooks）
- **深度评估**: 集成流水线（CI/CD）
- **持续监控**: 技术债务扫描

### 4. 多样性约束定律

依据控制论原理，调节器需具备对应系统的复杂度。通过固化标准架构拓扑来"压缩 Agent 可能生成的代码范围，让全面监管变得切实可行"。

## 三大支柱

### 1. Context Engineering (上下文工程)

确保 Agent 获得正确的信息：
- 仓库必须是单一事实来源
- 管理上下文窗口作为"工作内存预算"
- 创建持久化、仓库本地的指令（如 `AGENTS.md`）

### 2. Architectural Constraints (架构约束)

使用确定性 Linter 和 CI 验证来机械地强制执行代码结构：
- 定义清晰的模块边界
- 设置性能阈值
- 建立可观测性标准

### 3. Entropy Management (熵管理)

部署调度 Agent 来：
- 清理死代码
- 修复文档漂移
- 维护依赖健康

## 调节维度

### 可维护性
- 内部代码健康度（重复率、圈复杂度、覆盖率）
- 现有静态分析工具最易接入

### 架构适应性
- 性能阈值
- 可观测性标准
- 模块边界规则

### 行为控制
- 功能与需求一致性验证
- 规范文档与自动化测试

## 实施路径

1. **起步**: 从简单设置开始（convention 文件、pre-commit hooks）
2. **演进**: 构建"可撕裂的 Harness"——避免过度工程化
3. **迭代**: 根据重复故障持续调优控制网

## 人类角色

开发者从手动编码转向环境设计：
- 需要系统思维
- 定义精确边界
- 分析 Agent 行为模式
- 验证输出质量

**核心理念**: 不是消除人工，而是"把专家经验显式化，将人类干预精准导向高价值环节"。

## 常见陷阱

- 创建静态系统，无法随模型更新而演进
- 忽视文档质量（直接影响 Agent 表现）
- 设计缺乏自验证反馈循环的工作流
- 将关键架构知识放在代码库外

## 文件结构

```
harness-engineering/
├── HARNESS.md                    # 本文件 - 总体规范
├── AGENTS.md                     # Agent 配置和使用指南
├── ARCHITECTURE.md               # 架构约束定义
├── docs/
│   ├── core-principles.md        # 核心原则详解
│   ├── context-engineering.md   # 上下文工程实践
│   ├── evaluation.md            # 评估体系
│   ├── observability.md         # 可观测性设计
│   ├── orchestration.md         # 编排策略
│   ├── security.md              # 安全规范
│   └── best-practices.md        # 最佳实践
├── tools/
│   └── index.md                 # 工具定义
├── templates/
│   ├── agent-prompt.md          # Agent 提示词模板
│   └── task-spec.md             # 任务规范模板
├── configs/
│   ├── agent-config.yaml        # Agent 配置
│   └── constraints.yaml         # 约束配置
└── evals/
    ├── README.md                # 评估指南
    └── cases/                   # 评估用例
```

## 相关资源

- [OpenAI Harness Engineering](https://openai.com/index/harness-engineering/)
- [Martin Fowler - Harness Engineering](https://martinfowler.com/articles/harness-engineering.html)
- [Awesome Harness Engineering](https://github.com/walkinglabs/awesome-harness-engineering)
