# Everything Claude Code — Harness Setup Guide

A complete reference for your Claude Code harness setup, how to maintain it,
and how to spin up new projects using it.

---

## What You Have Installed

### Global (per machine, set up once)

```
C:\Users\Abbas\.claude\
  plugins\
    everything-claude-code\     ← 28 agents, 116 skills, 59 commands, hooks
  settings.json                 ← global Claude Code settings

C:\Users\Abbas\.claude-harness\ ← source repo, kept for rules copying
  rules\
    common\                     ← language-agnostic rules
    typescript\                 ← JS/Node.js/React/Next.js rules
    python\                     ← Python rules
    golang\                     ← Go rules
    swift\                      ← Swift/iOS rules
    php\                        ← PHP rules
```

### Per Project

```
your-project\
  .claude\
    settings.json               ← created by Claude Code automatically
    rules\                      ← copied from .claude-harness for this project
      common\
      typescript\               ← or whatever stack this project uses
  CLAUDE.md                     ← project context + overrides
```

---

## One-Time Machine Setup

These steps only ever need to be done once per machine.

### Step 1 — Install the plugin (inside Claude Code CLI)

```
/plugin marketplace add affaan-m/everything-claude-code
/plugin install everything-claude-code@everything-claude-code
```

### Step 2 — Clone the harness as your rules source (terminal)

```powershell
cd %USERPROFILE%
git clone https://github.com/affaan-m/everything-claude-code.git .claude-harness
cd .claude-harness
npm install
```

### Step 3 — Add token optimization settings

Open `C:\Users\Abbas\.claude\settings.json` and add:

```json
{
  "model": "sonnet",
  "env": {
    "MAX_THINKING_TOKENS": "10000",
    "CLAUDE_AUTOCOMPACT_PCT_OVERRIDE": "50",
    "CLAUDE_CODE_SUBAGENT_MODEL": "haiku"
  }
}
```

---

## Starting a New Project

Do this every time you start a new project.

### Step 1 — Create the rules folder

```powershell
cd path\to\your-new-project
mkdir .claude\rules
```

### Step 2 — Copy rules for your stack

Always copy common rules. Then add your language-specific rules.

```powershell
# always
xcopy %USERPROFILE%\.claude-harness\rules\common\* .claude\rules\ /E /I

# pick your stack
xcopy %USERPROFILE%\.claude-harness\rules\typescript\* .claude\rules\ /E /I   # JS/TS/Node/React
xcopy %USERPROFILE%\.claude-harness\rules\python\* .claude\rules\ /E /I       # Python
xcopy %USERPROFILE%\.claude-harness\rules\golang\* .claude\rules\ /E /I       # Go
xcopy %USERPROFILE%\.claude-harness\rules\swift\* .claude\rules\ /E /I        # Swift/iOS
xcopy %USERPROFILE%\.claude-harness\rules\php\* .claude\rules\ /E /I          # PHP
```

### Step 3 — Add a CLAUDE.md to the project root

Create `CLAUDE.md` at your project root. See the CLAUDE.md template section
below for the full file. At minimum update:

- Project name
- Stack details
- Any project-specific overrides

### Step 4 — Verify in Claude Code CLI

```
/plugin list everything-claude-code@everything-claude-code
```

You should see all agents, skills and commands listed. The harness is active.

---

## Available Stacks

| Stack flag | Covers |
|---|---|
| `typescript` | JavaScript, TypeScript, Node.js, React, Next.js |
| `python` | Python, Django, FastAPI |
| `golang` | Go |
| `swift` | Swift, iOS |
| `php` | PHP, Laravel |

For a project using multiple languages copy multiple stacks:

```powershell
xcopy %USERPROFILE%\.claude-harness\rules\common\* .claude\rules\ /E /I
xcopy %USERPROFILE%\.claude-harness\rules\typescript\* .claude\rules\ /E /I
xcopy %USERPROFILE%\.claude-harness\rules\python\* .claude\rules\ /E /I
```

---

## Key Commands Available

Once the harness is active these commands are available in any project:

| Command | When to use |
|---|---|
| `/plan` | Before starting any new feature |
| `/tdd` | When writing new functionality |
| `/code-review` | Before committing |
| `/build-fix` | When a build fails |
| `/security-scan` | Before shipping |
| `/refactor-clean` | Remove dead code |
| `/e2e` | Generate end-to-end tests |
| `/update-docs` | Sync documentation |
| `/learn` | Extract patterns from current session |
| `/checkpoint` | Save verification state |

