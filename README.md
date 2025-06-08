# LMP: Language Model Prompt

LMP: A spec, file format, and parser for giving large language models context about your projects

## What is LMP?

LMP (Language Model Prompt) files combine human-readable documentation with machine-parseable configuration to provide rich context to AI coding assistants.

**Problem**: Explaining your codebase to AI tools is tedious and inconsistent  
**Solution**: `.lmp` file(s) that captures your project's context perfectly

````markdown
# My Awesome Project

This is a React TypeScript application for task management.

## Architecture

- Frontend: React 18 with TypeScript
- State: Zustand for state management
- Styling: Tailwind CSS
- Build: Vite

## Development Notes

Always use functional components and prefer composition over inheritance.

```json
{
  "name": "My Awesome Project",
  "include": [
    {
      "path": "./src/",
      "type": "dir",
      "patterns": ["*.tsx", "*.ts"],
      "exclude": ["*.test.ts"]
    },
    {
      "path": "./package.json",
      "type": "file"
    }
  ]
}
```
````

## Why LMP?

🎯 **One file, complete context** - Everything an LLM needs to understand your project  
📝 **Human + Machine readable** - Natural documentation with structured configuration  
🔄 **Version controlled** - Context evolves with your codebase  
🌍 **Universal** - Works with any LLM, any language, any project  
⚡ **Zero configuration** - Drop a `.lmp` file anywhere in your project

## Quick Start

### 1. Install

Choose your preferred language:

```bash
# JavaScript/Node.js
npm install -g lmp4u

# Python
pip install lmp4u

# Go
go install github.com/lmp-format/lmp@latest

# Rust
cargo install lmp4u

# Or download pre-built binaries
curl -L https://github.com/lmp-format/lmp/releases/latest/download/lmp-linux-x64 -o lmp
chmod +x lmp && sudo mv lmp /usr/local/bin/
```

### 2. Create a .lmp file

```bash
lmp init
```

This creates a `.lmp` file with smart defaults based on your project structure.

### 3. Generate context

```bash
lmp .
```

This outputs rich context that you can copy-paste to any AI coding assistant.

### 4. Use with AI tools

```bash
# Save for later use
lmp . --output context.md

# Copy to clipboard (macOS)
lmp . | pbcopy

# Integrate with your workflow
lmp . --output .github/ai-context.md
```

## Real-World Examples

### React TypeScript Project

````markdown
# E-commerce Frontend

Modern React e-commerce application with TypeScript.

## Key Features

- Product catalog with search and filters
- Shopping cart with persistent state
- Stripe payment integration
- Admin dashboard

## Architecture Decisions

- **State Management**: Zustand for client state, React Query for server state
- **Styling**: Tailwind CSS with custom design system tokens
- **Forms**: React Hook Form with Zod validation
- **Testing**: Jest + React Testing Library, focusing on user interactions

## Code Conventions

- Functional components with hooks only
- Custom hooks for complex logic
- Barrel exports from index files
- Strict TypeScript with no `any`

```json
{
  "name": "E-commerce Frontend",
  "description": "React TypeScript e-commerce application",
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
      "description": "Dependencies and scripts"
    },
    {
      "path": "./tailwind.config.js",
      "type": "file",
      "description": "Design system configuration"
    }
  ],
  "exclude": ["node_modules/", ".next/", "dist/", "*.log"],
  "tech_stack": {
    "framework": "React 18",
    "language": "TypeScript",
    "styling": "Tailwind CSS",
    "state": "Zustand + React Query",
    "build": "Vite",
    "testing": "Jest + RTL"
  },
  "conventions": {
    "components": "Functional components with hooks, prefer composition",
    "styling": "Tailwind utility classes, custom components in design system",
    "state": "Zustand stores for global state, local state for component-specific",
    "imports": "Absolute imports from src/, barrel exports from directories"
  },
  "ai_context": {
    "focus_areas": [
      "component architecture",
      "state management patterns",
      "TypeScript usage"
    ],
    "avoid": ["class components", "inline styles", "prop drilling"],
    "patterns_to_follow": [
      "custom hooks for logic",
      "compound components",
      "render props when needed"
    ]
  }
}
```
````

### Python FastAPI Service

````markdown
# Task Management API

RESTful API built with FastAPI, PostgreSQL, and Redis.

## System Overview

High-performance async API handling 10k+ requests/second with real-time features.

## Key Features

