# PRD.md

## Product Name

**Prazium**
Domain: **prazium.com**

## Summary

Prazium is a self-hostable, AI-powered application factory that transforms a short product description into a production-ready web application in minutes.

Prazium operates as a **code factory**, not a low-code platform.

It uses:

* an **internal Git repository per product**
* **parallel AI agents** operating in isolated sandboxes
* a deterministic **spec → build → verify → release** SDLC
* a **watchable UI** where users can observe progress without touching code

Prazium is open source and deployable on any server.
A hosted version adds previews, collaboration, and managed builds.

---

## Goals

* Convert "Describe the app you want built" into a working production app
* Scale horizontally using parallel AI agents
* Avoid dependency on GitHub or third-party SCM services
* Provide a deterministic, auditable SDLC
* Allow non-technical users to never see code
* Allow technical users to export full source at any time
* Safely execute untrusted generated code

---

## Non-Goals

* No drag-and-drop UI builder
* No proprietary runtime abstraction layer
* No vendor lock-in
* No uncontrolled agent behavior
* No requirement for users to understand Git

---

## Target Users

1. **Founders / Product Managers**

   * Want a working app quickly
   * Do not want to read or manage code
2. **Agencies**

   * Want repeatable $2,500–$10,000 app builds
3. **Developers**

   * Want full source code and self-hosting
4. **Operators**

   * Want reproducible, auditable builds

---

## Core Architecture

### System Components

1. **Web UI**

   * App intake
   * Clarifying questions
   * Live build watchers
   * Export & deploy controls

2. **Orchestrator API**

   * Controls SDLC state machine
   * Creates internal repos
   * Spawns agent tasks
   * Merges branches
   * Runs verification gates

3. **Worker / Runner**

   * Pulls tasks from queue
   * Executes agents in Docker sandboxes
   * Emits diffs, logs, and events

4. **Storage**

   * Postgres: projects, runs, tasks, events
   * Filesystem: internal Git repos and worktrees
   * Optional S3-compatible object storage for artifacts

---

## Internal Git Model (Required)

### Repo Strategy

* **One internal Git repository per product**
* Repository is **bare**
* Git is an internal implementation detail
* Users are never required to understand or interact with Git

### Filesystem Layout

```
data/
  repos/
    <productId>.git
  workspaces/
    <productId>/
      main/
      wt/
        feat-task-001/
        feat-task-002/
```

### Branching Convention

* `main` – last verified green build
* `spec/<runId>` – approved spec and acceptance criteria
* `feat/<taskId>-<slug>` – agent task branches
* `fix/<runId>-<slug>` – verifier fixes
* `release/<runId>` – immutable release snapshot

### Tags

* `spec/<runId>`
* `release/<runId>`

---

## SDLC Workflow

### 1. Intake & Spec Gate

User provides:

* Product description
* Answers to clarifying questions (max 10)

System generates:

* `SPEC.md`
* `ACCEPTANCE.md`
* `ROUTES.md`
* `DATA.md`
* `TASKS.json`

User must explicitly approve the spec before build begins.

---

### 2. Task Decomposition

Tasks must be:

* Atomic
* Parallelizable
* Path-scoped to minimize conflicts

Example tasks:

* UI shell
* Supabase authentication
* Database schema + RLS
* API routes
* Golden-path feature
* Tests

---

### 3. Parallel Agent Execution

* One Git worktree per task
* One AI agent per worktree
* Agents may:

  * Read/write files
  * Run shell commands
* Agents may not:

  * Modify files outside task scope
  * Upgrade dependencies unless explicitly instructed

Each agent commits changes to its own branch.

---

### 4. Integration

Integrator merges branches in deterministic order:

1. Schema & shared contracts
2. Authentication
3. API routes
4. UI
5. Tests

---

### 5. Verifier Gate

A build is considered successful only if:

* `pnpm lint`
* `pnpm test`
* `pnpm build`
* Acceptance checks pass

Failures create a `fix/<runId>` branch.

---

### 6. Release

* Tag `release/<runId>`
* Export build artifacts
* Optional preview deployment

---

## Watchers (User-Facing Observability)

### Event Types

* `SPEC_CREATED`
* `TASK_STARTED`
* `FILE_PATCHED`
* `COMMAND_OUTPUT`
* `TEST_RESULT`
* `BUILD_RESULT`
* `DEPLOYMENT_STATUS`

### UI Requirements

* Live updates via WebSocket or SSE
* Timeline view of build activity
* Per-task logs
* High-level diff summaries
* No Git concepts exposed to users

---

## Generated Application Stack (Output)

### Default Stack

* Next.js (App Router)
* React 19
* pnpm
* Tailwind CSS
* Supabase (auth + database)
* Docker
* Railway-compatible deployment

### Required Output Files

* Full source code
* `.env.example`
* Dockerfile
* `docker-compose.yml`
* `README_DEPLOY.md`

---

## Deployment Modes

### Preview Deployment (Hosted Prazium)

* Build Docker image
* Deploy to Railway
* Provide preview URL

### Export Bundle (Mandatory)

* Zip or tarball of full application
* Deployable anywhere
* No Prazium dependency required

---

## Security Requirements

* All builds execute in Docker sandboxes
* No host networking by default
* Resource limits enforced
* Generated code treated as untrusted
* Secrets injected only at runtime

---

## Open Source vs Hosted

### Open Source (Self-Hosted)

* Single tenant
* Local admin authentication
* No billing
* Manual deploy/export

### Hosted (prazium.com)

* Multi-tenant
* Usage quotas
* Hosted previews
* Team collaboration
* Audit logs
* Optional human review step

---

## CLI Surface (Internal / Optional)

```
prazium new
prazium spec
prazium build
prazium verify
prazium deploy
prazium export
```

---

## Acceptance Criteria

* A user can describe an app and receive a working build without touching code
* Parallel agent execution does not corrupt builds
* Builds are reproducible from Git tags
* Exported apps deploy independently of Prazium
* System runs offline except for LLM access

---

## Success Metrics

* Time to first working app < 10 minutes
* ≥90% builds pass verification without manual fixes
* Zero cross-project contamination
* Deterministic rebuilds from release tags

---

## Out of Scope (v1)

* Visual editor
* Plugin marketplace
* Multi-cloud deployment
* Fine-grained RBAC
* Persistent runtime hosting

---

**End of PRD – Prazium**