Note: after plugin install commands are namespaced as
`/everything-claude-code:plan` etc. If you want the short form use the
manual install option instead (see Changing the Harness below).

---

## Changing the Harness

### Editing rules for a specific project

Rules are just markdown files. Open and edit directly:

```powershell
# see what rules are active
dir .claude\rules

# edit a rule
notepad .claude\rules\coding-style.md
```

Changes take effect on the next Claude Code session.

### Adding a custom rule to a project

Create a new `.md` file in `.claude\rules\`:

```powershell
notepad .claude\rules\my-project-rules.md
```

Example content:

```markdown
# My Project Rules

- Always use arrow functions
- Never use var, always const or let
- Prefer async/await over .then() chains
- All API responses must be typed
```

### Removing rules from a project

```powershell
# remove a specific rule file
del .claude\rules\coding-style.md

# remove all rules for a language
rmdir /s /q .claude\rules\python
```

### Editing global rules (affects all projects)

Only do this if you want a rule to apply everywhere. Edit directly in
`~\.claude-harness\rules\` then re-copy to any projects that need it.

### Updating the harness to the latest version

```powershell
cd %USERPROFILE%\.claude-harness
git pull
npm install
```

Then re-copy rules to any active projects:

```powershell
cd path\to\your-project
xcopy %USERPROFILE%\.claude-harness\rules\common\* .claude\rules\ /E /I /Y
xcopy %USERPROFILE%\.claude-harness\rules\typescript\* .claude\rules\ /E /I /Y
```

The `/Y` flag overwrites existing files without prompting.

### Updating the plugin

Inside Claude Code CLI:

```
/plugin update everything-claude-code@everything-claude-code
```

---

## Managing Context Window

The context window is a shared resource. Keep it lean.

### MCPs — biggest impact

Each enabled MCP eats into your 200k token window. Keep under 10 active
per project. Disable unused ones in `.claude\settings.json`:

```json
{
  "disabledMcpServers": ["supabase", "railway", "vercel"]
}
```

Check what is active:

```
/plugins
```

### Rules — small impact but worth knowing

Each rule file adds tokens. Only copy the rules you actually need for the
project stack. Delete rules you don't need:

```powershell
rmdir /s /q .claude\rules\python   # remove if not a Python project
```

### Model routing

The token optimization settings in `settings.json` route most tasks to
Sonnet (cheaper, fast) and only use Opus when you explicitly ask:

```
/model opus    # switch to Opus for complex architecture decisions
/model sonnet  # switch back
```

---

## Project CLAUDE.md Template

Use this as the starting point for every new project. Update the Project
Context section for each project. The engineering rules sections stay the
same across all projects.

---

```markdown
# CLAUDE.md

# AI Agent Engineering Rules

This document defines mandatory engineering and explanation rules for any AI
agent operating in this repository.

These rules apply to:
- Claude Code
- Cursor AI
- automated coding agents
- LLM-based development tools
- any system capable of modifying or analyzing this codebase

---

# Project Context

## Stack
- Runtime: Node.js
- Language: JavaScript
- Package manager: npm

## Harness
This project uses the everything-claude-code harness (globally installed).
Agents, skills, commands and hooks are managed globally — do not redefine them here.

## Active Rules
- .claude/rules/common/ — universal engineering principles
- .claude/rules/typescript/ — JS/Node.js patterns

## Preferred Commands
- /plan — before starting any new feature
- /tdd — when writing new functionality
- /code-review — before committing
- /build-fix — when builds fail
- /security-scan — before shipping

---

# 1. Atomic Engineering Rules

The following rules are **atomic** — they must always be applied, no
exceptions for small scope.

---

## 1.1 Explain Data Flow

Always explain how information moves through the system.

Example structure:

​```
User input
→ input processing / validation
→ service / business logic
→ data transformation
→ persistence layer
→ response propagation
​```

---

## 1.2 Explain Control Flow

Always explain how execution moves through the system.

Example:

​```
User action
→ event / request handler triggered
→ service processes request
→ persistence operation performed
→ result returned and rendered
​```

---

## 1.3 Explain Dependency Flow

Always explain how modules depend on one another — including:
- direct dependencies
- indirect dependencies
- dependency direction
- why the dependency structure was chosen

Example:

​```
Controller → Service → Repository → Database Adapter
​```

---

## 1.4 Explain Design Patterns

When a design pattern is used, explain:
1. The pattern used
2. Why it was chosen
3. What problem it solves
4. How it appears in the code

