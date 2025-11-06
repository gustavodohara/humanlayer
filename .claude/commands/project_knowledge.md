---
description: Analyze project structure and generate comprehensive knowledge documentation
---

# Project Knowledge Base

You are tasked with analyzing a project's structure and creating comprehensive knowledge documentation. This command generates structured markdown files in `thoughts/shared/knowledge/` covering architecture, best practices, testing strategies, QA processes, features, and other critical project knowledge.

## Initial Setup

When this command is invoked, respond with:
```
I'll analyze this project's structure and create comprehensive knowledge documentation.

The analysis will create documents covering:
- Architecture and system design
- Folder structure and directory organization
- Best practices and conventions
- Testing strategies
- QA and quality processes
- Feature documentation (one per major feature)
- Development workflow
- Technology stack
- Dependencies and integrations

I'll save all documents to `thoughts/shared/knowledge/`. Shall I proceed?
```

Then wait for user confirmation, or proceed immediately if invoked with a parameter.

## Process Steps

### Step 1: Initial Project Discovery

1. **Read key project files first**:
   - Read `README.md` if it exists (FULLY, no limit/offset)
   - Read `package.json`, `go.mod`, `Cargo.toml`, or other dependency files
   - Read configuration files (`.gitignore`, `Makefile`, `docker-compose.yml`, etc.)
   - Read any existing documentation in `docs/` or similar
   - **IMPORTANT**: Use Read tool WITHOUT limit/offset parameters
   - **CRITICAL**: Read these files yourself in main context before spawning sub-tasks

2. **Analyze project structure**:
   - Identify project type (monorepo, single repo, microservices, etc.)
   - Identify technology stack (languages, frameworks, tools)
   - Identify build system and dependencies
   - Create a TodoWrite list to track all analysis tasks

3. **Spawn parallel discovery tasks**:
   ```
   Task 1 - Project Structure Analysis:
   Analyze the directory structure to understand:
   1. Main directories and their purposes
   2. Entry points (main files, config files)
   3. Build and dependency management
   4. Test directories and patterns
   5. Documentation structure
   Use tools: ListDir, Read, Glob
   Return: Directory structure map with descriptions
   ```

   ```
   Task 1b - Detailed Folder Structure Documentation:
   Create comprehensive folder structure documentation:
   1. Map complete directory tree with descriptions
   2. Document purpose of each major directory
   3. Identify patterns in folder organization
   4. Document naming conventions for directories
   5. Note any special folder structures (monorepo, workspaces, etc.)
   6. Document configuration file locations
   7. Document build artifact locations
   Use tools: ListDir, Read, codebase-locator
   Return: Detailed folder structure with explanations for each directory level
   ```

   ```
   Task 2 - Technology Stack Identification:
   Identify all technologies used:
   1. Programming languages and versions
   2. Frameworks and libraries
   3. Build tools and scripts
   4. Database and storage systems
   5. External services and APIs
   Use tools: Read, Grep
   Return: Complete technology inventory
   ```

   ```
   Task 3 - Configuration and Environment:
   Find and document:
   1. Environment variables and configuration
   2. Build configurations
   3. Deployment configurations
   4. CI/CD setup
   5. Development environment setup
   Use tools: Read, Glob, Grep
   Return: Configuration documentation
   ```

### Step 2: Deep Analysis Phase

After initial discovery completes, spawn comprehensive parallel research:

1. **Architecture Analysis**:
   ```
   Task - Architecture Documentation:
   Document the system architecture:
   1. Overall architecture patterns (monolith, microservices, etc.)
   2. Component structure and organization
   3. Data flow and communication patterns
   4. Entry points and main modules
   5. Key architectural decisions
   Use: codebase-locator, codebase-analyzer, codebase-pattern-finder
   Return: Architectural overview with file references
   ```

2. **Best Practices and Conventions**:
   ```
   Task - Best Practices Documentation:
   Identify coding standards and conventions:
   1. Code style and formatting rules
   2. Naming conventions
   3. File organization patterns
   4. Error handling patterns
   5. Logging patterns
   6. Documentation standards
   Use: codebase-pattern-finder, codebase-analyzer
   Return: Documented conventions with examples
   ```

