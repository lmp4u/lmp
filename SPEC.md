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

### 2.1 File Extensions

LMP defines two file types:

- **`.lmp`** - Source files (human-authored, version controlled)
- **`.lmpc`** - Context files (machine-generated, usually git-ignored)

### 2.2 LMP Source File Structure

An LMP source file (`.lmp`) consists of two optional sections:

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

### 2.4 LMP Context File Structure

An LMP context file (`.lmpc`) contains the generated output ready for consumption by AI tools:

```markdown
# LLM Context

## Metadata

- Generated: [timestamp]
- Source: [.lmp files used]
- Token count: [estimate]

## Project Documentation

[Merged documentation from .lmp files]

## Included Files

[File contents with syntax highlighting]
```

### 2.5 Minimal Examples

**Documentation only (.lmp):**

```markdown
# My Project

This project implements a task management system.
```

**Configuration only (.lmp):**

```json
{
  "name": "My Project",
  "include": [{ "path": "./src/", "type": "dir" }]
}
```

**Combined (.lmp):**

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
  // Project metadata
  name?: string;
  description?: string;
  version?: string;

  // Core functionality
  include: IncludeEntry[];
  exclude?: string[];

  // Rich project context
  tech_stack?: Record<string, string>;
  conventions?: Record<string, string>;
  ai_context?: AIContext;

  // Parser configuration
  context?: ContextOptions;

  // Legacy/simple field
  ai_instructions?: string;
}

interface AIContext {
  focus_areas?: string[];
  domain_knowledge?: string[];
  avoid?: string[];
  patterns_to_follow?: string[];
  performance_considerations?: string[];
  security_notes?: string[];
}

interface ContextOptions {
  max_tokens?: number;
  merge_strategy?: "inherit" | "replace" | "append";
  output_format?: "markdown" | "json";
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

### 3.3 Path Resolution

All paths in `include` entries are resolved relative to the directory containing the LMP file.

**Examples:**

- `./src/` - Relative to LMP file directory
- `../shared/` - Parent directory
- `/absolute/path` - Absolute path (discouraged for portability)

### 3.4 Pattern Matching

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

### 3.5 Exclusion Patterns

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
    "target/",
    "*.lmpc"
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

### 4.4 Scope Control

The scope of context generation is determined by where the command is executed:

```bash
# Whole project context
cd /project-root && lmp .

# Component-specific context
cd /project-root/frontend && lmp .

# Module-specific context
cd /project-root/backend/auth && lmp .
```

Each execution finds all relevant `.lmp` files in the hierarchy and generates context appropriate to that scope.

## 5. Output Format

### 5.1 Standard Output Structure

Generated `.lmpc` files follow this structure:

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
```

### 6.2 Standard Options

```bash
--output, -o FILE            # Output to specific file
--save, -s                   # Auto-generate filename: <directory>.lmpc
--verbose, -v                # Enable verbose logging
--quiet, -q                  # Suppress non-essential output
--max-tokens, -t NUMBER      # Override token limit
--format, -f FORMAT          # Output format (markdown, json)
--dry-run, -n                # Show actions without executing
```

### 6.3 Output Behavior

**Default (no output flags)**: Output to stdout

```bash
lmp .                        # Print context to terminal
lmp . | pbcopy              # Pipe to clipboard
```

**Explicit file output**:

```bash
lmp . --output context.lmpc  # Save to specific file
lmp . -o my-context.lmpc    # Short form
```

**Auto-generated filename**:

```bash
lmp . --save                 # Creates: <current-directory-name>.lmpc
cd my-project && lmp . --save # Creates: my-project.lmpc
```

### 6.4 Exit Codes

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

**Recommendation**: Add `*.lmpc` to `.gitignore` to prevent accidental commit of generated context files.

## 8. Validation

### 8.1 Validation Rules

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

### 8.2 Common Issues

**Auto-fixable issues**:

- Missing trailing slashes on directory paths
- Common typos in field names
- Invalid glob patterns

**Warning conditions**:

- Large number of included files
- Very large individual files
- Potential token limit exceeded

## 9. Best Practices

### 9.1 File Organization

**Source files (.lmp)**:

- Place root `.lmp` at project root for global context
- Use component-specific `.lmp` files for complex modules
- Keep descriptions concise but informative
- Version control all `.lmp` files

**Generated files (.lmpc)**:

- Add `*.lmpc` to `.gitignore`
- Generate as needed, don't commit
- Use descriptive filenames for manual generation
- Clean up old context files periodically

### 9.2 Configuration Guidelines

**Include patterns**:

- Be selective with includes - more isn't always better
- Use exclusion patterns liberally for large projects
- Set appropriate token limits for your use case
- Document why specific files are included

**Context content**:

- Focus on architectural decisions and "why" not "what"
- Include domain-specific knowledge the LLM wouldn't know
- Document conventions and patterns to follow
- Avoid sensitive information

### 9.3 Workflow Integration

**Development workflow**:

```bash
# Generate context when starting work on a feature
cd my-project/frontend
lmp . --save

# Send to AI tool
cat frontend.lmpc | pbcopy
```

**CI/CD integration**:

```yaml
# Auto-update context files
- run: lmp . --output docs/project-context.lmpc
```

## 10. Examples

### 10.1 Complete Example

```json
{
  "name": "E-commerce API",
  "description": "FastAPI backend with PostgreSQL",
  "version": "2.1.0",
  "tech_stack": {
    "framework": "FastAPI",
    "language": "Python 3.11",
    "database": "PostgreSQL 15",
    "cache": "Redis",
    "deployment": "Docker + Kubernetes"
  },
  "include": [
    {
      "path": "./app/",
      "type": "dir",
      "patterns": ["*.py"],
      "exclude": ["*_test.py"],
      "description": "Main application code"
    },
    {
      "path": "./requirements.txt",
      "type": "file",
      "description": "Python dependencies"
    }
  ],
  "exclude": ["__pycache__/", "*.pyc", ".env*"],
  "conventions": {
    "code_style": "Black formatting, type hints required",
    "api_design": "RESTful with consistent error responses",
    "testing": "Pytest with factory pattern for test data"
  },
  "ai_context": {
    "focus_areas": ["async/await patterns", "database relationships"],
    "domain_knowledge": ["e-commerce workflows", "payment processing"],
    "avoid": ["synchronous database calls", "hardcoded credentials"],
    "patterns_to_follow": ["dependency injection", "repository pattern"],
    "performance_considerations": [
      "database query optimization",
      "caching strategies"
    ],
    "security_notes": ["JWT token validation", "input sanitization"]
  },
  "context": {
    "max_tokens": 75000,
    "merge_strategy": "inherit"
  }
}
```
