# Prazium v1.0 Implementation Checklist

This document tracks all tasks required to ship Prazium v1.0. Tasks are organized by component and should be completed in roughly the order listed.

---

## 🏗️ Project Setup

- [ ] Initialize monorepo with pnpm workspaces
- [ ] Configure TypeScript with strict mode
- [ ] Set up ESLint and Prettier
- [ ] Configure Vitest for testing
- [ ] Create shared types package (`@prazium/shared`)
- [ ] Set up CI/CD with GitHub Actions
- [ ] Configure Docker build pipeline
- [ ] Create development environment scripts

---

## 🗄️ Database Layer

### Schema
- [ ] Create PostgreSQL database initialization script
- [ ] Implement `users` table and indexes
- [ ] Implement `projects` table with JSONB spec column
- [ ] Implement `runs` table with status tracking
- [ ] Implement `tasks` table with definition JSONB
- [ ] Implement `events` table for audit log
- [ ] Implement `exports` table for artifact tracking
- [ ] Implement `api_keys` table for authentication
- [ ] Create database migration system
- [ ] Write seed data for development

### Data Access
- [ ] Set up Drizzle ORM or Prisma
- [ ] Create repository pattern for data access
- [ ] Implement connection pooling
- [ ] Add query logging for development
- [ ] Write database integration tests

---

## 🔧 Orchestrator API

### Core Infrastructure
- [ ] Set up Fastify server with TypeScript
- [ ] Configure CORS and security headers
- [ ] Implement request validation with Zod
- [ ] Set up structured logging (pino)
- [ ] Create error handling middleware
- [ ] Implement health check endpoint

### Authentication
- [ ] Implement session-based auth for Web UI
- [ ] Implement API key authentication
- [ ] Create auth middleware
- [ ] Add rate limiting
- [ ] Implement logout and session invalidation

### Project Endpoints
- [ ] `POST /projects` - Create project
- [ ] `GET /projects` - List projects with pagination
- [ ] `GET /projects/:id` - Get project details
- [ ] `PATCH /projects/:id` - Update project
- [ ] `DELETE /projects/:id` - Archive project

### Spec Generation Endpoints
- [ ] `POST /projects/:id/clarify` - Generate questions
- [ ] `POST /projects/:id/answers` - Submit answers
- [ ] `POST /projects/:id/approve` - Approve spec
- [ ] `POST /projects/:id/regenerate` - Regenerate spec

### Build Endpoints
- [ ] `POST /projects/:id/runs` - Start build
- [ ] `GET /projects/:id/runs` - List runs
- [ ] `GET /projects/:id/runs/:runId` - Get run details
- [ ] `POST /projects/:id/runs/:runId/cancel` - Cancel run
- [ ] `POST /projects/:id/runs/:runId/retry` - Retry run

### Task Endpoints
- [ ] `GET /runs/:runId/tasks` - List tasks
- [ ] `GET /runs/:runId/tasks/:taskId` - Get task details

### Event Endpoints
- [ ] `GET /runs/:runId/events` - List events
- [ ] `GET /runs/:runId/events/stream` - SSE event stream
- [ ] Implement WebSocket server for real-time updates

### Export Endpoints
- [ ] `POST /projects/:id/export` - Create export
- [ ] `GET /projects/:id/exports/:exportId` - Get export status
- [ ] `GET /projects/:id/exports/:exportId/download` - Download export

### File Endpoints
- [ ] `GET /projects/:id/files` - List files
- [ ] `GET /projects/:id/files/*` - Get file content

---

## 📋 State Machine

- [ ] Define project state transitions
- [ ] Define run state transitions
- [ ] Define task state transitions
- [ ] Implement state machine library (XState or custom)
- [ ] Add state transition validation
- [ ] Emit events on state changes
- [ ] Write state machine tests

---

## 📨 Job Queue

- [ ] Set up Redis connection
- [ ] Configure BullMQ queues
- [ ] Create `prazium:tasks` queue for agent execution
- [ ] Create `prazium:integration` queue for merging
- [ ] Create `prazium:verification` queue for checks
- [ ] Implement job retry logic with exponential backoff
- [ ] Add job progress tracking
- [ ] Create queue monitoring dashboard
- [ ] Write queue integration tests

---

## 🌿 Git Operations