3. **Testing Strategy**:
   ```
   Task - Testing Documentation:
   Document testing approach:
   1. Test types (unit, integration, e2e)
   2. Test frameworks and tools
   3. Test structure and organization
   4. Test coverage approach
   5. Testing patterns and conventions
   6. How to run tests
   Use: codebase-locator, codebase-analyzer
   Return: Testing strategy documentation
   ```

4. **QA and Quality Processes**:
   ```
   Task - QA Documentation:
   Document quality assurance:
   1. Code review process
   2. Linting and static analysis
   3. Pre-commit hooks
   4. CI/CD quality gates
   5. Performance testing
   6. Security practices
   7. Release process
   Use: codebase-locator, Grep, Read
   Return: QA processes documentation
   ```

5. **Feature Identification**:
   ```
   Task - Feature Discovery:
   Identify major features and capabilities:
   1. Main features and modules
   2. API endpoints or public interfaces
   3. User-facing features
   4. Backend services
   5. Integration points
   Use: codebase-locator, codebase-analyzer
   Return: List of features with descriptions and file references
   ```

6. **Development Workflow**:
   ```
   Task - Workflow Documentation:
   Document development processes:
   1. How to set up development environment
   2. How to build the project
   3. How to run locally
   4. Git workflow and branching
   5. How to contribute
   6. Common development tasks
   Use: Read, codebase-locator
   Return: Development workflow guide
   ```

7. **Dependencies and Integrations**:
   ```
   Task - Dependencies Documentation:
   Document external dependencies:
   1. Third-party libraries and versions
   2. External services and APIs
   3. Database schemas
   4. Infrastructure dependencies
   5. Tool dependencies
   Use: Read, codebase-analyzer
   Return: Dependencies inventory
   ```

### Step 3: Wait for All Tasks to Complete

- **IMPORTANT**: Wait for ALL sub-agent tasks to complete before proceeding
- Compile all findings from all tasks
- Cross-reference findings to ensure consistency
- Identify any gaps that need additional research

### Step 4: Generate Metadata

1. **Run metadata script**:
   - Run `hack/spec_metadata.sh` to generate relevant metadata
   - Or gather manually: current date, git commit, branch, repository name

2. **Generate document structure**:
   All documents will be saved to `thoughts/shared/knowledge/` with the following structure:
   ```
   thoughts/shared/knowledge/
   ├── architecture.md
   ├── folder-structure.md
   ├── best-practices.md
   ├── testing.md
   ├── qa.md
   ├── development-workflow.md
   ├── technology-stack.md
   ├── dependencies.md
   ├── features/
   │   ├── feature-1-name.md
   │   ├── feature-2-name.md
   │   └── ...
   └── README.md (index of all knowledge documents)
   ```

### Step 5: Generate Knowledge Documents

For each document category, create a comprehensive markdown file with YAML frontmatter:

#### Architecture Document (`architecture.md`)
```markdown
---
date: [Current date and time with timezone in ISO format]
researcher: [Researcher name from thoughts status]
git_commit: [Current commit hash]
branch: [Current branch name]
repository: [Repository name]
topic: "Project Architecture"
tags: [architecture, knowledge, system-design]
status: complete
last_updated: [Current date in YYYY-MM-DD format]
last_updated_by: [Researcher name]
---

# Architecture: [Project Name]

**Date**: [Current date]
**Repository**: [Repository name]
**Git Commit**: [Current commit hash]

## Overview
[High-level architectural description]

## Architecture Pattern
[Monolithic, Microservices, Modular Monolith, etc.]

## System Components

### [Component 1]
- **Location**: `path/to/component/`
- **Purpose**: [Description]
- **Key Files**:
  - `file1.ext:line` - [Description]
  - `file2.ext:line` - [Description]
- **Dependencies**: [What it depends on]
- **Dependents**: [What depends on it]

### [Component 2]
...

## Data Flow
[How data flows through the system]

## Communication Patterns
[How components communicate]

## Key Architectural Decisions
- [Decision 1] - [Rationale]
- [Decision 2] - [Rationale]

## Entry Points
- `path/to/main.ext:line` - [Description]
- `path/to/entry.ext:line` - [Description]

## Code References
- `path/to/file.ext:line` - [Description]
- `another/file.ext:line` - [Description]
```

