# Prazium CLI

## Overview

The Prazium CLI provides command-line access to all Prazium functionality. It's designed for developers who prefer terminal workflows and for automation/CI integration.

## Installation

### npm/pnpm

```bash
# Global installation
pnpm add -g @prazium/cli

# Or with npm
npm install -g @prazium/cli
```

### Homebrew (macOS/Linux)

```bash
brew tap prazium/tap
brew install prazium
```

### Binary Download

```bash
# Linux
curl -fsSL https://get.prazium.com/cli | sh

# macOS
curl -fsSL https://get.prazium.com/cli | sh

# Windows (PowerShell)
iwr https://get.prazium.com/cli.ps1 -useb | iex
```

### Verify Installation

```bash
prazium --version
# prazium 1.0.0
```

---

## Configuration

### Authentication

```bash
# Login to hosted Prazium
prazium login

# Login with API key
prazium login --api-key pzm_xxxxxxxxxxxx

# Login to self-hosted instance
prazium login --url https://prazium.example.com

# Check authentication status
prazium whoami

# Logout
prazium logout
```

### Configuration File

Location: `~/.prazium/config.json`

```json
{
  "apiUrl": "https://api.prazium.com",
  "apiKey": "pzm_xxxxxxxxxxxx",
  "defaultFormat": "zip",
  "outputDir": "./exports"
}
```

### Environment Variables

| Variable | Description |
|----------|-------------|
| `PRAZIUM_API_URL` | API endpoint URL |
| `PRAZIUM_API_KEY` | API key for authentication |
| `PRAZIUM_OUTPUT_DIR` | Default export directory |
| `PRAZIUM_LOG_LEVEL` | Log verbosity (debug, info, warn, error) |

---

## Commands

### prazium new

Create a new project interactively.

```bash
prazium new [name]
```

**Options**:
| Option | Description |
|--------|-------------|
| `--name, -n` | Project name |
| `--description, -d` | Project description |
| `--file, -f` | Read description from file |
| `--no-interactive` | Skip interactive prompts |

**Examples**:

```bash
# Interactive mode
prazium new

# With name
prazium new "My SaaS App"

# With description
prazium new "My App" -d "A project management tool with..."

# From file
prazium new "My App" -f ./description.md

# Non-interactive (requires all inputs)
prazium new "My App" -d "Description..." --no-interactive
```

**Output**:
```
✓ Project created: proj_abc123
  Name: My SaaS App
  Status: draft

Next steps:
  prazium spec proj_abc123    # Generate specification
```

---

### prazium spec

Generate or view project specification.

```bash
prazium spec <project-id> [action]
```

**Actions**:
| Action | Description |
|--------|-------------|
| `generate` | Generate spec from description (default) |
| `view` | View current spec |
| `approve` | Approve spec for building |
| `regenerate` | Regenerate with feedback |

**Options**:
| Option | Description |
|--------|-------------|
| `--answers, -a` | JSON file with question answers |
| `--feedback` | Feedback for regeneration |
| `--output, -o` | Output spec to file |
| `--format` | Output format (json, markdown) |

**Examples**:

```bash
# Generate spec (interactive questions)
prazium spec proj_abc123

# Generate with pre-defined answers
prazium spec proj_abc123 -a answers.json

# View current spec
prazium spec proj_abc123 view

# View as markdown
prazium spec proj_abc123 view --format markdown > spec.md

# Approve spec
prazium spec proj_abc123 approve

# Regenerate with feedback
prazium spec proj_abc123 regenerate --feedback "Add more detail about auth"
```

**Interactive Flow**:
```
Generating specification for "My SaaS App"...

? What authentication method should users use?
  ○ Email/Password
  ● Magic Link
  ○ OAuth (Google, GitHub)
  ○ All of the above

? How many team members per workspace?
  ○ 1-5
  ● 5-20
  ○ 20-100
  ○ Unlimited

? Should the app support real-time collaboration? (Y/n) Y

Generating spec...

✓ Specification generated

Features:
  • User Authentication (Magic Link)
  • Team Workspaces (up to 20 members)
  • Real-time Collaboration

? Approve this specification? (Y/n)
```

---

### prazium build

Start a build for a project.

```bash
prazium build <project-id>
```

**Options**:
| Option | Description |
|--------|-------------|
| `--watch, -w` | Watch build progress |
| `--wait` | Wait for build to complete |
| `--timeout` | Build timeout in minutes |
| `--quiet, -q` | Minimal output |

**Examples**:

```bash
# Start build
prazium build proj_abc123

# Start and watch progress
prazium build proj_abc123 --watch

# Start and wait for completion
prazium build proj_abc123 --wait

# With timeout
prazium build proj_abc123 --wait --timeout 30
```

**Watch Output**:
```
Building proj_abc123 (Run #1)

Phase: Executing Tasks
Progress: ████████████░░░░░░░░ 60%

Tasks:
  ✓ project-setup      00:12
  ✓ database-schema    00:23
  ✓ authentication     00:45
  ◐ api-routes         01:12...
  ○ ui-components      --:--
  ○ tests              --:--

Activity:
  [12:34:56] Creating src/app/api/tasks/route.ts
  [12:34:58] Running pnpm install...
  [12:35:10] ✓ Dependencies installed
```

