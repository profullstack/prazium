# Prazium Architecture

## Overview

Prazium is a distributed system composed of four primary components that work together to transform product descriptions into production-ready applications.

## System Components

```mermaid
graph TB
    subgraph Client Layer
        UI[Web UI]
        CLI[CLI Tool]
    end

    subgraph API Layer
        API[Orchestrator API]
        WS[WebSocket Server]
    end

    subgraph Processing Layer
        ORCH[Orchestrator Service]
        QUEUE[Job Queue]
        WORKERS[Worker Pool]
    end

    subgraph Storage Layer
        PG[(PostgreSQL)]
        FS[Filesystem - Git Repos]
        S3[S3-Compatible Storage]
    end

    subgraph Execution Layer
        SANDBOX1[Docker Sandbox 1]
        SANDBOX2[Docker Sandbox 2]
        SANDBOXN[Docker Sandbox N]
    end

    UI --> API
    CLI --> API
    UI --> WS
    
    API --> ORCH
    API --> PG
    
    ORCH --> QUEUE
    ORCH --> PG
    ORCH --> FS
    
    QUEUE --> WORKERS
    
    WORKERS --> SANDBOX1
    WORKERS --> SANDBOX2
    WORKERS --> SANDBOXN
    WORKERS --> FS
    WORKERS --> PG
    
    WS --> PG
    
    ORCH --> S3
```

## Component Details

### 1. Web UI

**Purpose**: User-facing interface for app creation and monitoring

**Responsibilities**:
- Product description intake form
- Clarifying question wizard
- Spec review and approval interface
- Live build watcher dashboard
- Export and deploy controls
- Project management

**Technology**: Next.js App Router, React 19, Tailwind CSS

**Key Pages**:
| Route | Purpose |
|-------|---------|
| `/` | Landing / Dashboard |
| `/new` | New product wizard |
| `/projects` | Project list |
| `/projects/[id]` | Project detail |
| `/projects/[id]/runs/[runId]` | Build run watcher |
| `/projects/[id]/export` | Export controls |

---

### 2. Orchestrator API

**Purpose**: Central control plane for all operations

**Responsibilities**:
- SDLC state machine management
- Internal Git repository lifecycle
- Task decomposition and scheduling
- Branch merging and integration
- Verification gate execution
- Event emission for watchers

**Technology**: Node.js, Express/Fastify, TypeScript

**Key Modules**:

```mermaid
graph LR
    subgraph Orchestrator API
        REST[REST Endpoints]
        SM[State Machine]
        GIT[Git Manager]
        TASK[Task Scheduler]
        MERGE[Integrator]
        VERIFY[Verifier]
        EVENT[Event Emitter]
    end

    REST --> SM
    SM --> GIT
    SM --> TASK
    SM --> MERGE
    SM --> VERIFY
    SM --> EVENT
```

---

### 3. Worker / Runner

**Purpose**: Execute agent tasks in isolated environments

**Responsibilities**:
- Pull tasks from job queue
- Create and manage Git worktrees
- Spawn Docker sandboxes
- Execute AI agents within sandboxes
- Capture diffs, logs, and events
- Commit changes to task branches

**Technology**: Node.js, Docker SDK, TypeScript

**Worker Lifecycle**:

```mermaid
sequenceDiagram
    participant Q as Job Queue
    participant W as Worker
    participant G as Git Repo
    participant D as Docker Sandbox
    participant A as AI Agent
    participant DB as PostgreSQL

    W->>Q: Poll for task
    Q-->>W: Task payload
    W->>G: Create worktree
    W->>D: Spawn sandbox
    W->>D: Mount worktree
    W->>A: Initialize agent
    
    loop Agent Execution
        A->>D: Read/Write files
        A->>D: Run commands
        A->>W: Emit events
        W->>DB: Store events
    end
    
    A->>W: Task complete
    W->>G: Commit changes
    W->>D: Destroy sandbox
    W->>Q: Acknowledge task
```

---

### 4. Storage Layer

#### PostgreSQL

