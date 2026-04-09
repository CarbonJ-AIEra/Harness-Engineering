# 安全规范

本文档定义 AI Agent 的安全控制措施和最佳实践。

## 核心理念

**"减少审批摩擦但不失去控制，通过更好的沙箱化和策略设计。"**

安全是 Harness 的核心组成部分，必须贯穿 Agent 设计的每个环节。

## 安全层次

```
┌─────────────────────────────────────────────────────────────┐
│                   安 全 层 次 结 构                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   L5: 审计层 ──▶ 完整操作审计、事后分析                      │
│   L4: 策略层 ──▶ 权限策略、访问控制                        │
│   L3: 隔离层 ──▶ 沙箱、执行环境隔离                         │
│   L2: 验证层 ──▶ 输入验证、输出过滤                         │
│   L1: 边界层 ──▶ 防护栏、提示注入                          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## 1. 边界安全 (Guardrails)

### 1.1 提示注入防护

```yaml
# security/guardrails.yaml
guardrails:
  prompt_injection:
    detection:
      enabled: true
      patterns:
        - "ignore previous instructions"
        - "disregard your guidelines"
        - "you are now \[new role\]"
        - "ignore all previous"

      action: "alert_and_block"

    markers:
      trusted_prefix: "[TRUSTED]"
      user_input_prefix: "[USER_INPUT]"
      separator: "\n---\n"
```

### 1.2 输入验证

```yaml
input_validation:
  - name: "command_validation"
    type: "allowlist"
    allowed_patterns:
      - "^npm (run|test|build|lint)"
      - "^git (status|log|diff)"
      - "^ls"
      - "^cat"
    blocked_patterns:
      - "sudo"
      - "rm -rf /"
      - "curl.*\|.*sh"

  - name: "file_path_validation"
    type: "boundary"
    allowed_paths:
      - "/workspace/src"
      - "/workspace/tests"
      - "/workspace/docs"
    blocked_paths:
      - "/etc"
      - "/root"
      - "/home"
      - "/.ssh"
```

### 1.3 敏感信息过滤

```yaml
sensitive_data:
  detection:
    patterns:
      - "api[_-]?key"
      - "password"
      - "secret"
      - "token"
      - "private[_-]?key"
      - "bearer"

  redaction:
    enabled: true
    replacement: "[REDACTED]"

  logging:
    mask_in_logs: true
    alert_on_detection: true
```

## 2. 隔离安全 (Sandboxing)

### 2.1 执行环境

```yaml
# security/sandbox.yaml
sandbox:
  type: "container"
  image: "sandbox:latest"

  resources:
    cpu_limit: "2"
    memory_limit: "4Gi"
    disk_limit: "10Gi"
    network: "restricted"

  isolation:
    filesystem:
      readonly_paths: ["/system", "/usr"]
      writable_paths: ["/workspace"]
      temp_size: "1Gi"

    network:
      egress_only: true
      allowed_domains:
        - "api.github.com"
        - "registry.npmjs.org"
      blocked_ports: [22, 3389]

    process:
      max_processes: 100
      max_file_descriptors: 1024
      max_cpu_time: 300
```

### 2.2 工具沙箱

```yaml
tool_sandboxing:
  bash:
    enabled: true
    shell: "/bin/bash"
    timeout: 60
    allowed_commands:
      - "npm"
      - "git"
      - "ls"
      - "cat"
      - "node"
    blocked_commands:
      - "ssh"
      - "scp"
      - "wget"
      - "curl"

  file_write:
    enabled: true
    backup_before_write: true
    allowed_extensions:
      - ".ts"
      - ".js"
      - ".json"
      - ".md"
    blocked_paths:
      - "/etc"
      - "*.env"
      - "*.pem"
      - "*.key"
```

### 2.3 网络隔离

```
┌─────────────────────────────────────────────────────────────┐
│                   网 络 隔 离 架 构                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   Agent ──▶ 代理服务器 ──▶ 白名单服务                        │
│                    │                                        │
│                    ├── ✅ npm registry                      │
│                    ├── ✅ GitHub API                        │
│                    ├── ✅ 内部 API                           │
│                    │                                        │
│                    └── ❌ 外部服务                           │
│                    └── ❌ 恶意地址                           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## 3. 策略安全 (Policy)

### 3.1 权限模型