---

## 1.5 Use Appropriate Design Patterns

Prefer well-known patterns when they solve a real design problem. Patterns
should improve modularity, maintainability, testability, or extensibility.

Avoid unnecessary abstraction.

---

# 2. Feature-First Explanation

Never explain systems purely file-by-file.

Instead, use a **feature-first approach**:
1. Identify the feature
2. Explain its purpose
3. Trace its full implementation across all relevant files

Example:

​```
Feature: User Authentication

Login UI
→ Auth API endpoint
→ Auth service / validation
→ Database lookup
→ Token generation
→ Response to client
​```

---

# 3. Granular Technical Explanation

Provide deep explanations of system components, including:
- what each module does
- why the abstraction exists
- how modules interact
- what data they exchange

Avoid vague descriptions.

---

# 4. Architecture Decisions

When building or analyzing a system, explicitly state:
- Architecture chosen
- Why it was chosen
- Tradeoffs involved
- Alternatives considered

Common architectures include: Layered, Clean, Hexagonal, Microservices,
Event-Driven — but the chosen architecture should match the project's actual
needs, not be applied by default.

---

# 5. Tech Stack Explanation

Explain the full technology stack, including:
- Language, frameworks, libraries
- Databases and infrastructure
- Why each technology was chosen
- How each integrates with the rest of the stack

---

# 6. Feature Implementation Walkthrough

For every implemented feature, trace the complete execution lifecycle:
1. User interaction or system trigger
2. System processing
3. Data persistence (if applicable)
4. Result propagation

**This walkthrough must use an ASCII Art Flow Diagram (see Section 14).
Plain text traces like `A → B → C` are not sufficient.**

---

# 7. Module Design Principles

Design modules using:
- **High Cohesion** — each module has a clearly defined, single role
- **Loose Coupling** — modules interact through well-defined interfaces
- **Dependency Inversion** — high-level modules don't depend on low-level implementations
- **Single Responsibility** — each component does one thing

---

# 8. Common Design Patterns

Use appropriate patterns when warranted:
- **Dependency Injection** — decouple implementations, improve testability
- **Repository Pattern** — separate data access from business logic
- **Factory Pattern** — encapsulate complex or variable object creation
- **Strategy Pattern** — swap algorithms or implementations at runtime
- **Observer Pattern** — decouple event producers from consumers
- **Facade Pattern** — simplify complex subsystems behind a clean interface

---

# 9. Architecture and Pattern Alignment

Align design patterns with the chosen architecture. Patterns should
reinforce — not contradict — the architectural boundaries of the system.

---

# 10. Code Explanation Requirements

When writing code, always explain:
- What the function or module does
- Why it exists
- How it interacts with other modules
- What design patterns are applied

---

# 11. System-Level Thinking

Always reason about scalability, maintainability, extensibility, and
testability — even when implementing small features.

---

# 12. Codebase Analysis

When analyzing existing code:
- Identify the architecture and design patterns in use
- Trace feature implementations across modules
- Flag architectural problems such as tight coupling, God classes, circular
  dependencies, or duplicated logic

---

# 13. Anti-Patterns to Avoid

- God Objects / Massive Controllers
- Hidden Dependencies
- Global Mutable State
- Over-Engineering
- Shallow "generate and move on" code without explanation

---

# 14. Required Explanation Format

Every system overview and feature breakdown must include the following
sections. Mermaid and other rendered diagram formats are prohibited — use
terminal-friendly ASCII art only.

---

### 1. System Overview

A concise summary of what the system or feature does.

---

### 2. ASCII Art Flow Diagram

A text-based visual using ASCII boxes and arrows.

Example:

​```
+---------------------------+
|        CLIENT             |
|  (UI / CLI / Consumer)    |
+---------------------------+
             |
             | User action / input
             v
+---------------------------+
|      ENTRY POINT          |
|  (Route / Command / Handler)|
+---------------------------+
             |
             v
+---------------------------+
|    SERVICE / USE CASE     |
|  (Business Logic)         |
+---------------------------+
             |
             v
+---------------------------+
|   PERSISTENCE / ADAPTER   |
|  (DB / API / File System) |
+---------------------------+
             |
             v
+---------------------------+
|        RESPONSE           |
|  (Output / UI Update)     |
+---------------------------+
​```

---

### 3. State-by-State Architectural Rationale

For every node in the diagram:
- **Data Flow:** What happens to the data at this step
- **Design Choice:** What pattern or concept is applied
- **Why:** The technical justification for this choice

