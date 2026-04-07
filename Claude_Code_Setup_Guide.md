# Claude Code — Complete Setup & Usage Guide
> **For Senior Software Engineers & Web Developers**  
> Shakil Uddin · Senior Software Engineer · Dhaka, Bangladesh  
> Stack: Laravel · .NET · React · TypeScript · Node.js · Docker  
> `Version 1.1 | April 2026 | Windows 10 | Git Bash`

---

## Table of Contents

1. [Prerequisites & System Requirements](#1-prerequisites--system-requirements)
2. [Installing Claude Code (Global)](#2-installing-claude-code-global)
3. [Windows PATH Configuration](#3-windows-path-configuration)
4. [Git Bash Configuration](#4-git-bash-configuration)
5. [Global Folder Structure](#5-global-folder-structure)
6. [CLAUDE.md — Global Preferences File](#6-claudemd--global-preferences-file)
7. [Installing Official Plugins](#7-installing-official-plugins)
8. [Installing Community Plugins & Skills](#8-installing-community-plugins--skills)
9. [LSP Setup (Code Intelligence)](#9-lsp-setup-code-intelligence)
10. [Settings.json Configuration](#10-settingsjson-configuration)
11. [MCP Server Setup](#11-mcp-server-setup)
12. [Plugin Scope Reference](#12-plugin-scope-reference)
13. [All Skills & Plugins Master List](#13-all-skills--plugins-master-list)
14. [Using Skills Inside Claude Code](#14-using-skills-inside-claude-code)
15. [PhpStorm Integration](#15-phpstorm-integration)
16. [New Machine Migration Checklist](#16-new-machine-migration-checklist)
17. [Troubleshooting](#17-troubleshooting)

---

## 1. Prerequisites & System Requirements

### Required Software

| Software | Version | Download | Required? |
|---|---|---|---|
| Node.js | 18+ LTS | nodejs.org | ✅ Required |
| Git Bash | Latest | git-scm.com | ✅ Required |
| npm | 9+ | Comes with Node.js | ✅ Required |
| .NET SDK | 8.0 LTS | dotnet.microsoft.com | ✅ For .NET dev |
| PHP | 8.3+ | php.net or Laravel Herd | ✅ For Laravel dev |
| Composer | Latest | getcomposer.org | ✅ For Laravel dev |
| Docker Desktop | Latest | docker.com | ✅ Recommended |

### Anthropic Account Required

- Claude **Pro**, **Max**, or **Team** subscription — OR an Anthropic API key
- Sign up at: [claude.ai](https://claude.ai) or [console.anthropic.com](https://console.anthropic.com)

### Verify Node.js Installation

```bash
# Open Git Bash and run:
node --version    # Should show v18.x.x or higher
npm --version     # Should show 9.x.x or higher
git --version     # Should show 2.x.x or higher
```

---

## 2. Installing Claude Code (Global)

### Install via npm

```bash
# Install Claude Code globally
npm install -g @anthropic-ai/claude-code

# Verify installation
claude --version

# Login to your Anthropic account
claude
# Follow the prompts to authenticate
```

> 💡 **TIP:** Always install globally with `-g` so `claude` is available from any folder on your machine.

### First Time Login

1. Run: `claude` in Git Bash
2. A browser window opens automatically
3. Log in with your Claude / Anthropic account
4. Return to Git Bash — you are now authenticated
5. Credentials are saved to: `~/.claude/.credentials.json`

### Update Claude Code

```bash
# Update to latest version
npm update -g @anthropic-ai/claude-code

# Check current version
claude --version
```

### Starting Claude Code in a Project

```bash
# Navigate to your project folder
cd ~/projects/my-laravel-project

# Start Claude Code
claude

# Start with all permissions pre-approved (use for trusted projects)
claude --dangerously-skip-permissions

# Exit Claude Code session
/exit
# OR press Ctrl+C
```

---

## 3. Windows PATH Configuration

> ⚠️ **IMPORTANT for Windows users:** The most reliable way to fix `claude: command not found` or missing language server binaries is to add paths via **Windows System Environment Variables**, not just `.bashrc`. Git Bash reads the Windows PATH at startup.

### Add to Windows System Environment Variables

1. Press `Win + R` → type `sysdm.cpl` → press Enter
2. Click **Advanced** tab → **Environment Variables**
3. Under **System variables**, select **Path** → click **Edit**
4. Click **New** and add each path below:

```
C:\Users\Shakil Uddin\AppData\Roaming\npm
C:\Users\Shakil Uddin\AppData\Roaming\npm\node_modules\.bin
C:\Users\Shakil Uddin\.dotnet\tools
C:\Users\Shakil Uddin\AppData\Roaming\Composer\vendor\bin
```

> Replace `Shakil Uddin` with your actual Windows username.

5. Click **OK** on all dialogs
6. **Restart Git Bash** (close all windows — not just the tab)

### Verify PATH is Working

```bash
# After restarting Git Bash:
which claude
which typescript-language-server
which intelephense
which csharp-ls
```

---

## 4. Git Bash Configuration

### Open and Edit `~/.bashrc`

```bash
# Open ~/.bashrc in Notepad
notepad ~/.bashrc

# Windows file location:
# C:\Users\Shakil Uddin\.bashrc
```

### Add These Lines to `~/.bashrc`

```bash
# ═══════════════════════════════════════════
# Claude Code Configuration
# ═══════════════════════════════════════════

# Enable LSP (Language Server Protocol) — IDE-like code intelligence
export ENABLE_LSP_TOOL=1

# Optional: Set default model
# export ANTHROPIC_MODEL=claude-sonnet-4-6

# ═══════════════════════════════════════════
# Useful Aliases
# ═══════════════════════════════════════════

# Start Claude Code in current directory
alias cc='claude'

# Start Claude Code skipping permission prompts (trusted projects only)
alias ccf='claude --dangerously-skip-permissions'

# Open Claude settings folder
alias ccs='cd ~/.claude && ls -la'

# List all installed plugins
alias ccpl='claude plugin list'

# ═══════════════════════════════════════════
# PATH Configuration
# ═══════════════════════════════════════════

# npm global packages (Claude Code, language servers)
export PATH="$PATH:$HOME/AppData/Roaming/npm"

# .NET tools
export PATH="$PATH:$HOME/.dotnet/tools"

# Composer global packages (Laravel)
export PATH="$PATH:$HOME/AppData/Roaming/Composer/vendor/bin"
```

### Reload Git Bash

```bash
# Apply changes without restarting terminal
source ~/.bashrc

# Verify LSP is set
echo $ENABLE_LSP_TOOL    # Should print: 1
```

---

## 5. Global Folder Structure

### The `~/.claude` Directory

Everything Claude Code needs globally lives in `~/.claude`  
(`C:\Users\Shakil Uddin\.claude` on Windows).

```
C:\Users\Shakil Uddin\.claude\
├── CLAUDE.md               ← Global AI preferences (loads every session)
├── settings.json           ← Claude Code config (plugins, env vars)
├── .credentials.json       ← Auth tokens (auto-managed, don't edit)
├── keybindings.json        ← Custom keyboard shortcuts
│
├── skills\                 ← Global skills (available in ALL projects)
│   ├── frontend-design\
│   │   └── SKILL.md
│   └── landing-page\
│       └── SKILL.md
│
├── commands\               ← Global slash commands
│   └── my-command.md
│
├── agents\                 ← Global subagents
│   └── my-agent.md
│
├── plugins\                ← Installed plugin cache
│   └── cache\
│       └── claude-plugins-official\
│
├── tasks\                  ← Claude task lists (global only)
├── sessions\               ← Session history
└── projects\               ← Per-project auto-memory
```

### Create All Required Folders

```bash
# Run this in Git Bash to create your full structure
mkdir -p ~/.claude/skills/frontend-design
mkdir -p ~/.claude/skills/landing-page
mkdir -p ~/.claude/commands
mkdir -p ~/.claude/agents

# Verify the structure
ls -la ~/.claude/
```

### Project-Level Structure

Inside each project, local config overrides global:

```
my-project\
├── .claude\
│   ├── CLAUDE.md               ← Project-specific instructions
│   ├── settings.json           ← Shared team settings (commit to git)
│   ├── settings.local.json     ← Your personal overrides (gitignored)
│   ├── skills\                 ← Project-only skills
│   ├── commands\               ← Project-only commands
│   └── agents\                 ← Project-only agents
└── .mcp.json                   ← MCP server config for this project
```

---

## 6. CLAUDE.md — Global Preferences File

### What Is CLAUDE.md?

`CLAUDE.md` is automatically loaded into every Claude Code session. It tells Claude who you are, your tech stack, code standards, and available skills — without repeating yourself every time.

### Open the File

```bash
# Open in Notepad
notepad ~/.claude/CLAUDE.md

# Windows path:
# C:\Users\Shakil Uddin\.claude\CLAUDE.md
```

### Full CLAUDE.md Content

```markdown
# Shakil Uddin — Global Claude Configuration
# Senior Software Engineer | Dhaka, Bangladesh
# Stack: Laravel · .NET · React · TypeScript · DevOps

## IDENTITY & CONTEXT
I am a Senior Software Engineer and IT company founder.
Primary backend stacks: Laravel (PHP 8.3+) and .NET 8/9 (C#).
Frontend: React, TypeScript, Tailwind CSS, Next.js.
I build client projects for small businesses in Bangladesh.

## LARAVEL — INDUSTRY BEST PRACTICES
- Laravel 11+ with PHP 8.3+
- declare(strict_types=1) in every file
- Return types on ALL methods — never omit
- Use Service classes for business logic (App\Services\)
- Use Action classes for single-responsibility (App\Actions\)
- Use Form Requests for ALL validation
- Use API Resources for ALL API responses — never return raw models
- Eager load always: use with() — treat N+1 as a bug
- Hash passwords with bcrypt — never md5/sha1
- PHPStan level 8 — treat all errors as blockers
- Pest PHP for testing — target 80%+ coverage on business logic

## .NET — INDUSTRY BEST PRACTICES
- .NET 8 LTS preferred, C# 12 features
- Nullable reference types ENABLED in all projects
- Clean Architecture: Domain > Application > Infrastructure > Presentation
- CQRS pattern with MediatR — Commands mutate, Queries read
- FluentValidation for all input validation
- async/await everywhere — no .Result or .Wait() ever
- Use AsNoTracking() for all read-only EF Core queries
- xUnit for testing, FluentAssertions for readable assertions
- Testcontainers for DB integration tests

## FRONTEND
- React 19 + TypeScript 5 strict mode
- Tailwind CSS v4 — utility-first
- Zod for schema validation
- TanStack Query for server state
- Zustand for client state
- Shadcn/ui as component base

## UNIVERSAL CODE RULES
- camelCase: JS/TS/C# | snake_case: PHP | PascalCase: all classes
- Database columns: snake_case always
- API routes: kebab-case (/user-profiles not /userProfiles)
- Conventional Commits: feat|fix|docs|style|refactor|test|chore
- Never commit .env files, secrets, or credentials
- Never use raw queries when ORM supports it
- Always handle errors — never silently swallow exceptions

## AVAILABLE GLOBAL SKILLS
- frontend-design    → bold production-grade UI/UX
- landing-page       → high-converting marketing pages
- senior-architect   → system architecture decisions
- code-reviewer      → deep code review
- senior-backend     → backend engineering
- senior-frontend    → frontend engineering
- senior-devops      → Docker, CI/CD, infrastructure
- senior-security    → security auditing
- cto-advisor        → technical leadership
```

---

## 7. Installing Official Plugins

> 📌 **NOTE:** The official Anthropic marketplace (`claude-plugins-official`) is built into Claude Code automatically — you do NOT need to add it. Just install directly.

Run these commands in Git Bash **OUTSIDE** of any Claude Code session:

```bash
# ═══════════════════════════════════════════
# OFFICIAL ANTHROPIC PLUGINS
# Install once — available in all projects
# ═══════════════════════════════════════════

# UI/UX and frontend design guidance
claude plugin install frontend-design@claude-plugins-official

# Automated multi-agent PR code review
claude plugin install code-review@claude-plugins-official

# Guided feature development workflow
claude plugin install feature-dev@claude-plugins-official

# Git commit message conventions
claude plugin install commit-commands@claude-plugins-official

# Security vulnerability scanning
claude plugin install security-guidance@claude-plugins-official
```

### Verify Installation

```bash
# List all installed plugins
claude plugin list
# You should see each plugin with status: enabled (scope: user)
```

### Plugin Management Commands

| Command | What It Does |
|---|---|
| `claude plugin list` | List all installed plugins and their status |
| `claude plugin install <name>` | Install a plugin globally (user scope) |
| `claude plugin install <name> --scope project` | Install for current project only |
| `claude plugin install <name> --scope local` | Install for you in current project only |
| `claude plugin disable <name>` | Disable without uninstalling |
| `claude plugin enable <name>` | Re-enable a disabled plugin |
| `claude plugin remove <name>` | Completely uninstall a plugin |
| `claude plugin update --all` | Update all plugins to latest version |
| `claude plugin marketplace list` | List all registered marketplaces |

---

## 8. Installing Community Plugins & Skills

### Method 1 — npx ai-agent-skills (Recommended)

Installs skills directly into `~/.claude/skills/` — the most reliable method.

```bash
# ═══════════════════════════════════════════
# ENGINEERING CORE
# ═══════════════════════════════════════════

npx ai-agent-skills install alirezarezvani/claude-skills/engineering-team/senior-architect --agent claude
npx ai-agent-skills install alirezarezvani/claude-skills/engineering-team/senior-backend --agent claude
npx ai-agent-skills install alirezarezvani/claude-skills/engineering-team/senior-frontend --agent claude
npx ai-agent-skills install alirezarezvani/claude-skills/engineering-team/senior-fullstack --agent claude
npx ai-agent-skills install alirezarezvani/claude-skills/engineering-team/code-reviewer --agent claude
npx ai-agent-skills install alirezarezvani/claude-skills/engineering-team/senior-qa --agent claude
npx ai-agent-skills install alirezarezvani/claude-skills/engineering-team/senior-devops --agent claude
npx ai-agent-skills install alirezarezvani/claude-skills/engineering-team/senior-security --agent claude
npx ai-agent-skills install alirezarezvani/claude-skills/engineering-team/senior-secops --agent claude

# ═══════════════════════════════════════════
# CLOUD & DATABASE
# ═══════════════════════════════════════════

npx ai-agent-skills install alirezarezvani/claude-skills/engineering-team/aws-solution-architect --agent claude
npx ai-agent-skills install alirezarezvani/claude-skills/engineering-team/senior-dba --agent claude
npx ai-agent-skills install alirezarezvani/claude-skills/engineering-team/senior-sre --agent claude

# ═══════════════════════════════════════════
# PROJECT MANAGEMENT
# ═══════════════════════════════════════════

npx ai-agent-skills install alirezarezvani/claude-skills/project-management/senior-pm --agent claude
npx ai-agent-skills install alirezarezvani/claude-skills/project-management/scrum-master --agent claude
npx ai-agent-skills install alirezarezvani/claude-skills/project-management/jira-expert --agent claude

# ═══════════════════════════════════════════
# C-LEVEL ADVISORY
# ═══════════════════════════════════════════

npx ai-agent-skills install alirezarezvani/claude-skills/c-level-advisor/cto-advisor --agent claude
npx ai-agent-skills install alirezarezvani/claude-skills/c-level-advisor/ceo-advisor --agent claude

# ═══════════════════════════════════════════
# MARKETING (for your IT company)
# ═══════════════════════════════════════════

npx ai-agent-skills install alirezarezvani/claude-skills/marketing-skill/content-creator --agent claude
npx ai-agent-skills install alirezarezvani/claude-skills/marketing-skill/marketing-strategy-pmm --agent claude
npx ai-agent-skills install alirezarezvani/claude-skills/marketing-skill/social-media-analyzer --agent claude
```

### Method 2 — Manual SKILL.md Creation

For custom skills specific to your workflow:

```bash
# Create skill directory
mkdir -p ~/.claude/skills/my-custom-skill

# Create the SKILL.md file
notepad ~/.claude/skills/my-custom-skill/SKILL.md
```

SKILL.md template:

```markdown
---
name: my-custom-skill
description: Describe when Claude should use this skill. Be specific about triggers.
---

## Instructions

When this skill is invoked, Claude should:

1. First step...
2. Second step...
```

---

## 9. LSP Setup (Code Intelligence)

### What Is LSP?

Language Server Protocol gives Claude the same code intelligence as VS Code: go-to-definition, find all references, type checking, and real-time error detection.

| Without LSP | With LSP |
|---|---|
| Grep search ~45 seconds | Semantic lookup ~50ms |
| May miss function references | Finds every usage exactly |
| No type awareness | Full type checking |
| Text matching only | Go-to-definition, find references |
| May break on rename | Knows every file that will break |

### Step 1 — Install LSP Plugins

```bash
# TypeScript / JavaScript
claude plugin install typescript-lsp@claude-plugins-official

# Python
claude plugin install pyright-lsp@claude-plugins-official

# PHP (Laravel)
claude plugin install php-lsp@claude-plugins-official

# Add community marketplace for C#
claude plugin marketplace add boostvolt/claude-code-lsps

# C# / .NET
claude plugin install omnisharp@claude-code-lsps
```

### Step 2 — Install Language Server Binaries

```bash
# TypeScript / JavaScript
npm install -g typescript-language-server typescript

# Python
npm install -g pyright

# PHP (Intelephense — best PHP language server)
npm install -g intelephense

# C# / .NET (requires .NET SDK installed)
dotnet tool install -g csharp-ls

# Verify binaries are in PATH
which typescript-language-server    # TypeScript
which pyright-langserver             # Python
which intelephense                   # PHP
which csharp-ls                      # C#
```

> ⚠️ **If `which` returns nothing:** the binary is not in PATH. Fix it via Windows System Environment Variables (see [Section 3](#3-windows-path-configuration)) and restart Git Bash.

### Step 3 — Enable LSP

Add to `~/.bashrc` (covered in Section 4):

```bash
export ENABLE_LSP_TOOL=1
```

Also add to `~/.claude/settings.json` (covered in Section 10).

### Step 4 — Test LSP is Working

```bash
# Open a project with Claude Code
cd ~/my-laravel-project
claude

# Inside session, ask Claude:
# "Go to the definition of the User model"
# "Find all references to the authenticate method"
# "Check for type errors in this file"

# If you see "LSP" in the response (not grep) — it is working!
```

---

## 10. Settings.json Configuration

### File Location

```bash
# Windows path:
# C:\Users\Shakil Uddin\.claude\settings.json

# Open in Notepad:
notepad ~/.claude/settings.json
```

### Complete settings.json

> ⚠️ **WARNING:** This is JSON format. Do NOT append to the file — **replace the entire content** with this.

```json
{
  "env": {
    "ENABLE_LSP_TOOL": "1"
  },
  "enabledPlugins": {
    "frontend-design@claude-plugins-official": true,
    "code-review@claude-plugins-official": true,
    "feature-dev@claude-plugins-official": true,
    "commit-commands@claude-plugins-official": true,
    "security-guidance@claude-plugins-official": true,
    "typescript-lsp@claude-plugins-official": true,
    "pyright-lsp@claude-plugins-official": true,
    "php-lsp@claude-plugins-official": true,
    "omnisharp@claude-code-lsps": true
  }
}
```

### Verify JSON is Valid

```bash
# Check file contents
cat ~/.claude/settings.json

# Validate JSON syntax (requires Node.js)
node -e "JSON.parse(require('fs').readFileSync(process.env.USERPROFILE+'/.claude/settings.json', 'utf8')); console.log('JSON valid')"
```

---

## 11. MCP Server Setup

MCP (Model Context Protocol) servers give Claude Code access to external tools — databases, APIs, browsers, and more.

### What Is `.mcp.json`?

Place `.mcp.json` in your project root to give Claude Code tools specific to that project. This file can be committed to git so your whole team gets the same tools.

### Example `.mcp.json` for a Laravel Project

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "."]
    },
    "mysql": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-mysql"],
      "env": {
        "MYSQL_HOST": "127.0.0.1",
        "MYSQL_PORT": "3306",
        "MYSQL_USER": "root",
        "MYSQL_PASSWORD": "your_password",
        "MYSQL_DATABASE": "your_db_name"
      }
    }
  }
}
```

### Useful MCP Servers

```bash
# Install MCP servers globally (optional — npx handles this automatically)

# Filesystem access
npm install -g @modelcontextprotocol/server-filesystem

# MySQL / MariaDB
npm install -g @modelcontextprotocol/server-mysql

# Fetch / HTTP requests
npm install -g @modelcontextprotocol/server-fetch

# GitHub integration
npm install -g @modelcontextprotocol/server-github
```

### Add GitHub MCP (useful for PR review)

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "your_token_here"
      }
    }
  }
}
```

> 💡 **TIP:** Never commit `.mcp.json` files that contain secrets. Use environment variables or a `.mcp.local.json` (gitignored) for credentials.

---

## 12. Plugin Scope Reference

### The Three Scopes

| Scope | Flag | Stored In | Available Where | Who Sees It |
|---|---|---|---|---|
| user (default) | none / `--scope user` | `~/.claude/plugins/` | ALL projects on this machine | Only you |
| project | `--scope project` | `./.claude/settings.json` | This project only | Whole team (committed to git) |
| local | `--scope local` | `./.claude/settings.local.json` | This project only | Only you (gitignored) |

### When to Use Each Scope

| Plugin/Skill | Scope | Reason |
|---|---|---|
| frontend-design | user | You want it everywhere |
| code-review | user | Personal preference across all projects |
| senior-architect | user | Always useful regardless of project |
| laravel-standards | project | Whole team follows Laravel rules |
| dotnet-analyzer | project | Team-wide, committed to git |
| my-debug-helper | local | Personal tool for this project only |

### Priority Order

When the same plugin exists at multiple scopes:

```
local > project > user
```

> 📌 **NOTE:** Project scope is committed to git. Teammates get those plugins automatically when they clone the repo.

---

## 13. All Skills & Plugins Master List

### Official Anthropic Plugins (`claude-plugins-official`)

| Plugin Name | Install Command | What It Does |
|---|---|---|
| frontend-design | `claude plugin install frontend-design@claude-plugins-official` | Bold production-grade UI/UX design |
| code-review | `claude plugin install code-review@claude-plugins-official` | Multi-agent automated PR review |
| feature-dev | `claude plugin install feature-dev@claude-plugins-official` | Guided feature development |
| commit-commands | `claude plugin install commit-commands@claude-plugins-official` | Git commit conventions |
| security-guidance | `claude plugin install security-guidance@claude-plugins-official` | Security vulnerability detection |
| typescript-lsp | `claude plugin install typescript-lsp@claude-plugins-official` | TypeScript/JS code intelligence |
| pyright-lsp | `claude plugin install pyright-lsp@claude-plugins-official` | Python code intelligence |
| php-lsp | `claude plugin install php-lsp@claude-plugins-official` | PHP code intelligence |

### Community Skills — Engineering (`alirezarezvani/claude-skills`)

| Skill Name | npx Command Suffix | What It Does |
|---|---|---|
| senior-architect | `engineering-team/senior-architect` | System design, architecture decisions |
| senior-backend | `engineering-team/senior-backend` | Backend engineering best practices |
| senior-frontend | `engineering-team/senior-frontend` | Frontend engineering best practices |
| senior-fullstack | `engineering-team/senior-fullstack` | Full-stack development patterns |
| code-reviewer | `engineering-team/code-reviewer` | Deep code quality review |
| senior-qa | `engineering-team/senior-qa` | Test strategy and QA automation |
| senior-devops | `engineering-team/senior-devops` | Docker, CI/CD, infrastructure |
| senior-security | `engineering-team/senior-security` | Security architecture and auditing |
| senior-secops | `engineering-team/senior-secops` | Security operations |
| aws-solution-architect | `engineering-team/aws-solution-architect` | AWS cloud architecture |
| senior-dba | `engineering-team/senior-dba` | Database design and optimization |
| senior-sre | `engineering-team/senior-sre` | Site reliability engineering |

### Community Skills — Management & Business

| Skill Name | npx Command Suffix | What It Does |
|---|---|---|
| senior-pm | `project-management/senior-pm` | Project management expertise |
| scrum-master | `project-management/scrum-master` | Agile/Scrum facilitation |
| jira-expert | `project-management/jira-expert` | Jira setup and best practices |
| cto-advisor | `c-level-advisor/cto-advisor` | Technical leadership guidance |
| ceo-advisor | `c-level-advisor/ceo-advisor` | Business strategy and leadership |
| content-creator | `marketing-skill/content-creator` | Content writing and strategy |
| marketing-strategy-pmm | `marketing-skill/marketing-strategy-pmm` | Product marketing strategy |
| social-media-analyzer | `marketing-skill/social-media-analyzer` | Social media analysis |

---

## 14. Using Skills Inside Claude Code

### 3 Ways to Invoke a Skill

#### Way 1 — Direct Slash Command

```
# Inside Claude Code session:
/code-review
/frontend-design
/senior-architect

# With arguments:
/code-review UserController.php
/frontend-design build a dark hero section for an IT company
```

#### Way 2 — Auto-Invocation (Claude decides)

Claude reads all skill descriptions and automatically picks the right one:

```
"Review this Laravel controller for security issues"
→ Claude auto-invokes: code-reviewer + senior-security

"Design a landing page for my IT company"
→ Claude auto-invokes: frontend-design + landing-page

"Help me architect a multi-tenant SaaS system"
→ Claude auto-invokes: senior-architect
```

#### Way 3 — Reload After Installing New Skills

```bash
# Inside Claude Code session — soft reload:
/reload-plugins

# If skills still not showing — exit and restart:
/exit
claude
```

### Useful In-Session Commands

| Command | What It Does |
|---|---|
| `/help` | List all available skills and commands |
| `/reload-plugins` | Soft reload skills without restarting |
| `/plugin` | Open plugin manager UI |
| `/exit` | Exit the Claude Code session |
| `/clear` | Clear conversation history |
| `Ctrl+C` | Force quit session |

---

## 15. PhpStorm Integration

### Run Claude Code from PhpStorm Terminal

PhpStorm's built-in terminal works with Claude Code once the PATH is set correctly via Windows System Environment Variables (Section 3).

```bash
# In PhpStorm Terminal (bottom panel):
cd your-project-root
claude
```

### Set Git Bash as Default Terminal in PhpStorm

1. Go to **File → Settings → Tools → Terminal**
2. Set **Shell path** to:
   ```
   C:\Program Files\Git\bin\bash.exe
   ```
3. Click **OK** and restart the terminal panel

### Configure PhpStorm to Open Claude Code Quickly

1. Go to **File → Settings → Tools → External Tools**
2. Click **+** to add a new tool:
   - **Name:** `Claude Code`
   - **Program:** `C:\Program Files\Git\bin\bash.exe`
   - **Arguments:** `-c "claude"`
   - **Working directory:** `$ProjectFileDir$`
3. Assign a keyboard shortcut if desired

---

## 16. New Machine Migration Checklist

Follow this checklist **in order** when setting up Claude Code on a new Windows machine.

### Phase 1 — Install Required Software

| # | Task | Command / Action | Done? |
|---|---|---|---|
| 1 | Install Node.js 18+ LTS | Download from nodejs.org | ☐ |
| 2 | Install Git Bash | Download from git-scm.com | ☐ |
| 3 | Install .NET 8 SDK | Download from dotnet.microsoft.com | ☐ |
| 4 | Install PHP 8.3+ | Download from php.net or use Herd | ☐ |
| 5 | Install Composer | Download from getcomposer.org | ☐ |
| 6 | Install Docker Desktop | Download from docker.com | ☐ |

### Phase 2 — Install Claude Code

| # | Task | Command | Done? |
|---|---|---|---|
| 1 | Install Claude Code globally | `npm install -g @anthropic-ai/claude-code` | ☐ |
| 2 | Verify installation | `claude --version` | ☐ |
| 3 | Authenticate | `claude` (follow browser prompts) | ☐ |

### Phase 3 — Configure Windows PATH

| # | Task | Action | Done? |
|---|---|---|---|
| 1 | Open System Environment Variables | `Win+R` → `sysdm.cpl` → Advanced | ☐ |
| 2 | Add npm path | `C:\Users\<you>\AppData\Roaming\npm` | ☐ |
| 3 | Add .NET tools path | `C:\Users\<you>\.dotnet\tools` | ☐ |
| 4 | Add Composer path | `C:\Users\<you>\AppData\Roaming\Composer\vendor\bin` | ☐ |
| 5 | Restart Git Bash | Close all windows | ☐ |

### Phase 4 — Configure Git Bash

| # | Task | Command | Done? |
|---|---|---|---|
| 1 | Create/edit `~/.bashrc` | `notepad ~/.bashrc` | ☐ |
| 2 | Add `ENABLE_LSP_TOOL=1` | `export ENABLE_LSP_TOOL=1` | ☐ |
| 3 | Add aliases and PATH entries | See Section 4 | ☐ |
| 4 | Reload bashrc | `source ~/.bashrc` | ☐ |

### Phase 5 — Create Global Folder Structure

| # | Task | Command | Done? |
|---|---|---|---|
| 1 | Create skills folders | `mkdir -p ~/.claude/skills/frontend-design` | ☐ |
| 2 | Create landing-page skill | `mkdir -p ~/.claude/skills/landing-page` | ☐ |
| 3 | Create commands folder | `mkdir -p ~/.claude/commands` | ☐ |
| 4 | Create agents folder | `mkdir -p ~/.claude/agents` | ☐ |

### Phase 6 — Create Config Files

| # | Task | File to Create | Done? |
|---|---|---|---|
| 1 | Create CLAUDE.md | `~/.claude/CLAUDE.md` (see Section 6) | ☐ |
| 2 | Create settings.json | `~/.claude/settings.json` (see Section 10) | ☐ |
| 3 | Create frontend-design skill | `~/.claude/skills/frontend-design/SKILL.md` | ☐ |
| 4 | Create landing-page skill | `~/.claude/skills/landing-page/SKILL.md` | ☐ |

### Phase 7 — Install Official Plugins

| # | Plugin | Command | Done? |
|---|---|---|---|
| 1 | frontend-design | `claude plugin install frontend-design@claude-plugins-official` | ☐ |
| 2 | code-review | `claude plugin install code-review@claude-plugins-official` | ☐ |
| 3 | feature-dev | `claude plugin install feature-dev@claude-plugins-official` | ☐ |
| 4 | commit-commands | `claude plugin install commit-commands@claude-plugins-official` | ☐ |
| 5 | security-guidance | `claude plugin install security-guidance@claude-plugins-official` | ☐ |

### Phase 8 — Install LSP

| # | Task | Command | Done? |
|---|---|---|---|
| 1 | Install typescript-lsp plugin | `claude plugin install typescript-lsp@claude-plugins-official` | ☐ |
| 2 | Install pyright-lsp plugin | `claude plugin install pyright-lsp@claude-plugins-official` | ☐ |
| 3 | Install php-lsp plugin | `claude plugin install php-lsp@claude-plugins-official` | ☐ |
| 4 | Add boostvolt marketplace | `claude plugin marketplace add boostvolt/claude-code-lsps` | ☐ |
| 5 | Install omnisharp plugin | `claude plugin install omnisharp@claude-code-lsps` | ☐ |
| 6 | Install TS language server | `npm install -g typescript-language-server typescript` | ☐ |
| 7 | Install Python language server | `npm install -g pyright` | ☐ |
| 8 | Install PHP language server | `npm install -g intelephense` | ☐ |
| 9 | Install C# language server | `dotnet tool install -g csharp-ls` | ☐ |

### Phase 9 — Install Community Skills

| # | Skill | npx Command | Done? |
|---|---|---|---|
| 1 | senior-architect | `npx ai-agent-skills install alirezarezvani/claude-skills/engineering-team/senior-architect --agent claude` | ☐ |
| 2 | senior-backend | `npx ai-agent-skills install alirezarezvani/claude-skills/engineering-team/senior-backend --agent claude` | ☐ |
| 3 | senior-frontend | `npx ai-agent-skills install alirezarezvani/claude-skills/engineering-team/senior-frontend --agent claude` | ☐ |
| 4 | code-reviewer | `npx ai-agent-skills install alirezarezvani/claude-skills/engineering-team/code-reviewer --agent claude` | ☐ |
| 5 | senior-devops | `npx ai-agent-skills install alirezarezvani/claude-skills/engineering-team/senior-devops --agent claude` | ☐ |
| 6 | senior-security | `npx ai-agent-skills install alirezarezvani/claude-skills/engineering-team/senior-security --agent claude` | ☐ |
| 7 | senior-qa | `npx ai-agent-skills install alirezarezvani/claude-skills/engineering-team/senior-qa --agent claude` | ☐ |
| 8 | senior-dba | `npx ai-agent-skills install alirezarezvani/claude-skills/engineering-team/senior-dba --agent claude` | ☐ |
| 9 | cto-advisor | `npx ai-agent-skills install alirezarezvani/claude-skills/c-level-advisor/cto-advisor --agent claude` | ☐ |
| 10 | senior-pm | `npx ai-agent-skills install alirezarezvani/claude-skills/project-management/senior-pm --agent claude` | ☐ |

### Phase 10 — Verify Everything

| # | Verification Check | Command | Done? |
|---|---|---|---|
| 1 | Claude Code installed | `claude --version` | ☐ |
| 2 | Authenticated | `claude` (should open without login) | ☐ |
| 3 | LSP enabled | `echo $ENABLE_LSP_TOOL` (should print `1`) | ☐ |
| 4 | All plugins installed | `claude plugin list` | ☐ |
| 5 | Skills present | `ls ~/.claude/skills/` | ☐ |
| 6 | CLAUDE.md exists | `cat ~/.claude/CLAUDE.md` | ☐ |
| 7 | settings.json valid | `cat ~/.claude/settings.json` | ☐ |
| 8 | Test in project | `cd ~/project && claude` | ☐ |

---

## 17. Troubleshooting

### Common Issues & Fixes

| Problem | Cause | Fix |
|---|---|---|
| `claude: command not found` | npm global bin not in PATH | Add `C:\Users\<you>\AppData\Roaming\npm` to Windows System PATH, restart Git Bash |
| Plugin not found in marketplace | Wrong plugin name | Check exact name at claude.com/plugins |
| Invalid marketplace source format | Using name instead of owner/repo | Use: `owner/repo` format for community |
| Skill not available in session | Installed mid-session | Run `/reload-plugins` or restart with `claude` |
| LSP not working | Binary not in PATH or env var missing | Check: `echo $ENABLE_LSP_TOOL` and `which <binary>` |
| `Executable not found in $PATH` | Language server binary missing | Run binary installation commands, restart Git Bash |
| Authentication expired | Token expired | Run: `claude` — follow re-login prompts |
| `settings.json` broken | Invalid JSON syntax | Validate with: `node -e "JSON.parse(...)"` |
| Plugin shows "disabled" | Plugin installed but disabled | Run: `claude plugin enable <name>` |
| `cannot find module` error | Node.js version too old | Update Node.js to v18+ LTS |
| PhpStorm terminal can't find `claude` | PATH not applied to terminal | Set PATH via Windows System Variables (not `.bashrc`) and restart PhpStorm |

### Quick Diagnostic Commands

```bash
# Check Claude Code version
claude --version

# Check Node.js version
node --version

# Verify LSP environment variable
echo $ENABLE_LSP_TOOL

# Check all installed plugins
claude plugin list

# Check skills are present
ls ~/.claude/skills/

# Check CLAUDE.md is loaded
cat ~/.claude/CLAUDE.md | head -20

# Check settings.json is valid
cat ~/.claude/settings.json

# Check marketplaces registered
claude plugin marketplace list

# Check language server binaries exist
which typescript-language-server
which pyright-langserver
which intelephense
which csharp-ls

# Check Windows PATH includes npm (run in Git Bash)
echo $PATH | tr ':' '\n' | grep -i appdata
```

### Resources

| Resource | URL |
|---|---|
| Official Docs | code.claude.com/docs |
| Official Plugin Marketplace | claude.com/plugins |
| Community Skills Repo | github.com/alirezarezvani/claude-skills |
| LSP Community Plugins | github.com/boostvolt/claude-code-lsps |
| Claude Code Issues | github.com/anthropics/claude-code/issues |
| Anthropic Support | support.claude.com |

---

*Claude Code Setup Guide · Shakil Uddin · v1.1 · April 2026*
