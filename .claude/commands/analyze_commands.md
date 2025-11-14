---
description: Analyze slash commands structure, patterns, and help identify which command to use
---

# Analyze Commands

You are tasked with analyzing the available slash commands in the `.claude/commands/` directory, understanding their structure, patterns, and helping users identify which command is most appropriate for their current task.

## Initial Response

When invoked WITHOUT parameters:
```
I'll analyze the available slash commands and help you understand:
- What commands are available
- When to use each command
- Common patterns and conventions
- Command dependencies and workflows

What would you like to know?
1. List all available commands with descriptions
2. Find the right command for a specific task
3. Analyze command patterns and structure
4. Show command workflows and relationships
5. Deep dive into a specific command
```

When invoked WITH a specific question or task:
```
I'll analyze the commands to help with: [user's request]
```

## Command Categories

Commands are organized into these functional categories:

### 1. Research & Discovery
- `research_codebase` - Document codebase as-is with thoughts directory
- `research_codebase_nt` - Document codebase without thoughts directory
- `research_codebase_generic` - Generic codebase research with parallel agents
- `ralph_research` - Research highest priority Linear ticket

### 2. Planning & Implementation
- `create_plan` - Create detailed implementation plans with research
- `create_plan_nt` - Create plans without thoughts directory
- `create_plan_generic` - Generic plan creation with thorough research
- `iterate_plan` - Update existing plans based on new findings
- `implement_plan` - Execute implementation from a plan document
- `validate_plan` - Verify implementation matches plan specifications

### 3. Linear/Project Management
- `linear` - Manage tickets: create, update, comment, workflow
- `ralph_plan` - Create plan for highest priority ready-for-spec ticket
- `ralph_impl` - Implement highest priority small ticket with worktree
- `founder_mode` - Create Linear ticket and PR for experimental features

### 4. Development Workflow
- `debug` - Debug issues using logs, database state, git history
- `commit` - Create git commits with user approval
- `ci_commit` - Create git commits for session changes automatically
- `describe_pr` - Generate comprehensive PR descriptions
- `ci_describe_pr` - Generate PR descriptions following templates
- `local_review` - Set up worktree for reviewing colleague's branch

### 5. Session Management
- `create_handoff` - Create handoff document for transferring work
- `resume_handoff` - Resume work from handoff document with context

### 6. Specialized Workflows
- `oneshot` - Research ticket and launch planning session
- `oneshot_plan` - Execute ralph plan and implementation for ticket
- `project_knowledge` - Analyze project structure and generate documentation

## Common Patterns Across Commands

### 1. Structure Template
```markdown
---
description: [Brief description for command listing]
model: [optional - opus, sonnet, haiku]
---

# [Command Name]

[Introduction paragraph explaining the command's purpose]

## Initial Setup/Response
[What to say when the command is invoked]

## Steps to Follow
[Detailed step-by-step process]

## Important Notes/Guidelines
[Key considerations and conventions]
```

### 2. TodoWrite Integration
Most commands emphasize using TodoWrite to:
- Plan and track subtasks
- Show progress to users
- Break down complex work into manageable steps

### 3. Parallel Task Spawning
Commands frequently spawn parallel Task agents for:
- Efficient codebase exploration
- Simultaneous research across multiple areas
- Minimizing context usage
- Faster completion

### 4. Thoughts Directory Integration
Research and planning commands:
- Store findings in `thoughts/shared/research/`
- Store plans in `thoughts/shared/plans/`
- Store tickets in `thoughts/shared/tickets/`
- Filename format: `YYYY-MM-DD-ENG-XXXX-description.md`
- Always sync with `humanlayer thoughts sync`

### 5. Metadata Gathering
Commands that create documents:
- Run `hack/spec_metadata.sh` for metadata
- Include frontmatter with: date, researcher, git_commit, branch, tags
- Add GitHub permalinks when on main/pushed branches

### 6. Linear Integration
Commands working with tickets:
- Use MCP tools (`mcp__linear__*`)
- Follow workflow progression (Triage → Spec → Research → Plan → Dev)
- Attach links using the `links` parameter
- Add comments with key insights

## Command Selection Guide

### For Understanding Code
**Question**: "How does feature X work?"
**Command**: `/research_codebase` (or `_nt` if no thoughts directory)
**Why**: Spawns parallel agents to document existing code without critique

### For Fixing Bugs
**During Development**: `/debug` - Investigates logs, database, git state
**For Implementation**: `/create_plan` → `/implement_plan` → `/validate_plan`

### For New Features
**Full Workflow**:
1. `/research_codebase` - Understand existing implementation
2. `/create_plan` - Design the solution
3. `/implement_plan` - Execute the plan
4. `/validate_plan` - Verify completeness
5. `/commit` - Create commits
6. `/describe_pr` - Generate PR description

### For Linear Tickets
**Research Phase**: `/ralph_research` - Investigates and documents findings
**Planning Phase**: `/ralph_plan` - Creates implementation plan
**Implementation**: `/ralph_impl` - Builds the feature
**One-shot**: `/oneshot_plan` - Does research + plan + implementation

### For Session Management
**Before Break**: `/create_handoff` - Documents current state
**After Break**: `/resume_handoff` - Restores context and continues

### For Quick Tasks
**Experimental Features**: `/founder_mode` - Implement, commit, PR, ticket
**Single Ticket**: `/oneshot` - Research and plan only

## Command Workflows