#### Folder Structure Document (`folder-structure.md`)
```markdown
---
date: [Current date and time with timezone in ISO format]
researcher: [Researcher name from thoughts status]
git_commit: [Current commit hash]
branch: [Current branch name]
repository: [Repository name]
topic: "Folder Structure and Organization"
tags: [folder-structure, organization, knowledge]
status: complete
last_updated: [Current date in YYYY-MM-DD format]
last_updated_by: [Researcher name]
---

# Folder Structure: [Project Name]

**Date**: [Current date]
**Repository**: [Repository name]
**Git Commit**: [Current commit hash]

## Overview
[High-level description of how the project is organized]

## Project Type
[Monorepo, Single Repository, Multi-package, etc.]

## Directory Tree

```
project-root/
├── directory1/          # [Purpose and description]
│   ├── subdir1/         # [Purpose]
│   └── subdir2/         # [Purpose]
├── directory2/          # [Purpose and description]
└── ...
```

## Root Level Directories

### [Directory Name] (`path/to/dir/`)
- **Purpose**: [What this directory contains and why it exists]
- **Key Contents**:
  - `file1.ext` - [Description]
  - `file2.ext` - [Description]
- **Relationship**: [How it relates to other directories]
- **Conventions**: [Any naming or organization conventions specific to this directory]

### [Another Directory] (`path/to/another/`)
...

## Source Code Organization

### Main Source Directories
- **`src/`** or **`app/`** or **`lib/`** - [Purpose and structure]
- **`components/`** or **`modules/`** - [How modules/components are organized]
- **`services/`** or **`api/`** - [How services/APIs are organized]

### Entry Points
- `path/to/main.ext` - [Main entry point description]
- `path/to/entry.ext` - [Other entry points]

## Configuration Directories

### Build Configuration
- **Location**: `path/to/config/`
- **Files**: [List of config files and their purposes]

### Environment Configuration
- **Location**: `path/to/env/`
- **Files**: [Environment-specific files]

## Test Organization

### Test Directory Structure
- **Unit Tests**: `path/to/unit-tests/` - [Structure]
- **Integration Tests**: `path/to/integration-tests/` - [Structure]
- **E2E Tests**: `path/to/e2e-tests/` - [Structure]
- **Test Utilities**: `path/to/test-utils/` - [Structure]

### Test Naming Conventions
[How tests are named and organized]

## Documentation Organization

### Documentation Structure
- **`docs/`** - [What documentation lives here]
- **`README.md`** files - [Where READMEs are located and their purposes]
- **`examples/`** - [Example code organization]

## Build Artifacts

### Build Output Locations
- **`dist/`** or **`build/`** - [What gets built here]
- **`out/`** - [Other build outputs]
- **`.cache/`** - [Cache directories]

## Special Directories

### Workspace/Monorepo Structure
[If applicable, document workspace structure]

### Generated Directories
- **`node_modules/`** - [Purpose]
- **`.next/`** or **`.nuxt/`** - [Framework-specific generated content]
- **Other generated dirs** - [List]

### Hidden Directories
- **`.git/`** - [Git repository]
- **`.idea/`** or **`.vscode/`** - [IDE configuration]
- **Other hidden dirs** - [List with purposes]

## Naming Conventions

### Directory Naming
- **Pattern**: [kebab-case, camelCase, etc.]
- **Examples**: [Examples from codebase]

### File Organization Patterns
- **By Feature**: [If organized by feature]
- **By Type**: [If organized by file type]
- **By Layer**: [If organized by architectural layer]

## Cross-Directory Patterns

### Shared Code
- **Location**: `path/to/shared/`
- **Purpose**: [What's shared and why]

### Common Utilities
- **Location**: `path/to/utils/`
- **Purpose**: [Utility organization]

### Constants and Configuration
- **Location**: `path/to/config/`
- **Purpose**: [How configuration is organized]

## Migration Notes
[If the project has migrated folder structures, document the current state]

## Code References
- `path/to/structure/file.ext` - [Description]
- `another/structure/file.ext` - [Description]
```