**Purpose**: Persistent structured data

**Stores**:
- Projects and metadata
- Build runs and state
- Tasks and status
- Events and logs
- User data (hosted mode)

#### Filesystem (Git Repos)

**Purpose**: Source code and version control

**Structure**:
```
data/
├── repos/
│   ├── <productId-1>.git    # Bare repository
│   ├── <productId-2>.git
│   └── ...
└── workspaces/
    ├── <productId-1>/
    │   ├── main/            # Main branch checkout
    │   └── wt/
    │       ├── feat-task-001/
    │       └── feat-task-002/
    └── <productId-2>/
        └── ...
```

#### S3-Compatible Storage (Optional)

**Purpose**: Artifact storage

**Stores**:
- Export bundles (zip/tarball)
- Build artifacts
- Docker images (if not using registry)

---

## Data Flow

### New Product Flow

```mermaid
sequenceDiagram
    participant U as User
    participant UI as Web UI
    participant API as Orchestrator API
    participant LLM as LLM Provider
    participant DB as PostgreSQL
    participant GIT as Git Repo

    U->>UI: Enter product description
    UI->>API: POST /projects
    API->>DB: Create project record
    API->>GIT: Initialize bare repo
    API->>LLM: Generate clarifying questions
    LLM-->>API: Questions
    API-->>UI: Questions
    UI-->>U: Display questions
    
    U->>UI: Answer questions
    UI->>API: POST /projects/:id/answers
    API->>LLM: Generate spec documents
    LLM-->>API: SPEC.md, ACCEPTANCE.md, etc.
    API->>GIT: Commit spec to spec branch
    API->>DB: Update project state
    API-->>UI: Spec for review
    UI-->>U: Display spec
    
    U->>UI: Approve spec
    UI->>API: POST /projects/:id/approve
    API->>DB: Update state to BUILDING
```

### Build Execution Flow

```mermaid
sequenceDiagram
    participant API as Orchestrator
    participant Q as Job Queue
    participant W1 as Worker 1
    participant W2 as Worker 2
    participant GIT as Git Repo
    participant DB as PostgreSQL

    API->>Q: Enqueue tasks
    
    par Parallel Execution
        W1->>Q: Claim task-001
        W1->>GIT: Create worktree feat-task-001
        W1->>W1: Execute in sandbox
        W1->>GIT: Commit to feat-task-001
        W1->>DB: Mark task complete
    and
        W2->>Q: Claim task-002
        W2->>GIT: Create worktree feat-task-002
        W2->>W2: Execute in sandbox
        W2->>GIT: Commit to feat-task-002
        W2->>DB: Mark task complete
    end
    
    API->>API: All tasks complete
    API->>GIT: Merge branches in order
    API->>API: Run verification
    API->>GIT: Tag release
    API->>DB: Update run status
```

---

## Deployment Architecture

### Self-Hosted (Single Node)

```mermaid
graph TB
    subgraph Docker Host
        subgraph App Containers
            UI[Web UI Container]
            API[API Container]
            WORKER[Worker Container]
        end
        
        subgraph Infrastructure
            PG[(PostgreSQL)]
            REDIS[Redis - Queue]
        end
        
        subgraph Volumes
            DATA[/data - Git Repos]
            PGDATA[/pgdata]
        end
        
        subgraph Build Sandboxes
            SB1[Sandbox 1]
            SB2[Sandbox 2]
        end
    end

    UI --> API
    API --> PG
    API --> REDIS
    API --> DATA
    WORKER --> REDIS
    WORKER --> DATA
    WORKER --> PG
    WORKER -.-> SB1
    WORKER -.-> SB2
    PG --> PGDATA
```

### Hosted (Multi-Node)