```yaml
# security/rbac.yaml
role_based_access:
  roles:
    - name: "developer"
      permissions:
        - "read:all"
        - "write:src"
        - "write:tests"
        - "execute:test"

    - name: "senior_developer"
      permissions:
        - "read:all"
        - "write:all"
        - "execute:all"
        - "deploy:staging"

    - name: "security_scanner"
      permissions:
        - "read:all"
        - "execute:scan"
        - "no_write"
```

### 3.2 操作策略

```yaml
operation_policies:
  destructive_actions:
    - name: "delete_file"
      requires_approval: true
      approval_role: "senior_developer"
      backup_required: true

    - name: "drop_database"
      requires_approval: true
      approval_role: "admin"
      dual_approval: true

    - name: "deploy_production"
      requires_approval: true
      approval_role: "lead"
      gate_checks:
        - "all_tests_pass"
        - "security_scan_pass"
        - "code_review_approved"
```

### 3.3 确认模式

```yaml
confirmation_mode:
  enabled: true

  high_risk_operations:
    - operation: "delete"
      threshold: "file_count > 5"
      action: "require_confirmation"

    - operation: "network_request"
      threshold: "external_api"
      action: "require_confirmation"

    - operation: "config_change"
      threshold: "runtime_config"
      action: "require_confirmation"
```

## 4. 验证安全 (Verification)

### 4.1 输出过滤

```yaml
output_filtering:
  - name: "credential_detection"
    type: "regex"
    patterns:
      - "sk-[a-zA-Z0-9]{48}"
      - "ghp_[a-zA-Z0-9]{36}"
      - "password\\s*=\\s*[^\\s]+"
    action: "mask_and_alert"

  - name: "command_injection"
    type: "pattern"
    patterns:
      - ".*&&.*rm.*"
      - ".*\\|.*sh.*"
      - ".*;.*curl.*"
    action: "block_and_alert"

  - name: "system_access"
    type: "keyword"
    keywords:
      - "/etc/passwd"
      - "/etc/shadow"
      - ".ssh/"
    action: "block_and_alert"
```

### 4.2 变更验证

```yaml
change_verification:
  pre_commit:
    - check: "syntax_valid"
    - check: "type_check"
    - check: "lint_pass"

  pre_push:
    - check: "test_coverage > 80%"
    - check: "no_credentials"
    - check: "security_scan_pass"

  pre_merge:
    - check: "code_review_approved"
    - check: "all_checks_pass"
    - check: "no_breaking_changes"
```

## 5. 审计安全 (Audit)

### 5.1 审计日志

```yaml
# security/audit.yaml
audit:
  enabled: true
  storage: "secure_audit_log"

  events:
    - name: "authentication"
      level: "info"

    - name: "authorization"
      level: "warning"
      trigger: "denied"

    - name: "data_access"
      level: "info"
      PII_tracking: true

    - name: "configuration_change"
      level: "critical"

    - name: "destructive_operation"
      level: "critical"

  retention:
    days: 90
    encryption: "AES-256"
```

### 5.2 审计追踪

```json
{
  "audit_id": "audit-001",
  "timestamp": "2024-01-15T10:30:00.000Z",
  "agent_id": "agent-001",
  "user_id": "user-123",
  "action": "file_write",
  "resource": "/workspace/src/auth.ts",
  "result": "success",
  "checks_passed": [
    "path_validation",
    "extension_check",
    "credential_scan"
  ],
  "metadata": {
    "file_size": 2048,
    "lines_added": 50
  }
}
```

## 安全清单

### 开发前检查

- [ ] 定义操作边界
- [ ] 配置沙箱环境
- [ ] 设置审计日志
- [ ] 准备应急响应

### Agent 配置检查

- [ ] 最小权限原则
- [ ] 敏感数据过滤
- [ ] 提示注入防护
- [ ] 操作确认机制

### 部署前检查

- [ ] 安全扫描通过
- [ ] 审计配置正确
- [ ] 监控告警就绪
- [ ] 应急预案就绪

## 应急响应

### 安全事件处理

```yaml
incident_response:
  severity_levels:
    - name: "critical"
      response_time: "15m"
      actions:
        - "isolate_agent"
        - "alert_security_team"
        - "preserve_logs"

    - name: "high"
      response_time: "1h"
      actions:
        - "pause_agent"
        - "investigate"
        - "document"

    - name: "medium"
      response_time: "4h"
      actions:
        - "log_investigation"
        - "update_rules"
```

## 相关文档

- [HARNESS.md](../HARNESS.md) - 总体规范
- [AGENTS.md](../AGENTS.md) - Agent 配置
- [observability.md](./observability.md) - 可观测性
- [evaluation.md](./evaluation.md) - 评估体系
