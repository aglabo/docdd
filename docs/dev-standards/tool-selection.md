---
title: "Tool Selection Guide"
description: "Guide for selecting the right tool for each development task in deckrd project"
category: "dev-standards"
tags: ["tools", "mcp", "selection-guide"]
created: "2026-01-14"
version: "0.0.4"
authors:
  - atsushifx <https://github.com/atsushifx>
changes:
  - 0.0.4   2026-01-14  初版作成
copyright:
  - Copyright (c) 2026- atsushifx <https://github.com/atsushifx>
  - This software is released under the MIT License.
  - https://opensource.org/licenses/MIT
status: "published"
---

<!-- textlint-disable ja-technical-writing/sentence-length -->
<!-- markdownlint-disable line-length -->

## Tool Selection Guide

## Overview

The deckrd project uses multiple tools and MCP servers for different tasks. This guide helps you choose the right tool for each situation.

## Quick Reference

| Task                        | Tool                            | Why                        |
| --------------------------- | ------------------------------- | -------------------------- |
| Analyze bash scripts        | serena-mcp                      | Semantic analysis for bash |
| Search code patterns        | serena-mcp `search_for_pattern` | Fast regex search          |
| Find bash functions         | serena-mcp `find_symbol`        | Symbol-aware search        |
| Read documentation          | Read tool                       | Direct file access         |
| Search markdown files       | Grep tool                       | Quick keyword search       |
| List directory contents     | serena-mcp `list_dir`           | Respects gitignore         |
| Find files by pattern       | serena-mcp `find_file`          | Wildcard matching          |
| Analyze TypeScript (future) | lsmcp                           | LSP integration            |
| Generate code/templates     | codex-mcp                       | AI-powered generation      |
| Format code                 | dprint                          | Consistent formatting      |
| Check commit messages       | commitlint                      | Conventional Commits       |
| Detect secrets              | gitleaks                        | Security scanning          |

## For Bash Script Analysis

### Use serena-mcp

**When**:

- Analyzing bash scripts
- Finding functions/symbols
- Understanding code structure
- Searching code patterns

**Tools**:

#### get_symbols_overview

**Use for**: Getting high-level overview of a bash script

```bash
serena-mcp get_symbols_overview --relative-path "scripts/init.sh"
```

**Returns**: List of functions and symbols

#### find_symbol

**Use for**: Finding specific functions or symbols

```bash
serena-mcp find_symbol --name-path-pattern "main" --include-body true
```

**Returns**: Symbol definition with body

#### search_for_pattern

**Use for**: Searching for specific code patterns

```bash
serena-mcp search_for_pattern --substring-pattern "set -euo pipefail"
```

**Returns**: Matches with context

#### find_referencing_symbols

**Use for**: Understanding how a function is used

```bash
serena-mcp find_referencing_symbols --name-path "main" --relative-path "init.sh"
```

**Returns**: All references to the symbol

## For Documentation

### Use Read Tool

**When**:

- Reading specific files
- Known file paths
- Need exact content

**Examples**:

```bash
# Read entire file
Read README.md

# Read specific range
Read setup.md --offset 10 --limit 50
```

### Use Grep Tool

**When**:

- Quick keyword search
- Multiple files
- Pattern matching

**Examples**:

```bash
# Search for keyword
Grep "installation" --path docs/

# With context
Grep "error" --path logs/ -C 3
```

### Use serena-mcp search_for_pattern

**When**:

- Complex regex patterns
- Need structured results
- Searching code and docs

**Example**:

```bash
serena-mcp search_for_pattern --substring-pattern "## Overview" \
  --paths-include-glob "**/*.md"
```

## For Configuration Files

### YAML/TOML/JSON

**Option 1: Read Tool** (Simple)

```bash
# Direct read
Read .mcp.json
Read lefthook.yml
```

**Option 2: serena-mcp** (Complex)

```bash
# Pattern matching in config
serena-mcp search_for_pattern --substring-pattern "serena-mcp" \
  --relative-path ".mcp.json"
```

## For TypeScript/JavaScript (Future)

### Use lsmcp

**When**:

- TypeScript project added
- Need IDE-like features
- Refactoring TypeScript code

**Tools**:

#### search_symbols

**Use for**: Finding classes, interfaces, functions

```bash
lsmcp search_symbols --query "MyClass" --kind "Class"
```

#### lsp_get_definitions

**Use for**: Jump to definition

```bash
lsmcp lsp_get_definitions --root /path/to/project \
  --relativePath "src/index.ts" --line 10 --symbolName "MyClass"
```

#### lsp_rename_symbol

**Use for**: Refactoring

```bash
lsmcp lsp_rename_symbol --root /path/to/project \
  --relativePath "src/index.ts" --textTarget "OldName" --newName "NewName"
```

#### lsp_get_diagnostics

**Use for**: Finding errors and warnings

```bash
lsmcp lsp_get_diagnostics --root /path/to/project \
  --relativePath "src/index.ts"
```

## For Code Generation

### Use codex-mcp

**When**:

- Need AI-powered generation
- Template processing
- Code suggestions

**Use Cases**:

- Generate boilerplate code
- Fill in templates
- Create documentation from code
- Suggest improvements

## For Command/Plugin Search

### deckrd Commands

**Location**: `plugins/deckrd/skills/deckrd/`

**Tools**:

```bash
# Option 1: serena-mcp
serena-mcp list_dir --relative-path "plugins/deckrd/skills/deckrd/scripts"

# Option 2: serena-mcp search
serena-mcp search_for_pattern --substring-pattern "function main" \
  --relative-path "plugins/deckrd/skills/deckrd/scripts"

# Option 3: Grep
Grep "deckrd init" --path plugins/deckrd/
```

### IDD Framework Commands (External)

**Location**: `C:\Users\atsushifx\.claude\plugins\marketplaces\claude-idd-framework-marketplace\plugins\claude-idd-framework`

**Important**: Always search in this external path for IDD commands

**Tools**:

```bash
# Option 1: lsmcp (for external plugin directory navigation)
lsmcp list_dir --relativePath "." --recursive false

# Option 2: serena-mcp (if analyzing bash scripts)
serena-mcp find_symbol --name-path-pattern "issue"

# Option 3: Grep (for quick search)
Grep "/idd/issue" --path ~/.claude/plugins/marketplaces/
```

## Decision Tree

### For Code Analysis

```text
Is it bash script?
  ├─ Yes → serena-mcp
  │   ├─ Need overview? → get_symbols_overview
  │   ├─ Find function? → find_symbol
  │   ├─ Search pattern? → search_for_pattern
  │   └─ Find usages? → find_referencing_symbols
  │
  └─ No → Is it TypeScript?
      ├─ Yes → lsmcp
      │   ├─ Find symbol? → search_symbols
      │   ├─ Get definition? → lsp_get_definitions
      │   └─ Check errors? → lsp_get_diagnostics
      │
      └─ No → Read or Grep
```

### For File Operations

```text
What do you need?
  ├─ Specific file? → Read tool
  ├─ Find files by pattern? → serena-mcp find_file or Glob
  ├─ List directory? → serena-mcp list_dir
  ├─ Search content? → Grep or serena-mcp search_for_pattern
  └─ Generate code? → codex-mcp
```

### For Project Understanding

```text
What do you want to know?
  ├─ Project structure? → serena-mcp list_dir (recursive)
  ├─ Available commands? → Search plugin directories
  ├─ Configuration? → Read .mcp.json, lefthook.yml
  ├─ Documentation? → Read docs/*.md
  └─ Code patterns? → serena-mcp search_for_pattern
```

## Performance Considerations

### Token Usage

**Minimize**:

1. Use serena-mcp memories (cached context)
2. Prefer symbolic search over full reads
3. Use `relative_path` to restrict scope
4. Read only what you need

**Example** (Good):

```bash
# Efficient: Get overview first
serena-mcp get_symbols_overview --relative-path "scripts/init.sh"
# Then: Read specific function
serena-mcp find_symbol --name-path-pattern "main" --include-body true
```

**Example** (Bad):