#### Best Practices Document (`best-practices.md`)
```markdown
---
date: [Current date and time with timezone in ISO format]
researcher: [Researcher name from thoughts status]
git_commit: [Current commit hash]
branch: [Current branch name]
repository: [Repository name]
topic: "Best Practices and Conventions"
tags: [best-practices, conventions, coding-standards, knowledge]
status: complete
last_updated: [Current date in YYYY-MM-DD format]
last_updated_by: [Researcher name]
---

# Best Practices: [Project Name]

## Code Style

### Formatting
[Formatting rules and tools used]

### Naming Conventions
- **Files**: [File naming patterns]
- **Variables**: [Variable naming patterns]
- **Functions**: [Function naming patterns]
- **Classes**: [Class naming patterns]

### Examples
```language
// Example from codebase
path/to/example.ext:line
```

## File Organization

### Directory Structure
[How files are organized]

### Module Organization
[How modules/packages are structured]

## Error Handling

### Patterns Used
[Error handling patterns found]

### Examples
```language
// Example from codebase
path/to/error-handling.ext:line
```

## Logging

### Logging Strategy
[How logging is implemented]

### Examples
```language
// Example from codebase
path/to/logging.ext:line
```

## Documentation Standards

### Code Documentation
[How code is documented]

### README Standards
[What READMEs should contain]

## Code References
- `path/to/file.ext:line` - [Description]
```

#### Testing Document (`testing.md`)
```markdown
---
date: [Current date and time with timezone in ISO format]
researcher: [Researcher name from thoughts status]
git_commit: [Current commit hash]
branch: [Current branch name]
repository: [Repository name]
topic: "Testing Strategy"
tags: [testing, quality, knowledge]
status: complete
last_updated: [Current date in YYYY-MM-DD format]
last_updated_by: [Researcher name]
---

# Testing: [Project Name]

## Overview
[Testing philosophy and approach]

## Test Types

### Unit Tests
- **Location**: `path/to/unit-tests/`
- **Framework**: [Framework name]
- **Pattern**: [How unit tests are structured]
- **Examples**: `path/to/example-test.ext:line`

### Integration Tests
- **Location**: `path/to/integration-tests/`
- **Framework**: [Framework name]
- **Pattern**: [How integration tests are structured]
- **Examples**: `path/to/example-test.ext:line`

### End-to-End Tests
- **Location**: `path/to/e2e-tests/`
- **Framework**: [Framework name]
- **Pattern**: [How e2e tests are structured]
- **Examples**: `path/to/example-test.ext:line`˚

## Running Tests

### Commands
```bash
# Unit tests
make test

# All tests
make test-all

# Coverage
make test-coverage
```

## Test Organization

### Structure
[How tests are organized]

### Naming Conventions
[Test naming patterns]

## Test Coverage
[Coverage approach and targets]

## Mocking and Fixtures
[How mocking is done]

## Code References
- `path/to/test.ext:line` - [Description]
```

#### QA Document (`qa.md`)
```markdown
---
date: [Current date and time with timezone in ISO format]
researcher: [Researcher name from thoughts status]
git_commit: [Current commit hash]
branch: [Current branch name]
repository: [Repository name]
topic: "Quality Assurance"
tags: [qa, quality, processes, knowledge]
status: complete
last_updated: [Current date in YYYY-MM-DD format]
last_updated_by: [Researcher name]
---

# QA and Quality: [Project Name]

## Code Review Process

### Review Requirements
[What's required for code review]

### Review Checklist
- [ ] Tests pass
- [ ] Code follows style guide
- [ ] Documentation updated
- ...

## Static Analysis

### Linting
- **Tool**: [Linter name]
- **Config**: `path/to/config.ext`
- **Command**: `make lint`

### Type Checking
- **Tool**: [Type checker name]
- **Command**: `make typecheck`

## Pre-commit Hooks
[What runs before commits]

## CI/CD Quality Gates

### Automated Checks
[What runs in CI/CD]

### Quality Metrics
[Metrics tracked]

## Performance Testing
[How performance is tested]

## Security Practices
[Security measures in place]

## Release Process
[How releases are made]

## Code References
- `path/to/config.ext` - [Description]
```

