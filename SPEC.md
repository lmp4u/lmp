# LMP Specification v1.0

## Abstract

The Language Model Prompt (LMP) format is a standardized file format for providing structured context to Large Language Models (LLMs). LMP files combine human-readable documentation with machine-parseable configuration to enable consistent, rich context generation across different projects, languages, and AI tools.

## 1. Introduction

### 1.1 Purpose

Large Language Models require context to effectively assist with code-related tasks. Currently, this context is provided ad-hoc through copy-pasting code snippets, README files, or manual explanations. LMP standardizes this process by defining a structured format that combines:

1. **Human documentation**: Natural language explanations of project architecture, conventions, and domain knowledge
2. **Machine configuration**: Precise specifications of which files to include, exclude patterns, and processing options

### 1.2 Goals

- **Universality**: Work with any LLM, programming language, and project structure
- **Simplicity**: Easy to create, read, and maintain by developers
- **Consistency**: Reproducible context generation across different environments
- **Extensibility**: Support for future enhancements while maintaining backward compatibility

### 1.3 Non-Goals

- Replace existing documentation formats (README, wikis, etc.)
- Provide runtime configuration for applications
- Store sensitive information (credentials, secrets, etc.)

## 2. File Format

### 2.1 File Extension

LMP files use the `.lmp` extension following the three-letter convention for file extensions.

### 2.2 File Structure

An LMP file consists of two optional sections:

1. **Documentation Section**: Free-form Markdown content
2. **Configuration Section**: Structured data in JSON, YAML, or TOML format

```
┌─────────────────────────────────────┐
│         Documentation Section       │
│         (Markdown, optional)        │
├─────────────────────────────────────┤
│         Configuration Section       │
│    (JSON/YAML/TOML, optional)      │
└─────────────────────────────────────┘
```

### 2.3 Section Delimiter

The configuration section is delimited using code fences that specify the format:

````markdown
# Documentation content here...

```json
{
  "configuration": "here"
}
```
````

Supported code fence languages:

- `json` - JSON configuration
- `yaml` or `yml` - YAML configuration
- `toml` - TOML configuration

### 2.4 Minimal Examples

**Documentation only:**

```markdown
# My Project

This project implements a task management system.
```

**Configuration only:**

```json
{
  "name": "My Project",
  "include": [{ "path": "./src/", "type": "dir" }]
}
```

**Combined:**

````markdown
# My Project

This project implements a task management system.

```json
{
  "name": "My Project",
  "include": [{ "path": "./src/", "type": "dir" }]
}
```
````

## 3. Configuration Schema

### 3.1 Root Object

```typescript
interface LMPConfig {
  name?: string; // Project name
  description?: string; // Brief description
  version?: string; // Project version

  include: IncludeEntry[]; // Files/directories to include
  exclude?: string[]; // Global exclusion patterns

  context?: ContextOptions; // Parser configuration
  conventions?: Record<string, string>; // Code conventions
  ai_instructions?: string; // AI-specific guidance
}
```

### 3.2 Include Entry

```typescript
interface IncludeEntry {
  path: string; // File or directory path (required)
  type: "file" | "dir"; // Entry type (required)

  // Directory-specific options
  recursive?: boolean; // Recurse into subdirectories (default: true)
  patterns?: string[]; // File patterns to include (default: ["*"])
  exclude?: string[]; // Local exclusion patterns
  max_depth?: number; // Maximum recursion depth

  // Metadata
  description?: string; // Human-readable description
  priority?: number; // Priority for token limits (1-10, default: 5)
}
```

### 3.3 Context Options

```typescript
interface ContextOptions {
  max_tokens?: number; // Token limit (default: 50000)
  merge_strategy?: "inherit" | "replace" | "append"; // Multi-file merge strategy
  output_format?: "markdown" | "json"; // Output format preference
}
```

### 3.4 Path Resolution

All paths in `include` entries are resolved relative to the directory containing the LMP file.

**Examples:**

- `./src/` - Relative to LMP file directory
- `../shared/` - Parent directory
- `/absolute/path` - Absolute path (discouraged for portability)

### 3.5 Pattern Matching

The `patterns` field supports glob-style patterns:

- `*` - Match any characters except path separator
- `**` - Match any characters including path separators (recursive)
- `?` - Match single character
- `[abc]` - Match any character in brackets
- `{a,b}` - Match any alternative in braces

**Examples:**

- `["*.js", "*.ts"]` - JavaScript and TypeScript files
- `["**/*.py"]` - All Python files recursively
- `["src/**/*.{js,ts}"]` - JS/TS files in src directory
- `["*.test.*"]` - Test files with any extension

### 3.6 Exclusion Patterns