---

### prazium verify

Run verification on a build.

```bash
prazium verify <project-id> [run-id]
```

**Options**:
| Option | Description |
|--------|-------------|
| `--checks` | Specific checks to run (lint,test,build) |
| `--fix` | Attempt to auto-fix issues |

**Examples**:

```bash
# Verify latest build
prazium verify proj_abc123

# Verify specific run
prazium verify proj_abc123 run_xyz789

# Run only lint and test
prazium verify proj_abc123 --checks lint,test

# Verify with auto-fix
prazium verify proj_abc123 --fix
```

**Output**:
```
Verifying proj_abc123 (Run #1)

Checks:
  ✓ Lint          3 warnings, 0 errors
  ✓ Test          25/25 passed (85% coverage)
  ✓ Build         Succeeded
  ✓ Acceptance    All criteria met

Result: PASSED

Release tag: release/run-1
```

---

### prazium deploy

Deploy a build to preview environment.

```bash
prazium deploy <project-id> [run-id]
```

**Options**:
| Option | Description |
|--------|-------------|
| `--provider` | Deployment provider (railway) |
| `--wait` | Wait for deployment to complete |
| `--env` | Environment variables file |

**Examples**:

```bash
# Deploy latest build
prazium deploy proj_abc123

# Deploy specific run
prazium deploy proj_abc123 run_xyz789

# Deploy with env vars
prazium deploy proj_abc123 --env .env.production

# Deploy and wait
prazium deploy proj_abc123 --wait
```

**Output**:
```
Deploying proj_abc123 (Run #1) to Railway...

Steps:
  ✓ Building Docker image
  ✓ Pushing to registry
  ✓ Provisioning resources
  ✓ Starting application

✓ Deployment complete!

Preview URL: https://my-app-preview.up.railway.app
```

---

### prazium export

Export a build as a downloadable bundle.

```bash
prazium export <project-id> [run-id]
```

**Options**:
| Option | Description |
|--------|-------------|
| `--format, -f` | Export format (zip, tar.gz) |
| `--output, -o` | Output path |
| `--include-git` | Include .git directory |

**Examples**:

```bash
# Export latest build
prazium export proj_abc123

# Export specific run
prazium export proj_abc123 run_xyz789

# Export as tarball
prazium export proj_abc123 -f tar.gz

# Export to specific path
prazium export proj_abc123 -o ./my-app.zip

# Include git history
prazium export proj_abc123 --include-git
```

**Output**:
```
Exporting proj_abc123 (Run #1)...

✓ Export complete!

File: ./my-saas-app-run-1.zip
Size: 5.2 MB

Contents:
  • Full source code
  • Dockerfile
  • docker-compose.yml
  • README_DEPLOY.md
  • .env.example
```

---

### prazium list

List projects or runs.

```bash
prazium list [resource]
```

**Resources**:
| Resource | Description |
|----------|-------------|
| `projects` | List all projects (default) |
| `runs` | List runs for a project |
| `tasks` | List tasks for a run |
| `exports` | List exports for a project |

**Options**:
| Option | Description |
|--------|-------------|
| `--project, -p` | Project ID (for runs/tasks/exports) |
| `--run, -r` | Run ID (for tasks) |
| `--status` | Filter by status |
| `--limit` | Number of items |
| `--json` | Output as JSON |

**Examples**:

```bash
# List all projects
prazium list

# List projects as JSON
prazium list --json

# List runs for a project
prazium list runs -p proj_abc123

# List failed runs
prazium list runs -p proj_abc123 --status failed

# List tasks for a run
prazium list tasks -p proj_abc123 -r run_xyz789
```

**Output**:
```
Projects:

ID            NAME           STATUS     LAST BUILD
proj_abc123   My SaaS App    completed  2 hours ago
proj_def456   Landing Page   building   in progress
proj_ghi789   API Service    draft      --
```

---

### prazium status

Get status of a project or run.

```bash
prazium status <project-id> [run-id]
```

**Options**:
| Option | Description |
|--------|-------------|
| `--watch, -w` | Watch for changes |
| `--json` | Output as JSON |

**Examples**:

```bash
# Project status
prazium status proj_abc123

# Run status
prazium status proj_abc123 run_xyz789

# Watch run status
prazium status proj_abc123 run_xyz789 --watch
```

**Output**:
```
Project: My SaaS App (proj_abc123)
Status: completed
Created: 2024-01-15 10:30:00

Latest Run: #3
  Status: succeeded
  Started: 2024-01-15 12:00:00
  Duration: 8m 45s
  Tasks: 6/6 completed

Preview: https://my-app-preview.up.railway.app
```

---

### prazium logs

View logs for a run or task.

```bash
prazium logs <project-id> <run-id> [task-id]
```