### Standard Feature Development
```
/research_codebase → /create_plan → /implement_plan → /validate_plan → /commit → /describe_pr
```

### Linear-Driven Development
```
/ralph_research → /ralph_plan → /ralph_impl → /commit → /describe_pr
```

### Quick Experiments
```
/founder_mode (implements, commits, creates PR and ticket)
```

### Debugging Session
```
/debug → [manual fixes] → /commit
```

### Long-Running Work
```
Day 1: /research_codebase → /create_handoff
Day 2: /resume_handoff → /implement_plan → /create_handoff
Day 3: /resume_handoff → /validate_plan → /commit → /describe_pr
```

## Action-Specific Instructions

### 1. List All Commands with Descriptions

When user requests a command listing:

1. **Read all command files** to get their frontmatter descriptions
2. **Group by category** as shown above
3. **Present in a table format**:
   ```markdown
   ## Available Commands

   ### Research & Discovery
   | Command | Description |
   |---------|-------------|
   | /research_codebase | Document codebase as-is with thoughts |
   | ... | ... |
   ```

### 2. Find Right Command for Task

When user describes what they want to do:

1. **Analyze the request** to identify:
   - Is it research, planning, implementation, or debugging?
   - Does it involve Linear tickets?
   - Is it a quick task or long-running work?
   - Do they need session handoff?

2. **Match patterns**:
   - Keywords like "understand", "how does" → research commands
   - "implement", "build", "add feature" → planning/implementation
   - "bug", "error", "not working" → debug command
   - "ticket", "Linear" → ralph_* or linear commands

3. **Provide recommendation**:
   ```markdown
   Based on your request, I recommend: `/[command]`

   **Why**: [Explanation of why this fits]

   **Alternative**: If you need [different aspect], consider `/[other_command]`

   **Workflow**:
   1. Start with `/[command]`
   2. Then use `/[next_command]`
   3. Finally `/[final_command]`
   ```

### 3. Analyze Command Patterns

When user wants to understand command structure:

1. **Spawn parallel Task agents** to analyze different aspects:
   ```
   Task 1 - Structure Analysis:
   Read all command files and analyze:
   - Frontmatter patterns
   - Section organization
   - Common sections used
   Return: Structural patterns with examples

   Task 2 - Convention Analysis:
   Identify conventions across commands:
   - Naming patterns
   - TodoWrite usage
   - Task spawning patterns
   Return: Convention patterns with frequencies

   Task 3 - Integration Analysis:
   Map tool integrations:
   - Linear MCP usage
   - Thoughts directory patterns
   - Git/GitHub integration
   Return: Integration patterns and dependencies
   ```

2. **Synthesize findings** into a report
3. **Present with examples** from actual commands

### 4. Show Command Workflows

When user wants to understand command relationships:

1. **Identify workflow types**:
   - Feature development workflows
   - Bug fixing workflows
   - Research workflows
   - Linear ticket workflows

2. **Map command chains** using actual command examples

3. **Visualize with flowcharts** using markdown

4. **Explain decision points**: When to use one command vs another

### 5. Deep Dive into Specific Command

When user asks about a specific command:

1. **Read the command file** completely
2. **Extract key information**:
   - Purpose and use cases
   - Step-by-step process
   - Important conventions
   - Integration points
   - Examples if available

3. **Present structured summary**:
   ```markdown
   ## /[command] - Deep Dive

   ### Purpose
   [What it does and when to use it]

   ### Process Overview
   1. [High-level step 1]
   2. [High-level step 2]
   ...

   ### Key Features
   - [Important capability 1]
   - [Important capability 2]

   ### Integrations
   - [Tool/system it integrates with]

   ### Tips & Best Practices
   - [Pro tip 1]
   - [Pro tip 2]

   ### Related Commands
   - Use before: `/[command]`
   - Use after: `/[command]`
   ```

## Important Notes

- **Read-only analysis**: This command only analyzes, never modifies commands
- **Stay current**: Always read actual command files, don't rely on cached knowledge
- **Provide examples**: Use real command patterns from the codebase
- **Consider context**: Some commands are project-specific (thoughts/, Linear)
- **Explain tradeoffs**: Help users understand why one command over another
- **Workflow thinking**: Show how commands chain together
- **Progressive disclosure**: Start simple, go deeper as needed

## Meta-Analysis Capabilities

This command can also analyze:
- **Command evolution**: Compare command patterns over time (via git history)
- **Usage patterns**: Identify which commands are commonly chained
- **Complexity metrics**: Identify overly complex commands that might need splitting
- **Convention compliance**: Check if new commands follow established patterns
- **Documentation quality**: Assess completeness of command documentation

## Example Interactions

### Example 1: General Help
```
User: /analyze_commands
Assistant: [Lists categories and asks what they want to know]
User: List all commands
Assistant: [Provides categorized table with descriptions]
```

### Example 2: Task-Specific Help
```
User: /analyze_commands I need to understand how authentication works in the codebase
Assistant: Based on your request, I recommend: `/research_codebase`
[Provides explanation and workflow]
```

### Example 3: Pattern Analysis
```
User: /analyze_commands show me how commands use the thoughts directory
Assistant: [Spawns tasks to analyze patterns, presents findings]
```

## Remember

- **Be helpful**: The goal is to make commands discoverable and understandable
- **Be accurate**: Always read actual files rather than assuming
- **Be practical**: Focus on helping users accomplish their actual goals
- **Be educational**: Explain the "why" behind command design choices
