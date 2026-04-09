# 架构约束规范

本文档定义 AI Agent 必须遵守的架构约束规则。

## 核心理念

**多样性约束定律**: 通过固化标准架构拓扑来"压缩 Agent 可能生成的代码范围，让全面监管变得切实可行"。

架构约束使用确定性 Linter 和 CI 验证来机械地强制执行代码结构，而非依赖建议。

## 约束层次

### 1. 模块边界约束

```
┌─────────────────────────────────────────────────────┐
│                    模块边界                          │
├─────────────────────────────────────────────────────┤
│  ┌─────────┐    ┌─────────┐    ┌─────────┐         │
│  │ Module A │───▶│ Module B │───▶│ Module C │         │
│  └─────────┘    └────┬────┘    └─────────┘         │
│                     │                               │
│              允许的依赖方向                          │
└─────────────────────────────────────────────────────┘
```

**规则**:
- 禁止逆向依赖
- 禁止跨层直接调用
- 必须通过接口/抽象层通信

### 2. 依赖规则

```yaml
# configs/constraints.yaml
dependency_rules:
  allowed_layers:
    - presentation
    - application
    - domain
    - infrastructure

  forbidden_dependencies:
    - presentation → infrastructure  # 不允许直接依赖
    - domain → infrastructure       # 不允许直接依赖

  allowed_transitions:
    - presentation → application → domain → infrastructure
    - presentation → domain
```

### 3. 命名约定

| 类型 | 规则 | 示例 |
|------|------|------|
| 文件名 | kebab-case | `user-service.ts` |
| 类名 | PascalCase | `UserService` |
| 函数名 | camelCase | `getUserById` |
| 常量 | SCREAMING_SNAKE | `MAX_RETRY_COUNT` |
| 组件 | PascalCase | `UserCard.vue` |

## 架构模式

### 分层架构

```
┌────────────────────────────────────────┐
│           Presentation Layer            │
│    (UI Components, Controllers)        │
├────────────────────────────────────────┤
│           Application Layer             │
│      (Use Cases, Application Services) │
├────────────────────────────────────────┤
│              Domain Layer               │
│    (Entities, Value Objects, Domain    │
│              Services)                  │
├────────────────────────────────────────┤
│          Infrastructure Layer           │
│  (Repositories, External Services,     │
│            Database)                    │
└────────────────────────────────────────┘
```

### 整洁架构

```
┌─────────────────────────────────────────────┐
│           Frameworks & Drivers              │
│    (Web, UI, External Interfaces)          │
├─────────────────────────────────────────────┤
│              Interface Adapters            │
│      (Controllers, Gateways, Presenters)    │
├─────────────────────────────────────────────┤
│                Use Cases                   │
│         (Application Business Rules)        │
├─────────────────────────────────────────────┤
│                  Entities                   │
│        (Enterprise Business Rules)          │
└─────────────────────────────────────────────┘
```

### 插件架构

```
┌─────────────────────────────────────────────┐
│              Core System                    │
│    (Business Logic, Domain Models)          │
├─────────────────────────────────────────────┤
│           Plugin Interface                  │
│         (Well-defined Contracts)            │
├─────────────────────────────────────────────┤
│              Plugins                        │
│  ┌────────┐ ┌────────┐ ┌────────┐          │
│  │Plugin A│ │Plugin B│ │Plugin C│          │
│  └────────┘ └────────┘ └────────┘          │
└─────────────────────────────────────────────┘
```

## 强制工具

### Linter 规则

```yaml
# .eslintrc.yaml - 代码检查规则
rules:
  no-restricted-imports:
    - error
    - patterns:
        - "*/infrastructure/*"
        - "*/database/*"

  import/order:
    - error
    - groups:
        - external
        - internal
        - parent
        - sibling

  max-lines-per-function:
    - error
    - max: 100
    - skipBlankLines: true
```

### 模块化检查

```yaml
# dependency-cruiser.yaml
rules:
  no-orphans: true
  no-deprecated-imports: true
  no-implicit-or:
    - severity: error
      from: { path: "src/presentation" }
      to: { path: "src/infrastructure" }
```

### 圈复杂度限制

```yaml
# .pylintrc (Python)
max-complexity = 10

# tsconfig (TypeScript)
"complexity": 20
```

## 架构验证流程

### CI 集成

```yaml
# .github/workflows/architecture.yml
name: Architecture Check

on:
  pull_request:
    paths:
      - 'src/**'
      - 'lib/**'

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Dependency Cruiser
        run: npx depcruise src

      - name: ESLint
        run: npm run lint:architecture

      - name: Import Order
        run: npm run check:imports
```

### Pre-commit Hooks

```yaml
# .husky/pre-commit
- id: dependency-cruiser
- id: eslint
- id: check-imports
```

## 性能约束

### 文件大小限制

| 类型 | 最大行数 | 最大大小 |
|------|----------|----------|
| 单文件 | 500 | 50KB |
| 组件 | 300 | 30KB |
| 服务 | 200 | 20KB |
| 测试 | 300 | 30KB |

### 依赖限制

```yaml
max_dependencies:
  direct: 10
  transitive: 50
  peer: 5
```

## 文档约束

### 必需文档

- 每个模块必须有 README.md
- 每个公共 API 必须有 JSDoc/TSDoc
- 复杂逻辑必须内联注释
- 决策必须有 ADR (Architecture Decision Records)

### 文档格式

```typescript
/**
 * 获取用户详情
 *
 * @param id - 用户 ID
 * @returns 用户详情，包含基本信息、权限列表
 *
 * @throws {NotFoundError} 用户不存在
 * @throws {PermissionError} 无权限访问
 *
 * @example
 * ```typescript
 * const user = await getUserById('123');
 * console.log(user.name);
 * ```
 */
async function getUserById(id: string): Promise<User> {
  // ...
}
```

## 相关文档

- [HARNESS.md](./HARNESS.md) - 总体规范
- [AGENTS.md](./AGENTS.md) - Agent 配置
- [docs/core-principles.md](./docs/core-principles.md) - 核心原则
- [configs/constraints.yaml](./configs/constraints.yaml) - 约束配置