### Repository Management
- [ ] Implement bare repository initialization
- [ ] Create initial commit with template files
- [ ] Implement worktree creation
- [ ] Implement worktree cleanup
- [ ] Add repository integrity checks

### Branching
- [ ] Implement branch creation from main
- [ ] Implement spec branch workflow
- [ ] Implement feature branch workflow
- [ ] Implement fix branch workflow
- [ ] Implement release tagging

### Merging
- [ ] Implement deterministic merge ordering
- [ ] Detect merge conflicts
- [ ] Create conflict resolution workflow
- [ ] Implement merge commit creation

### Testing
- [ ] Write Git operation unit tests
- [ ] Write integration tests with real repos
- [ ] Test parallel worktree operations

---

## 🤖 Worker Service

### Core
- [ ] Set up worker process
- [ ] Implement job claiming from queue
- [ ] Add worker heartbeat
- [ ] Implement graceful shutdown
- [ ] Add worker metrics

### Sandbox Management
- [ ] Create Docker sandbox image
- [ ] Implement container spawning
- [ ] Configure resource limits (CPU, memory, disk)
- [ ] Implement network isolation
- [ ] Mount worktree volumes
- [ ] Implement container cleanup

### Agent Execution
- [ ] Create agent runtime package
- [ ] Implement LLM client (OpenAI/Anthropic)
- [ ] Build context assembly for prompts
- [ ] Implement file operations (read/write/delete)
- [ ] Implement command execution with allowlist
- [ ] Add path scope validation
- [ ] Implement event emission from agent
- [ ] Add timeout handling

### Verification
- [ ] Implement lint check runner
- [ ] Implement test runner
- [ ] Implement build runner
- [ ] Implement acceptance criteria checker
- [ ] Create fix task generation for failures

---

## 🎨 Web UI

### Setup
- [ ] Initialize Next.js 15 with App Router
- [ ] Configure Tailwind CSS
- [ ] Set up shadcn/ui components
- [ ] Configure TanStack Query
- [ ] Set up Zustand for state management
- [ ] Create API client with fetch

### Layout & Navigation
- [ ] Create root layout with navigation
- [ ] Implement responsive sidebar
- [ ] Add user menu dropdown
- [ ] Create loading states
- [ ] Implement error boundaries

### Authentication Pages
- [ ] Login page
- [ ] Register page (hosted mode)
- [ ] Password reset flow
- [ ] Session management

### Dashboard
- [ ] Project list with cards
- [ ] Project status indicators
- [ ] Search and filter
- [ ] Empty state for new users
- [ ] Pagination

### New Project Wizard
- [ ] Step 1: Description input
- [ ] Step 2: Clarifying questions
- [ ] Step 3: Spec review
- [ ] Step 4: Build initiation
- [ ] Progress indicator
- [ ] Back/forward navigation

### Build Watcher
- [ ] Overall progress bar
- [ ] Phase timeline
- [ ] Task list with status
- [ ] Live activity feed
- [ ] File change summary
- [ ] SSE connection management
- [ ] Auto-reconnect on disconnect

### Project Detail
- [ ] Project header with status
- [ ] Quick action buttons
- [ ] Build history list
- [ ] Preview link display

### Export Page
- [ ] Format selector
- [ ] Download button
- [ ] Deployment instructions
- [ ] Platform-specific guides

### Settings
- [ ] User profile settings
- [ ] API key management
- [ ] Theme toggle (light/dark)

### Components
- [ ] Button variants
- [ ] Card component
- [ ] Progress bar
- [ ] Timeline component
- [ ] Task list component
- [ ] Activity feed component
- [ ] File tree component
- [ ] Code viewer component
- [ ] Toast notifications
- [ ] Modal dialogs
- [ ] Form inputs with validation

---

## 💻 CLI Tool

### Setup
- [ ] Initialize CLI package with Commander.js
- [ ] Configure TypeScript build
- [ ] Set up npm publishing
- [ ] Create shell completion scripts

### Commands
- [ ] `prazium login` - Authentication
- [ ] `prazium logout` - Clear credentials
- [ ] `prazium whoami` - Show current user
- [ ] `prazium new` - Create project
- [ ] `prazium spec` - Generate/view/approve spec
- [ ] `prazium build` - Start build
- [ ] `prazium verify` - Run verification
- [ ] `prazium deploy` - Deploy preview
- [ ] `prazium export` - Download bundle
- [ ] `prazium list` - List resources
- [ ] `prazium status` - Get status
- [ ] `prazium logs` - View logs
- [ ] `prazium delete` - Delete resources
- [ ] `prazium config` - Manage configuration