```bash
# Inefficient: Read entire file
Read scripts/init.sh
# Then: Search within memory
```

### Execution Speed

**Fast**:

- Grep (simple pattern)
- Read (known path)
- serena-mcp find_symbol (indexed)

**Medium**:

- serena-mcp search_for_pattern (regex)
- lsmcp search_symbols (indexed)

**Slow**:

- serena-mcp with large scope
- lsmcp full workspace analysis
- codex-mcp generation

**Optimization**:

1. Narrow search scope with `relative_path`
2. Use specific patterns, not wildcards
3. Cache results in memories
4. Parallel queries when independent

## Common Scenarios

### Scenario 1: Understanding a New Bash Script

```bash
# Step 1: Get overview
serena-mcp get_symbols_overview --relative-path "scripts/new-script.sh"

# Step 2: Read main function
serena-mcp find_symbol --name-path-pattern "main" \
  --relative-path "scripts/new-script.sh" --include-body true

# Step 3: Find references if needed
serena-mcp find_referencing_symbols --name-path "helper_function" \
  --relative-path "scripts/new-script.sh"
```

### Scenario 2: Finding Command Implementation

```bash
# Step 1: Find command file
serena-mcp find_file --file-mask "*init*" \
  --relative-path "plugins/deckrd/skills/deckrd/scripts"

# Step 2: Get overview
serena-mcp get_symbols_overview \
  --relative-path "plugins/deckrd/skills/deckrd/scripts/init.sh"

# Step 3: Read specific function
serena-mcp find_symbol --name-path-pattern "initialize_session" \
  --include-body true
```

### Scenario 3: Searching Documentation

```bash
# Step 1: Find relevant docs
Grep "MCP servers" --path docs/ --output-mode files_with_matches

# Step 2: Read specific doc
Read docs/dev-standards/mcp-servers.md

# Step 3: Search for specific section
Grep "serena-mcp" --path docs/dev-standards/mcp-servers.md -C 5
```

### Scenario 4: Analyzing External Plugin

```bash
# Important: Use absolute path for external plugins
EXTERNAL_PLUGIN="C:\Users\atsushifx\.claude\plugins\marketplaces\claude-idd-framework-marketplace\plugins\claude-idd-framework"

# Step 1: List structure (use lsmcp for external paths)
lsmcp list_dir --relativePath "commands" --recursive false

# Step 2: Search for command
Grep "/idd/issue:new" --path $EXTERNAL_PLUGIN

# Step 3: Read implementation
Read $EXTERNAL_PLUGIN/commands/idd/issue/new.sh
```

## Anti-Patterns

### ❌ Don't Do

1. **Reading entire files when you need one function**

   ```bash
   # Bad
   Read scripts/init.sh
   # Find main function manually
   ```

2. **Using bash commands instead of tools**

   ```bash
   # Bad
   Bash cat scripts/init.sh
   Bash find . -name "*.sh"
   ```

3. **Not restricting search scope**

   ```bash
   # Bad: Searches entire project
   serena-mcp search_for_pattern --substring-pattern "function"
   ```

4. **Ignoring MCP memories**

   ```bash
   # Bad: Re-reading same context
   Read README.md  # Every time
   ```

### ✅ Do Instead

1. **Use symbolic search**

   ```bash
   # Good
   serena-mcp find_symbol --name-path-pattern "main" --include-body true
   ```

2. **Use dedicated tools**

   ```bash
   # Good
   Read scripts/init.sh
   Glob "**/*.sh"
   ```

3. **Restrict scope**

   ```bash
   # Good
   serena-mcp search_for_pattern --substring-pattern "function" \
     --relative-path "scripts/"
   ```

4. **Use memories**

   ```bash
   # Good
   serena-mcp read_memory --memory-file-name "project_overview"
   ```

## Related Documentation

- [MCP Servers API Reference](../dev-api/mcp-servers.md) - Detailed API docs
- [MCP Server Configuration](mcp-servers.md) - Setup and configuration
- [Architecture](../dev-architecture/architecture.md) - System design
- [Development Workflow](../dev-guides/workflow.md) - Workflow integration