---

### 4. Tech Stack Context

How the technologies used support this specific feature.

---

### 5. Design Patterns Applied

A summary of patterns used across the flow.

---

### 6. Tradeoffs

What is gained and what is lost by the chosen approach.

---

# 15. Objective

The goal is not just code generation.

Every response should:
- explain systems clearly
- design maintainable architectures
- trace feature implementations across files
- teach how the system works

Explanation quality should be comparable to a **senior engineer performing a
full system walkthrough**.

---

# 16. Overrides to Harness Defaults

- Plain JavaScript, no TypeScript compilation
- CommonJS modules (require/module.exports)
- No console.logs in committed code
- Conventional commits (feat:, fix:, chore:)
- Tests required for new features
```

---

## Quick Reference

### New project checklist

```
[ ] cd to project root
[ ] mkdir .claude\rules
[ ] xcopy common rules
[ ] xcopy stack-specific rules
[ ] create CLAUDE.md from template
[ ] update Project Context section in CLAUDE.md
[ ] open Claude Code and verify with /plugin list
```

### File locations

| What | Where |
|---|---|
| Plugin (agents, skills, commands) | `%USERPROFILE%\.claude\plugins\` |
| Rules source | `%USERPROFILE%\.claude-harness\rules\` |
| Global settings | `%USERPROFILE%\.claude\settings.json` |
| Project rules | `your-project\.claude\rules\` |
| Project context | `your-project\CLAUDE.md` |

### When something goes wrong

| Problem | Fix |
|---|---|
| Commands not found | Run `/plugin list everything-claude-code@everything-claude-code` to verify install |
| Context window shrinking | Run `/plugins` and disable unused MCPs |
| Rules not being followed | Check `.claude\rules\` exists in project root |
| Hooks not firing | Check Claude Code version is v2.1.0+ with `claude --version` |
| Harness out of date | `cd %USERPROFILE%\.claude-harness && git pull && npm install` |

## Repo specific claude file 


# CLAUDE.md

# AI Agent Engineering Rules

This document defines mandatory engineering and explanation rules for any AI
agent operating in this repository.

These rules apply to:
- Claude Code
- Cursor AI
- automated coding agents
- LLM-based development tools
- any system capable of modifying or analyzing this codebase

---

# Project Context

## Stack
- Runtime: Node.js
- Language: JavaScript
- Package manager: npm

## Harness
This project uses the everything-claude-code harness (globally installed).
Agents, skills, commands and hooks are managed globally — do not redefine them here.

## Active Rules
- .claude/rules/common/ — universal engineering principles
- .claude/rules/typescript/ — JS/Node.js patterns

## Preferred Commands
- /plan — before starting any new feature
- /tdd — when writing new functionality
- /code-review — before committing
- /build-fix — when builds fail
- /security-scan — before shipping

---

# 1. Atomic Engineering Rules

The following rules are **atomic** — they must always be applied, no
exceptions for small scope.

---

## 1.1 Explain Data Flow

Always explain how information moves through the system.

Example structure:

```
User input
→ input processing / validation
→ service / business logic
→ data transformation
→ persistence layer
→ response propagation
```

---

## 1.2 Explain Control Flow

Always explain how execution moves through the system.

Example:

```
User action
→ event / request handler triggered
→ service processes request
→ persistence operation performed
→ result returned and rendered
```

---

## 1.3 Explain Dependency Flow

Always explain how modules depend on one another — including:
- direct dependencies
- indirect dependencies
- dependency direction
- why the dependency structure was chosen

Example:

```
Controller → Service → Repository → Database Adapter
```

---

## 1.4 Explain Design Patterns

When a design pattern is used, explain:
1. The pattern used
2. Why it was chosen
3. What problem it solves
4. How it appears in the code

---

## 1.5 Use Appropriate Design Patterns

Prefer well-known patterns when they solve a real design problem. Patterns
should improve modularity, maintainability, testability, or extensibility.

Avoid unnecessary abstraction.

---

# 2. Feature-First Explanation

Never explain systems purely file-by-file.

Instead, use a **feature-first approach**:
1. Identify the feature
2. Explain its purpose
3. Trace its full implementation across all relevant files

Example:

```
Feature: User Authentication