### Features
- [ ] Interactive prompts
- [ ] JSON output mode
- [ ] Progress spinners
- [ ] Color output
- [ ] Verbose/quiet modes

---

## 📦 Export System

- [ ] Implement ZIP archive creation
- [ ] Implement tarball creation
- [ ] Generate `.env.example` file
- [ ] Generate `Dockerfile` for exported app
- [ ] Generate `docker-compose.yml`
- [ ] Generate `README_DEPLOY.md`
- [ ] Implement S3 upload (optional)
- [ ] Add export expiration cleanup job

---

## 🔒 Security

### Sandbox Hardening
- [ ] Create seccomp profile
- [ ] Configure AppArmor profile
- [ ] Implement capability dropping
- [ ] Test sandbox escape prevention

### Input Validation
- [ ] Validate all API inputs
- [ ] Sanitize user descriptions
- [ ] Validate file paths
- [ ] Validate commands

### Secret Management
- [ ] Implement secret injection at runtime
- [ ] Never log secrets
- [ ] Rotate session secrets
- [ ] Secure API key storage

### Audit Logging
- [ ] Log authentication events
- [ ] Log project operations
- [ ] Log build events
- [ ] Implement log retention policy

---

## 🐳 Docker & Deployment

### Images
- [ ] Create API Dockerfile
- [ ] Create Worker Dockerfile
- [ ] Create UI Dockerfile
- [ ] Create Agent sandbox Dockerfile
- [ ] Optimize image sizes with multi-stage builds

### Compose
- [ ] Create development docker-compose.yml
- [ ] Create production docker-compose.yml
- [ ] Add health checks
- [ ] Configure logging drivers
- [ ] Add volume management

### Documentation
- [ ] Write self-hosting guide
- [ ] Document environment variables
- [ ] Create backup/restore procedures
- [ ] Write upgrade procedures

---

## 🧪 Testing

### Unit Tests
- [ ] API route handlers
- [ ] State machine transitions
- [ ] Git operations
- [ ] Agent path validation
- [ ] Command validation

### Integration Tests
- [ ] Full build pipeline
- [ ] Event streaming
- [ ] Export generation
- [ ] Database operations

### E2E Tests
- [ ] New project flow
- [ ] Build watching
- [ ] Export download
- [ ] CLI commands

---

## 📚 Documentation

- [x] PRD
- [x] Architecture documentation
- [x] Database schema documentation
- [x] API documentation
- [x] Event system documentation
- [x] Git model documentation
- [x] SDLC documentation
- [x] Agent documentation
- [x] Security documentation
- [x] Deployment documentation
- [x] UI specification
- [x] CLI documentation
- [ ] API reference (OpenAPI/Swagger)
- [ ] Contributing guide
- [ ] Code of conduct
- [ ] Changelog

---

## 🚀 Launch Checklist

- [ ] All tests passing
- [ ] Security audit complete
- [ ] Documentation reviewed
- [ ] Docker images published
- [ ] npm packages published
- [ ] Domain configured
- [ ] SSL certificates installed
- [ ] Monitoring configured
- [ ] Backup system tested
- [ ] Launch announcement prepared

---

## Progress Tracking

| Component | Status | Progress |
|-----------|--------|----------|
| Project Setup | 🔲 Not Started | 0% |
| Database | 🔲 Not Started | 0% |
| API | 🔲 Not Started | 0% |
| State Machine | 🔲 Not Started | 0% |
| Job Queue | 🔲 Not Started | 0% |
| Git Operations | 🔲 Not Started | 0% |
| Worker | 🔲 Not Started | 0% |
| Web UI | 🔲 Not Started | 0% |
| CLI | 🔲 Not Started | 0% |
| Export | 🔲 Not Started | 0% |
| Security | 🔲 Not Started | 0% |
| Docker | 🔲 Not Started | 0% |
| Testing | 🔲 Not Started | 0% |
| Documentation | ✅ Complete | 100% |

---

**Last Updated**: 2024-01-22