Exclusions can be specified at two levels:

1. **Global exclusions** (`exclude` at root level): Apply to entire project
2. **Local exclusions** (`exclude` in include entry): Apply to specific include

Exclusion patterns use the same glob syntax as inclusion patterns.

**Common exclusions:**

```json
{
  "exclude": [
    "node_modules/",
    ".git/",
    "*.log",
    "dist/",
    "build/",
    "__pycache__/",
    "target/"
  ]
}
```

## 4. File Discovery

### 4.1 Hierarchical Discovery

LMP parsers discover `.lmp` files using hierarchical traversal:

1. **Upward traversal**: Walk up directory tree from target to find parent contexts
2. **Downward traversal**: Walk down from target to find child contexts
3. **Merge**: Combine contexts according to merge strategy

### 4.2 Processing Order

1. Root-level `.lmp` files (highest precedence)
2. Intermediate directory `.lmp` files
3. Target directory `.lmp` file (lowest precedence)

### 4.3 Merge Strategies

**inherit** (default): Child contexts inherit and extend parent contexts
**replace**: Child contexts completely replace parent contexts  
**append**: Child contexts append to parent contexts

## 5. Output Format

### 5.1 Standard Output Structure

```markdown
# LLM Context

## Metadata

- Generated: [ISO 8601 timestamp]
- Root: [root directory path]
- Files processed: [count]
- LMP files: [count]
- Token count: [estimated tokens]

## Project Documentation

[Merged markdown content from all .lmp files]

## Configuration Summary

[Summary of configuration from all .lmp files]

## Included Files

[Content of all included files with syntax highlighting]
```

### 5.2 File Content Format

Each included file is formatted as:

````markdown
### [relative path]

[optional description]

```[language]
[file content]
```
````

````

Language detection is based on file extension using common mappings.

### 5.3 Token Estimation

Parsers should provide token count estimates using the approximation:
- 1 token ≈ 4 characters for English text
- Adjust for specific model tokenizers when known

## 6. CLI Interface

### 6.1 Standard Commands

All implementations must support these commands:

```bash
lmp [path]                   # Generate context (default command)
lmp init [path]              # Initialize new .lmp file
lmp validate [file]          # Validate .lmp file syntax
lmp preview [path]           # Preview what would be included
````

### 6.2 Standard Options

```bash
--output, -o FILE            # Output to file instead of stdout
--verbose, -v                # Enable verbose logging
--quiet, -q                  # Suppress non-essential output
--max-tokens, -t NUMBER      # Override token limit
--format, -f FORMAT          # Output format (markdown, json)
--dry-run, -n                # Show actions without executing
```

### 6.3 Exit Codes

- `0` - Success
- `1` - General error (file not found, parse error, etc.)
- `2` - Invalid command line arguments
- `3` - Configuration error (invalid .lmp file)

## 7. Security Considerations

### 7.1 Path Traversal

Implementations must prevent path traversal attacks:

- Resolve all paths relative to LMP file directory
- Reject paths that escape project boundaries without explicit opt-in
- Validate all file access permissions

### 7.2 File Size Limits

Implementations should enforce reasonable limits:

- Maximum individual file size (e.g., 10MB)
- Maximum total context size
- Maximum number of included files

### 7.3 Sensitive Information

LMP files should not include:

- Credentials, API keys, passwords
- Personal identifiable information (PII)
- Internal network configurations
- Proprietary algorithms (unless intentionally shared)

## 8. Validation

### 8.1 JSON Schema

A formal JSON schema is provided at `spec/schema.json` for validation tools.

### 8.2 Validation Rules

**Required validations:**

- Configuration section must be valid JSON/YAML/TOML
- All required fields must be present
- Path references must be valid
- Pattern syntax must be valid glob patterns

**Recommended validations:**

- Warn on missing referenced files
- Warn on empty include sections
- Warn on potential token limit exceeded
- Warn on common misconfigurations

## 9. Extensibility

### 9.1 Reserved Fields

The following field names are reserved for future specification versions:

- `spec_version`
- `schema_url`
- `extensions`
- `metadata`

### 9.2 Custom Fields

Implementations may support custom fields using vendor prefixes:

- `x-vendor-field` - Vendor-specific extensions
- `_internal_field` - Implementation-internal fields

### 9.3 Version Compatibility

Future specification versions will maintain backward compatibility:

- New optional fields may be added
- Existing field semantics will not change
- Deprecated fields will be marked before removal

## 10. Examples

### 10.1 React TypeScript Project

````markdown
# E-commerce Frontend

Modern React application with TypeScript and Tailwind.

## Architecture

- State: Zustand for client state, React Query for server state
- Routing: React Router v6
- Styling: Tailwind CSS with custom design system
- Testing: Jest + React Testing Library

```json
{
  "name": "E-commerce Frontend",
  "version": "2.1.0",
  "include": [
    {
      "path": "./src/",
      "type": "dir",
      "patterns": ["*.tsx", "*.ts", "*.css"],
      "exclude": ["*.test.tsx", "*.stories.tsx"],
      "description": "Main application source code"
    },
    {
      "path": "./package.json",
      "type": "file",
      "description": "Dependencies and build scripts"
    },
    {
      "path": "./tailwind.config.js",
      "type": "file",
      "description": "Tailwind CSS configuration"
    }
  ],
  "exclude": ["node_modules/", ".next/", "dist/", "*.log"],
  "conventions": {
    "components": "Functional components with hooks, prefer composition",
    "styling": "Tailwind CSS classes, custom components in design system",
    "state": "Zustand stores for global state, local state for component-specific",
    "testing": "Test user interactions, not implementation details"
  },
  "ai_instructions": "Follow existing patterns for component structure. Use TypeScript strictly. Prefer React Query for server state management."
}
```
````

### 10.2 Python FastAPI Service

````markdown
# Task Management API

RESTful API built with FastAPI, PostgreSQL, and Redis.

## Features

- JWT authentication with refresh tokens
- Real-time WebSocket notifications
- Background task processing with Celery
- OpenAPI documentation

```yaml
name: "Task Management API"
description: "FastAPI backend with PostgreSQL and Redis"
include:
  - path: "./app/"
    type: "dir"
    patterns: ["*.py"]
    exclude: ["*_test.py", "test_*.py"]
    description: "Application source code"
  - path: "./requirements.txt"
    type: "file"
    description: "Python dependencies"
  - path: "./docker-compose.yml"
    type: "file"
    description: "Development environment setup"