- JWT authentication with refresh tokens
- Real-time WebSocket notifications
- Background task processing with Celery
- OpenAPI documentation with Swagger UI
- Rate limiting and request validation

## Database Design

- **PostgreSQL**: Main data store with JSONB for flexible schemas
- **Redis**: Session storage, rate limiting, pub/sub for real-time
- **Alembic**: Database migrations with rollback support

```yaml
name: "Task Management API"
description: "FastAPI backend with PostgreSQL and Redis"
tech_stack:
  framework: "FastAPI"
  language: "Python 3.11"
  database: "PostgreSQL 15"
  cache: "Redis 7"
  task_queue: "Celery"
  deployment: "Docker + Kubernetes"

include:
  - path: "./app/"
    type: "dir"
    patterns: ["*.py"]
    exclude: ["*_test.py", "test_*.py"]
    description: "Application source code"
  - path: "./requirements.txt"
    type: "file"
    description: "Python dependencies"
  - path: "./alembic/"
    type: "dir"
    patterns: ["*.py"]
    description: "Database migrations"
  - path: "./docker-compose.yml"
    type: "file"
    description: "Development environment"

exclude:
  - "__pycache__/"
  - ".pytest_cache/"
  - "*.pyc"
  - ".env*"

conventions:
  code_style: "Black formatting, isort imports, type hints required"
  api_design: "RESTful endpoints, consistent error responses, OpenAPI specs"
  database: "SQLAlchemy with async sessions, Alembic for migrations"
  testing: "Pytest with async test client, factory pattern for test data"
  security: "JWT tokens, password hashing with bcrypt, input validation"

ai_context:
  focus_areas: ["async/await patterns", "database relationships", "API design"]
  domain_knowledge: "Task management workflows, user permissions, notification systems"
  performance_considerations: "Database query optimization, caching strategies, background tasks"
```
````

### Go CLI Tool

````markdown
# Kubernetes Deployment Tool

Command-line tool for managing Kubernetes deployments with GitOps workflow.

## Purpose

Simplifies Kubernetes deployments by providing opinionated workflows and safety checks.

## Features

- GitOps-style deployments with rollback capability
- Multi-environment configuration management
- Pre-deployment validation and health checks
- Integration with popular CI/CD platforms

```toml
name = "k8s-deploy"
description = "Kubernetes deployment automation tool"

[tech_stack]
language = "Go 1.21"
framework = "Cobra CLI"
kubernetes = "client-go v0.28"
config = "Viper"

[[include]]
path = "./cmd/"
type = "dir"
patterns = ["*.go"]
description = "CLI commands and subcommands"

[[include]]
path = "./pkg/"
type = "dir"
patterns = ["*.go"]
exclude = ["*_test.go"]
description = "Core application packages"

[[include]]
path = "./internal/"
type = "dir"
patterns = ["*.go"]
exclude = ["*_test.go"]
description = "Internal application logic"

[[include]]
path = "./go.mod"
type = "file"

[[include]]
path = "./go.sum"
type = "file"

[conventions]
code_style = "gofmt, golint clean, effective Go principles"
cli_design = "Cobra framework with consistent flag naming and help text"
error_handling = "Wrap errors with context, fail fast with clear messages"
testing = "Table-driven tests, testify for assertions, mock external dependencies"
packaging = "Clear separation between cmd/, pkg/, and internal/"

[ai_context]
focus_areas = ["CLI design patterns", "Kubernetes API usage", "error handling"]
domain_knowledge = ["Kubernetes concepts", "GitOps workflows", "deployment strategies"]
go_patterns = ["interfaces for testability", "context for cancellation", "structured logging"]
```
````

## Language Support

| Language               | Package Name                | CLI Command | Status    |
| ---------------------- | --------------------------- | ----------- | --------- |
| **JavaScript/Node.js** | `lmp4u`                     | `lmp`       | ✅ Stable |
| **Python**             | `lmp4u`                     | `lmp`       | ✅ Stable |
| **Go**                 | `github.com/lmp-format/lmp` | `lmp`       | ✅ Stable |
| **Rust**               | `lmp4u`                     | `lmp`       | ✅ Stable |
| **Java**               | `com.lmp:lmp4u`             | `lmp`       | 🔄 Beta   |
| **C#/.NET**            | `LMP4U`                     | `lmp`       | 🔄 Beta   |

All implementations provide identical CLI interfaces and output formats.

## File Format Deep Dive

### Structure

LMP files combine two optional sections:

1. **Markdown Documentation**: Write anything - architecture notes, setup instructions, context for AI
2. **Structured Configuration**: JSON/YAML/TOML that tells the parser exactly what to include

### Code Fence Delimiters

Use code fences to separate documentation from configuration:

````markdown
# Your documentation here...

```json
{
  "your": "configuration here"
}
```
````

Supported formats: `json`, `yaml`, `yml`, `toml`

### Configuration Schema

The structured section can include:

```typescript
interface LMPConfig {
  // Project metadata
  name?: string;
  description?: string;
  version?: string;

  // What to include
  include: IncludeEntry[];
  exclude?: string[];

  // Project context
  tech_stack?: Record<string, string>;
  conventions?: Record<string, string>;
  ai_context?: {
    focus_areas?: string[];
    domain_knowledge?: string[];
    avoid?: string[];
    patterns_to_follow?: string[];
  };

  // Legacy/simple fields
  ai_instructions?: string;
}
```

## CLI Reference

### Commands

```bash
lmp [path]                   # Generate context (default)
lmp init [path]              # Create new .lmp file
lmp validate [file]          # Validate .lmp syntax
lmp preview [path]           # Preview what would be included
```

### Options

```bash
--output, -o FILE            # Save to file instead of stdout
--verbose, -v                # Show detailed logging
--max-tokens NUMBER          # Token limit (default: 50000)
--format FORMAT              # Output format: markdown, json
--dry-run                    # Show actions without executing
```

### Examples

```bash
lmp . --output context.md    # Save context to file
lmp . --verbose             # See what files are being processed
lmp . --max-tokens 100000   # Increase token limit
lmp init --template react   # Create React-specific template
lmp validate --fix          # Auto-fix common issues
```

## Integration Examples

### GitHub Actions

```yaml
name: Update AI Context
on:
  push:
    branches: [main]
jobs:
  update-context:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Generate LMP context
        run: |
          npm install -g lmp4u
          lmp . --output docs/ai-context.md
      - name: Commit updated context
        run: |
          git config --local user.email "action@github.com"
          git config --local user.name "GitHub Action"
          git add docs/ai-context.md
          git commit -m "Update AI context" || exit 0
          git push
```

### Pre-commit Hook

```bash
#!/bin/sh
# .git/hooks/pre-commit
lmp validate || exit 1
lmp . --output .github/ai-context.md
git add .github/ai-context.md
```

### VS Code Integration

```json
{
  "tasks": [
    {
      "label": "Generate LMP Context",
      "type": "shell",
      "command": "lmp",
      "args": [".", "--output", "context.md"],
      "group": "build"
    }
  ]
}
```

## Project Structure

```
lmp/
├── README.md               # This file
├── SPEC.md                # Technical specification
├── implementations/       # Language implementations
│   ├── javascript/        # Node.js parser
│   ├── python/           # Python parser
│   ├── go/               # Go parser
│   └── rust/             # Rust parser
└── examples/             # Example .lmp files
    ├── react-project.lmp
    ├── python-api.lmp
    └── go-cli.lmp
```

## Development

Each implementation has its own development environment:

```bash
cd implementations/javascript && nix develop
cd implementations/python && nix develop
cd implementations/go && nix develop
cd implementations/rust && nix develop
```

## FAQ

**Q: How is LMP different from README files?**
A: README files are for humans. LMP files are structured for both humans AND machines, with precise include/exclude rules for generating AI context.

**Q: Can I use multiple .lmp files in one project?**
A: Yes! LMP supports hierarchical contexts - place .lmp files in subdirectories and they'll be merged intelligently.

**Q: What's the performance impact?**
A: LMP parsers are fast - typically under 100ms for medium-sized projects. File content is only read when explicitly included.

**Q: Does LMP work with monorepos?**
A: Absolutely! Place .lmp files at different levels of your monorepo to provide context for different components.

**Q: How do I handle sensitive information?**
A: Use exclude patterns to skip sensitive files, or place .lmp files in subdirectories that don't contain secrets.

## Contributing

See individual implementation directories for language-specific contribution guides:

- [JavaScript](implementations/javascript/README.md)
- [Python](implementations/python/README.md)
- [Go](implementations/go/README.md)
- [Rust](implementations/rust/README.md)

For specification changes, please open an issue to discuss before submitting a PR.

The LMP specification is public domain to encourage adoption across tools and platforms.

---

**Ready to get started?** Run `lmp init` in your project and never explain your codebase to an AI again!