#### Feature Documents (`features/feature-name.md`)

For each major feature identified, create a dedicated document:

```markdown
---
date: [Current date and time with timezone in ISO format]
researcher: [Researcher name from thoughts status]
git_commit: [Current commit hash]
branch: [Current branch name]
repository: [Repository name]
topic: "[Feature Name]"
tags: [feature, knowledge, functionality]
status: complete
last_updated: [Current date in YYYY-MM-DD format]
last_updated_by: [Researcher name]
---

# Feature: [Feature Name]

## Overview
[What this feature does]

## Purpose
[Why this feature exists]

## Implementation

### Main Components
- **Location**: `path/to/feature/`
- **Key Files**:
  - `file1.ext:line` - [Description]
  - `file2.ext:line` - [Description]

### Data Flow
[How data flows through this feature]

### API/Interface
[Public interface for this feature]

## Dependencies
[What this feature depends on]

## Usage Examples
```language
// Example usage
path/to/example.ext:line
```

## Related Features
[Other features this relates to]

## Code References
- `path/to/file.ext:line` - [Description]
```

#### Development Workflow (`development-workflow.md`)
```markdown
---
date: [Current date and time with timezone in ISO format]
researcher: [Researcher name from thoughts status]
git_commit: [Current commit hash]
branch: [Current branch name]
repository: [Repository name]
topic: "Development Workflow"
tags: [workflow, development, knowledge]
status: complete
last_updated: [Current date in YYYY-MM-DD format]
last_updated_by: [Researcher name]
---

# Development Workflow: [Project Name]

## Setup

### Prerequisites
[What's needed to start]

### Initial Setup
```bash
# Commands to set up
```

## Building

### Build Commands
```bash
# Build commands
```

## Running Locally

### Development Server
```bash
# How to run dev server
```

### Testing Locally
```bash
# How to test locally
```

## Git Workflow

### Branching Strategy
[Branching approach]

### Commit Conventions
[Commit message format]

## Common Tasks

### Adding a New Feature
[Steps to add a feature]

### Running Tests
[How to run tests]

### Debugging
[How to debug]

## Code References
- `path/to/config.ext` - [Description]
```

#### Technology Stack (`technology-stack.md`)
```markdown
---
date: [Current date and time with timezone in ISO format]
researcher: [Researcher name from thoughts status]
git_commit: [Current commit hash]
branch: [Current branch name]
repository: [Repository name]
topic: "Technology Stack"
tags: [technology, stack, dependencies, knowledge]
status: complete
last_updated: [Current date in YYYY-MM-DD format]
last_updated_by: [Researcher name]
---

# Technology Stack: [Project Name]

## Languages
- [Language 1] - [Version] - [Purpose]
- [Language 2] - [Version] - [Purpose]

## Frameworks
- [Framework 1] - [Version] - [Purpose]
- [Framework 2] - [Version] - [Purpose]

## Build Tools
- [Tool 1] - [Version] - [Purpose]
- [Tool 2] - [Version] - [Purpose]

## Databases
- [Database 1] - [Version] - [Purpose]
- [Database 2] - [Version] - [Purpose]

## External Services
- [Service 1] - [Purpose]
- [Service 2] - [Purpose]

## Development Tools
- [Tool 1] - [Purpose]
- [Tool 2] - [Purpose]
```

#### Dependencies Document (`dependencies.md`)
```markdown
---
date: [Current date and time with timezone in ISO format]
researcher: [Researcher name from thoughts status]
git_commit: [Current commit hash]
branch: [Current branch name]
repository: [Repository name]
topic: "Dependencies"
tags: [dependencies, integrations, knowledge]
status: complete
last_updated: [Current date in YYYY-MM-DD format]
last_updated_by: [Researcher name]
---

# Dependencies: [Project Name]

## Runtime Dependencies
[List of runtime dependencies with versions]

## Development Dependencies
[List of dev dependencies]

## External APIs
- [API 1] - [Purpose] - [Documentation]
- [API 2] - [Purpose] - [Documentation]

## Infrastructure
- [Infrastructure component 1]
- [Infrastructure component 2]

## Database Schemas
[Database schema information]
```