exclude:
  - "__pycache__/"
  - ".pytest_cache/"
  - "*.pyc"
  - ".env"

conventions:
  code_style: "Black formatting, isort imports, type hints required"
  api_style: "RESTful endpoints with consistent error responses"
  database: "SQLAlchemy with Alembic migrations"
  testing: "Pytest with async test client"

ai_instructions: |
  Use async/await for all database operations. Follow FastAPI best practices 
  for dependency injection. Include proper error handling and validation.
```
````

### 10.3 Go CLI Application

````markdown
# Kubernetes Deployment Tool

Command-line tool for managing Kubernetes deployments with GitOps workflow.

```toml
name = "k8s-deploy"
description = "Kubernetes deployment automation tool"

[[include]]
path = "./cmd/"
type = "dir"
patterns = ["*.go"]
description = "CLI commands"

[[include]]
path = "./pkg/"
type = "dir"
patterns = ["*.go"]
exclude = ["*_test.go"]
description = "Core packages"

[[include]]
path = "./go.mod"
type = "file"

[[include]]
path = "./go.sum"
type = "file"

[context]
max_tokens = 75000

[conventions]
code_style = "gofmt, golint clean, effective Go principles"
cli_style = "Cobra framework with consistent flag naming"
error_handling = "Wrap errors with context, fail fast principle"
testing = "Table-driven tests, testify for assertions"
```
````

## 11. Implementation Notes

### 11.1 Performance Considerations

- **Lazy loading**: Only read file contents when explicitly included
- **Caching**: Cache file metadata to avoid repeated filesystem calls
- **Streaming**: Support streaming output for large contexts
- **Concurrency**: Parallelize file reading where appropriate

### 11.2 Error Handling

- **Graceful degradation**: Continue processing if individual files fail
- **Clear error messages**: Include file paths and line numbers in errors
- **Validation feedback**: Provide actionable feedback for configuration errors

### 11.3 Cross-Platform Compatibility

- **Path separators**: Handle both `/` and `\` path separators
- **File encoding**: Default to UTF-8, handle BOM markers
- **Line endings**: Normalize line endings in output
- **Case sensitivity**: Handle case-insensitive filesystems appropriately

## 12. Reference Implementation

The canonical reference implementation is provided in JavaScript/Node.js at `implementations/javascript/`. All other implementations should match its behavior for compatibility.

## 13. Conformance

An implementation conforms to this specification if it:

1. Supports all required configuration fields
2. Implements the standard CLI interface
3. Produces compatible output format
4. Passes the official test suite
5. Handles all specified error conditions

## 14. Changelog

### Version 1.0 (Initial Release)

- Core file format specification
- Configuration schema definition
- CLI interface standardization
- Security and validation requirements

---

This specification is released under CC0 (Public Domain) to encourage widespread adoption across tools and platforms.