**Options**:
| Option | Description |
|--------|-------------|
| `--follow, -f` | Follow log output |
| `--tail` | Number of lines to show |
| `--since` | Show logs since timestamp |
| `--type` | Filter by event type |

**Examples**:

```bash
# View run logs
prazium logs proj_abc123 run_xyz789

# Follow logs
prazium logs proj_abc123 run_xyz789 -f

# View task logs
prazium logs proj_abc123 run_xyz789 task_001

# Last 50 lines
prazium logs proj_abc123 run_xyz789 --tail 50

# Filter by type
prazium logs proj_abc123 run_xyz789 --type FILE_CREATED
```

**Output**:
```
[2024-01-15 12:00:01] RUN_STARTED
[2024-01-15 12:00:02] TASK_STARTED task-001 (project-setup)
[2024-01-15 12:00:05] FILE_CREATED package.json
[2024-01-15 12:00:06] FILE_CREATED tsconfig.json
[2024-01-15 12:00:10] COMMAND_STARTED pnpm install
[2024-01-15 12:00:25] COMMAND_COMPLETED exit=0
[2024-01-15 12:00:26] TASK_SUCCEEDED task-001 (14s)
```

---

### prazium delete

Delete a project or export.

```bash
prazium delete <resource> <id>
```

**Resources**:
| Resource | Description |
|----------|-------------|
| `project` | Delete (archive) a project |
| `export` | Delete an export |

**Options**:
| Option | Description |
|--------|-------------|
| `--force, -f` | Skip confirmation |

**Examples**:

```bash
# Delete project (with confirmation)
prazium delete project proj_abc123

# Force delete
prazium delete project proj_abc123 --force

# Delete export
prazium delete export exp_123
```

---

### prazium config

Manage CLI configuration.

```bash
prazium config <action> [key] [value]
```

**Actions**:
| Action | Description |
|--------|-------------|
| `get` | Get a config value |
| `set` | Set a config value |
| `list` | List all config |
| `reset` | Reset to defaults |

**Examples**:

```bash
# List all config
prazium config list

# Get a value
prazium config get apiUrl

# Set a value
prazium config set outputDir ./exports

# Reset config
prazium config reset
```

---

## Global Options

These options work with all commands:

| Option | Description |
|--------|-------------|
| `--help, -h` | Show help |
| `--version, -v` | Show version |
| `--verbose` | Verbose output |
| `--quiet, -q` | Minimal output |
| `--json` | JSON output |
| `--no-color` | Disable colors |
| `--api-url` | Override API URL |
| `--api-key` | Override API key |

---

## Exit Codes

| Code | Description |
|------|-------------|
| 0 | Success |
| 1 | General error |
| 2 | Invalid arguments |
| 3 | Authentication error |
| 4 | Not found |
| 5 | Build failed |
| 6 | Timeout |
| 130 | Interrupted (Ctrl+C) |

---

## Scripting Examples

### CI/CD Pipeline

```bash
#!/bin/bash
set -e

# Create project from description file
PROJECT_ID=$(prazium new "My App" -f ./description.md --no-interactive --json | jq -r '.id')

# Generate and approve spec
prazium spec $PROJECT_ID -a ./answers.json
prazium spec $PROJECT_ID approve

# Build and wait
prazium build $PROJECT_ID --wait --timeout 30

# Export
prazium export $PROJECT_ID -o ./dist/app.zip

echo "Build complete: ./dist/app.zip"
```

### Batch Processing

```bash
#!/bin/bash

# Build all projects with 'pending' status
prazium list --status approved --json | jq -r '.[].id' | while read id; do
  echo "Building $id..."
  prazium build $id --wait || echo "Failed: $id"
done
```

### Monitoring Script

```bash
#!/bin/bash

# Watch all active builds
while true; do
  clear
  echo "Active Builds:"
  echo "=============="
  prazium list runs --status running --json | jq -r '.[] | "\(.projectId): \(.status) (\(.progress)%)"'
  sleep 5
done
```

---

## Shell Completion

### Bash

```bash
# Add to ~/.bashrc
eval "$(prazium completion bash)"
```

### Zsh

```bash
# Add to ~/.zshrc
eval "$(prazium completion zsh)"
```

### Fish

```bash
# Add to ~/.config/fish/config.fish
prazium completion fish | source
```

---

## Troubleshooting

### Common Issues

| Issue | Solution |
|-------|----------|
| "Authentication failed" | Run `prazium login` |
| "Project not found" | Check project ID with `prazium list` |
| "Build timeout" | Increase with `--timeout` |
| "Connection refused" | Check `PRAZIUM_API_URL` |

### Debug Mode

```bash
# Enable debug logging
PRAZIUM_LOG_LEVEL=debug prazium build proj_abc123

# Or with flag
prazium build proj_abc123 --verbose
```

### Reset State

```bash
# Clear all local config
rm -rf ~/.prazium

# Re-authenticate
prazium login
```

---

## Related Documentation

- [API Specification](./API.md)
- [Deployment Guide](./DEPLOYMENT.md)
- [Architecture Overview](./ARCHITECTURE.md)