Login UI
→ Auth API endpoint
→ Auth service / validation
→ Database lookup
→ Token generation
→ Response to client
```

---

# 3. Granular Technical Explanation

Provide deep explanations of system components, including:
- what each module does
- why the abstraction exists
- how modules interact
- what data they exchange

Avoid vague descriptions.

---

# 4. Architecture Decisions

When building or analyzing a system, explicitly state:
- Architecture chosen
- Why it was chosen
- Tradeoffs involved
- Alternatives considered

Common architectures include: Layered, Clean, Hexagonal, Microservices,
Event-Driven — but the chosen architecture should match the project's actual
needs, not be applied by default.

---

# 5. Tech Stack Explanation

Explain the full technology stack, including:
- Language, frameworks, libraries
- Databases and infrastructure
- Why each technology was chosen
- How each integrates with the rest of the stack

---

# 6. Feature Implementation Walkthrough

For every implemented feature, trace the complete execution lifecycle:
1. User interaction or system trigger
2. System processing
3. Data persistence (if applicable)
4. Result propagation

**This walkthrough must use an ASCII Art Flow Diagram (see Section 14).
Plain text traces like `A → B → C` are not sufficient.**

---

# 7. Module Design Principles

Design modules using:
- **High Cohesion** — each module has a clearly defined, single role
- **Loose Coupling** — modules interact through well-defined interfaces
- **Dependency Inversion** — high-level modules don't depend on low-level implementations
- **Single Responsibility** — each component does one thing

---

# 8. Common Design Patterns

Use appropriate patterns when warranted:
- **Dependency Injection** — decouple implementations, improve testability
- **Repository Pattern** — separate data access from business logic
- **Factory Pattern** — encapsulate complex or variable object creation
- **Strategy Pattern** — swap algorithms or implementations at runtime
- **Observer Pattern** — decouple event producers from consumers
- **Facade Pattern** — simplify complex subsystems behind a clean interface

---

# 9. Architecture and Pattern Alignment

Align design patterns with the chosen architecture. Patterns should
reinforce — not contradict — the architectural boundaries of the system.

---

# 10. Code Explanation Requirements

When writing code, always explain:
- What the function or module does
- Why it exists
- How it interacts with other modules
- What design patterns are applied

---

# 11. System-Level Thinking

Always reason about scalability, maintainability, extensibility, and
testability — even when implementing small features.

---

# 12. Codebase Analysis

When analyzing existing code:
- Identify the architecture and design patterns in use
- Trace feature implementations across modules
- Flag architectural problems such as tight coupling, God classes, circular
  dependencies, or duplicated logic

---

# 13. Anti-Patterns to Avoid

- God Objects / Massive Controllers
- Hidden Dependencies
- Global Mutable State
- Over-Engineering
- Shallow "generate and move on" code without explanation

---

# 14. Required Explanation Format

Every system overview and feature breakdown must include the following
sections. Mermaid and other rendered diagram formats are prohibited — use
terminal-friendly ASCII art only.

---

### 1. System Overview

A concise summary of what the system or feature does.

---

### 2. ASCII Art Flow Diagram

A text-based visual using ASCII boxes and arrows.

Example:

```
+---------------------------+
|        CLIENT             |
|  (UI / CLI / Consumer)    |
+---------------------------+
             |
             | User action / input
             v
+---------------------------+
|      ENTRY POINT          |
|  (Route / Command / Handler)|
+---------------------------+
             |
             v
+---------------------------+
|    SERVICE / USE CASE     |
|  (Business Logic)         |
+---------------------------+
             |
             v
+---------------------------+
|   PERSISTENCE / ADAPTER   |
|  (DB / API / File System) |
+---------------------------+
             |
             v
+---------------------------+
|        RESPONSE           |
|  (Output / UI Update)     |
+---------------------------+
```

---

### 3. State-by-State Architectural Rationale

For every node in the diagram:
- **Data Flow:** What happens to the data at this step
- **Design Choice:** What pattern or concept is applied
- **Why:** The technical justification for this choice

---

### 4. Tech Stack Context

How the technologies used support this specific feature.

---

### 5. Design Patterns Applied

A summary of patterns used across the flow.

---

### 6. Tradeoffs

What is gained and what is lost by the chosen approach.

---

# 15. Objective

The goal is not just code generation.

Every response should:
- explain systems clearly
- design maintainable architectures
- trace feature implementations across files
- teach how the system works

Explanation quality should be comparable to a **senior engineer performing a
full system walkthrough**.

---

# 16. Overrides to Harness Defaults

- Plain JavaScript, no TypeScript compilation
- CommonJS modules (require/module.exports)
- No console.logs in committed code
- Conventional commits (feat:, fix:, chore:)
- Tests required for new features