#### Knowledge Base Index (`README.md`)
```markdown
# [Project Name] Knowledge Base

This directory contains comprehensive knowledge documentation for [Project Name].

## Documents

- [Architecture](architecture.md) - System architecture and design
- [Folder Structure](folder-structure.md) - Directory organization and structure
- [Best Practices](best-practices.md) - Coding standards and conventions
- [Testing](testing.md) - Testing strategies and approaches
- [QA](qa.md) - Quality assurance processes
- [Development Workflow](development-workflow.md) - How to develop and contribute
- [Technology Stack](technology-stack.md) - Technologies and tools used
- [Dependencies](dependencies.md) - External dependencies and integrations

## Features

- [Feature 1](features/feature-1.md)
- [Feature 2](features/feature-2.md)
- ...

## Last Updated
[Date] by [Researcher]

## Git Commit
[Commit hash]
```

### Step 6: Sync and Present

1. **Sync the knowledge documents**:
   - Run `git add thoughts/shared/knowledge/ && git commit -m "Add project knowledge base documentation"`

2. **Present the results**:
   ```
   I've created comprehensive knowledge documentation for [Project Name] in `thoughts/shared/knowledge/`.

   Generated documents:
   - architecture.md - System architecture overview
   - folder-structure.md - Directory organization and folder structure
   - best-practices.md - Coding standards and conventions
   - testing.md - Testing strategies
   - qa.md - Quality assurance processes
   - development-workflow.md - Development guide
   - technology-stack.md - Technology inventory
   - dependencies.md - Dependencies and integrations
   - features/ - Individual feature documentation ([N] features)
   - README.md - Knowledge base index

   All documents are ready for review and can be updated as the project evolves.
   ```

## Important Guidelines

1. **Be Comprehensive**:
   - Document everything that exists, not what should exist
   - Include specific file paths and line numbers
   - Reference actual code examples

2. **Use Parallel Research**:
   - Always spawn multiple sub-agents in parallel
   - Use specialized agents for different research areas
   - Wait for all tasks to complete before synthesizing

3. **Maintain Consistency**:
   - Use consistent frontmatter format across all documents
   - Follow the same structure patterns
   - Include code references with file:line format

4. **Document As-Is**:
   - Document what exists, not recommendations
   - Be factual and descriptive
   - Include examples from actual codebase

5. **File Reading**:
   - Always read mentioned files FULLY (no limit/offset)
   - Read key files before spawning sub-tasks
   - Ensure complete context before proceeding

6. **Critical Ordering**:
   - ALWAYS read key files first (step 1)
   - ALWAYS wait for all sub-agents to complete (step 3)
   - ALWAYS gather metadata before writing (step 4)
   - NEVER write documents with placeholder values

7. **Path Handling**:
   - Use correct paths for file references
   - Include GitHub permalinks when possible
   - Maintain consistent path formatting

## Customization

The user can request specific focuses:
- Focus on specific areas (e.g., "focus on backend architecture")
- Skip certain categories (e.g., "skip QA documentation")
- Add custom categories (e.g., "include deployment documentation")

Adapt the analysis and document generation accordingly.

## Example Interaction Flow

```
User: /project_knowledge
Assistant: I'll analyze this project's structure and create comprehensive knowledge documentation...

[Spawning parallel research tasks]
[Waiting for all tasks to complete]
[Generating documents]

I've created comprehensive knowledge documentation for [Project Name] in `thoughts/shared/knowledge/`.

Generated documents:
- architecture.md
- folder-structure.md
- best-practices.md
- testing.md
- qa.md
- development-workflow.md
- technology-stack.md
- dependencies.md
- features/ - 5 feature documents
- README.md

All documents are ready for review.
```

