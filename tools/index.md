# 工具定义

本文档定义 Agent 可用的标准工具集。

## 工具分类

```
┌─────────────────────────────────────────────────────────────┐
│                   工 具 分 类 体 系                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   📁 文件操作        📝 代码编辑        🔧 构建工具          │
│   ├── read_file     ├── edit_file      ├── run_test        │
│   ├── write_file    ├── create_file     ├── run_build       │
│   ├── list_dir      ├── delete_file     ├── run_lint        │
│   └── search        ├── move_file       └── run_format      │
│                                                             │
│   📦 包管理          🔍 代码分析        🌐 网络操作          │
│   ├── install_dep   ├── search_code     ├── http_get        │
│   ├── update_dep    ├── find_refs       └── http_post       │
│   └── list_deps     └── analyze         └── git_clone       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## 核心工具

### 1. 文件操作

#### read_file

读取单个文件的内容。

```yaml
tools:
  read_file:
    name: "read_file"
    description: "读取文件内容"
    category: "file"

    parameters:
      properties:
        path:
          type: string
          description: "文件路径"
          required: true
        line_start:
          type: integer
          description: "起始行号"
          default: 1
        line_end:
          type: integer
          description: "结束行号"
          default: null

    returns:
      content:
        type: string
        description: "文件内容"
      lines:
        type: integer
        description: "总行数"

    examples:
      - args: { path: "src/app.ts" }
        description: "读取整个文件"

      - args: { path: "src/app.ts", line_start: 1, line_end: 50 }
        description: "读取前 50 行"
```

#### write_file

创建或覆盖文件。

```yaml
tools:
  write_file:
    name: "write_file"
    description: "写入文件内容"
    category: "file"

    parameters:
      properties:
        path:
          type: string
          description: "文件路径"
          required: true
        content:
          type: string
          description: "文件内容"
          required: true
        create_backup:
          type: boolean
          description: "创建备份"
          default: true

    safety:
      - "backup_before_write"
      - "validate_extension"
      - "check_disk_space"

    returns:
      success:
        type: boolean
      file_path:
        type: string
```

#### edit_file

对文件进行精确编辑。

```yaml
tools:
  edit_file:
    name: "edit_file"
    description: "编辑文件内容"
    category: "file"

    parameters:
      properties:
        path:
          type: string
          description: "文件路径"
          required: true
        old_string:
          type: string
          description: "要替换的内容"
          required: true
        new_string:
          type: string
          description: "替换后的内容"
          required: true

    returns:
      success:
        type: boolean
      changes:
        type: object
        description: "变更详情"
```

#### search_code

搜索代码内容。

```yaml
tools:
  search_code:
    name: "search_code"
    description: "搜索代码中的文本"
    category: "analysis"

    parameters:
      properties:
        query:
          type: string
          description: "搜索关键词"
          required: true
        path:
          type: string
          description: "搜索路径"
          default: "src"
        extension:
          type: string
          description: "文件扩展名过滤"
          default: null
        case_sensitive:
          type: boolean
          description: "区分大小写"
          default: true

    returns:
      matches:
        type: array
        items:
          file: string
          line: integer
          content: string
```

### 2. 构建工具

#### run_command

执行 shell 命令。

```yaml
tools:
  run_command:
    name: "run_command"
    description: "执行 shell 命令"
    category: "execution"

    parameters:
      properties:
        command:
          type: string
          description: "要执行的命令"
          required: true
        timeout:
          type: integer
          description: "超时时间(秒)"
          default: 60
        working_dir:
          type: string
          description: "工作目录"
          default: "."

    safety:
      - "timeout_protection"
      - "command_whitelist"
      - "resource_limits"

    returns:
      exit_code: integer
      stdout: string
      stderr: string
```

#### run_test

运行测试套件。

```yaml
tools:
  run_test:
    name: "run_test"
    description: "运行测试"
    category: "execution"

    parameters:
      properties:
        pattern:
          type: string
          description: "测试文件匹配模式"
          default: "**/*.spec.ts"
        coverage:
          type: boolean
          description: "生成覆盖率报告"
          default: false
        watch:
          type: boolean
          description: "监视模式"
          default: false

    returns:
      passed: integer
      failed: integer
      coverage: object
```

### 3. Git 操作

```yaml
tools:
  git_status:
    name: "git_status"
    description: "查看 Git 状态"

  git_diff:
    name: "git_diff"
    description: "查看变更"
    parameters:
      properties:
        path:
          type: string
          description: "文件路径"

  git_log:
    name: "git_log"
    description: "查看提交历史"
    parameters:
      properties:
        limit:
          type: integer
          default: 10

  git_branch:
    name: "git_branch"
    description: "操作分支"
```

### 4. 代码分析

```yaml
tools:
  list_imports:
    name: "list_imports"
    description: "列出文件的所有导入"

  find_definitions:
    name: "find_definitions"
    description: "查找符号定义"
    parameters:
      properties:
        symbol:
          type: string
          required: true

  find_references:
    name: "find_references"
    description: "查找符号引用"
    parameters:
      properties:
        symbol:
          type: string
          required: true

  analyze_complexity:
    name: "analyze_complexity"
    description: "分析代码复杂度"
```

## 工具组合模式

### 读取-修改-验证

```yaml
patterns:
  read_modify_verify:
    description: "读取-修改-验证模式"

    steps:
      - tool: "read_file"
        description: "读取源文件"

      - tool: "edit_file"
        description: "修改内容"

      - tool: "run_test"
        description: "运行测试验证"

    atomic: false
    rollback_on_failure: true
```

### 搜索-读取-修改

```yaml
patterns:
  search_read_modify:
    description: "搜索-读取-修改模式"

    steps:
      - tool: "search_code"
        description: "定位要修改的代码"

      - tool: "read_file"
        description: "读取相关文件"

      - tool: "edit_file"
        description: "执行修改"

    parallel: true
```

## 工具配置

```yaml
# configs/tools.yaml
tools:
  enabled:
    - read_file
    - write_file
    - edit_file
    - search_code
    - run_command
    - run_test
    - run_build

  disabled:
    - delete_file
    - network_request

  configuration:
    read_file:
      max_file_size: "1MB"
      max_lines: 10000

    write_file:
      allowed_extensions:
        - ".ts"
        - ".js"
        - ".json"
        - ".md"
        - ".css"
        - ".html"

    run_command:
      allowed_commands:
        - "npm *"
        - "git *"
        - "node *"
        - "ls"
        - "cat"
      blocked_commands:
        - "rm -rf"
        - "sudo"
        - "ssh"
```

## 相关文档

- [AGENTS.md](../AGENTS.md) - Agent 配置
- [docs/context-engineering.md](../docs/context-engineering.md) - 上下文工程
- [docs/security.md](../docs/security.md) - 安全规范