```mermaid
graph TB
    subgraph Load Balancer
        LB[nginx / Traefik]
    end

    subgraph API Nodes
        API1[API Node 1]
        API2[API Node 2]
    end

    subgraph Worker Nodes
        W1[Worker Node 1]
        W2[Worker Node 2]
        W3[Worker Node 3]
    end

    subgraph Shared Storage
        PG[(PostgreSQL - Managed)]
        REDIS[Redis Cluster]
        NFS[NFS / EFS - Git Repos]
        S3[S3 - Artifacts]
    end

    LB --> API1
    LB --> API2
    
    API1 --> PG
    API1 --> REDIS
    API1 --> NFS
    
    API2 --> PG
    API2 --> REDIS
    API2 --> NFS
    
    W1 --> REDIS
    W1 --> NFS
    W1 --> PG
    
    W2 --> REDIS
    W2 --> NFS
    W2 --> PG
    
    W3 --> REDIS
    W3 --> NFS
    W3 --> PG
    
    API1 --> S3
    API2 --> S3
```

---

## Technology Stack

| Component | Technology | Rationale |
|-----------|------------|-----------|
| Web UI | Next.js 15, React 19 | SSR, App Router, modern React |
| API | Node.js, Fastify | Performance, TypeScript support |
| Queue | BullMQ + Redis | Reliable job processing |
| Database | PostgreSQL 16 | JSONB, reliability, ecosystem |
| Git Operations | isomorphic-git + libgit2 | Pure JS + native performance |
| Containers | Docker Engine API | Industry standard, isolation |
| Real-time | WebSocket / SSE | Live updates |
| LLM | OpenAI / Anthropic API | Agent intelligence |

---

## Scalability Considerations

### Horizontal Scaling

| Component | Scaling Strategy |
|-----------|------------------|
| Web UI | Stateless, scale behind LB |
| API | Stateless, scale behind LB |
| Workers | Add nodes, share queue |
| PostgreSQL | Read replicas, connection pooling |
| Git Repos | NFS/EFS for shared access |

### Resource Limits

| Resource | Default Limit |
|----------|---------------|
| Sandbox CPU | 2 cores |
| Sandbox Memory | 4GB |
| Sandbox Disk | 10GB |
| Build Timeout | 30 minutes |
| Concurrent Tasks | 4 per worker |

---

## Network Architecture

```mermaid
graph TB
    subgraph Public Network
        USER[User Browser]
    end

    subgraph DMZ
        LB[Load Balancer]
    end

    subgraph Private Network
        API[API Servers]
        WORKERS[Worker Nodes]
    end

    subgraph Isolated Network
        SANDBOXES[Build Sandboxes]
    end

    subgraph Data Network
        DB[(Database)]
        STORAGE[Storage]
    end

    USER --> LB
    LB --> API
    API --> WORKERS
    API --> DB
    API --> STORAGE
    WORKERS --> SANDBOXES
    WORKERS --> DB
    WORKERS --> STORAGE
    
    SANDBOXES -. No external access .- USER
```

**Key Security Boundaries**:
1. Sandboxes have no host networking
2. Sandboxes cannot reach external services (except LLM API via proxy)
3. Database is not accessible from sandboxes
4. Workers communicate with sandboxes via Docker API only

---

## Failure Modes and Recovery

| Failure | Detection | Recovery |
|---------|-----------|----------|
| Worker crash | Heartbeat timeout | Task re-queued |
| Sandbox timeout | Build timeout | Task failed, cleanup |
| Database unavailable | Connection error | Retry with backoff |
| Git corruption | Integrity check | Restore from backup |
| LLM API down | API error | Retry with backoff |

---

## Monitoring Points

| Metric | Source | Alert Threshold |
|--------|--------|-----------------|
| Build queue depth | Redis | > 100 pending |
| Build success rate | PostgreSQL | < 90% |
| Worker utilization | Worker metrics | > 80% sustained |
| Sandbox spawn time | Worker metrics | > 30s |
| API latency p99 | API metrics | > 500ms |

---

## Related Documentation

- [Database Schema](./DATABASE.md)
- [API Specification](./API.md)
- [Event System](./EVENTS.md)
- [Git Model](./GIT-MODEL.md)
- [SDLC Workflow](./SDLC.md)
- [Security Model](./SECURITY.md)
