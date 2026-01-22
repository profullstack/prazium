# Prazium API Specification

## Overview

The Orchestrator API is the central control plane for Prazium. It provides RESTful endpoints for project management, build orchestration, and system administration.

**Base URL**: `https://api.prazium.com/v1` (hosted) or `http://localhost:3001/v1` (self-hosted)

## Authentication

### API Key Authentication

```http
Authorization: Bearer pzm_xxxxxxxxxxxxxxxxxxxx
```

### Session Authentication (Web UI)

```http
Cookie: prazium_session=<session_token>
```

---

## Common Response Formats

### Success Response

```json
{
  "success": true,
  "data": { ... }
}
```

### Error Response

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Project name is required",
    "details": { ... }
  }
}
```

### Pagination

```json
{
  "success": true,
  "data": [ ... ],
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "total": 150,
    "totalPages": 8
  }
}
```

---

## Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `UNAUTHORIZED` | 401 | Missing or invalid authentication |
| `FORBIDDEN` | 403 | Insufficient permissions |
| `NOT_FOUND` | 404 | Resource not found |
| `VALIDATION_ERROR` | 400 | Invalid request data |
| `CONFLICT` | 409 | Resource state conflict |
| `RATE_LIMITED` | 429 | Too many requests |
| `INTERNAL_ERROR` | 500 | Server error |

---

## Endpoints

### Projects

#### Create Project

```http
POST /projects
```

**Request Body**:
```json
{
  "name": "My SaaS App",
  "description": "A project management tool with team collaboration features, task boards, and time tracking."
}
```

**Response** (201 Created):
```json
{
  "success": true,
  "data": {
    "id": "proj_abc123",
    "name": "My SaaS App",
    "description": "A project management tool...",
    "status": "draft",
    "repoPath": "/data/repos/proj_abc123.git",
    "createdAt": "2024-01-15T10:30:00Z",
    "updatedAt": "2024-01-15T10:30:00Z"
  }
}
```

---

#### List Projects

```http
GET /projects
```

**Query Parameters**:
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | int | 1 | Page number |
| `pageSize` | int | 20 | Items per page (max 100) |
| `status` | string | - | Filter by status |
| `search` | string | - | Search name/description |

**Response** (200 OK):
```json
{
  "success": true,
  "data": [
    {
      "id": "proj_abc123",
      "name": "My SaaS App",
      "status": "building",
      "latestRun": {
        "id": "run_xyz789",
        "runNumber": 3,
        "status": "executing"
      },
      "createdAt": "2024-01-15T10:30:00Z",
      "updatedAt": "2024-01-15T12:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "total": 5,
    "totalPages": 1
  }
}
```

---

#### Get Project

```http
GET /projects/:projectId
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "id": "proj_abc123",
    "name": "My SaaS App",
    "description": "A project management tool...",
    "status": "approved",
    "spec": {
      "version": 1,
      "documents": {
        "SPEC.md": "# My SaaS App Specification...",
        "ACCEPTANCE.md": "# Acceptance Criteria...",
        "ROUTES.md": "# API Routes...",
        "DATA.md": "# Data Model..."
      },
      "approvedAt": "2024-01-15T11:00:00Z"
    },
    "repoPath": "/data/repos/proj_abc123.git",
    "createdAt": "2024-01-15T10:30:00Z",
    "updatedAt": "2024-01-15T11:00:00Z"
  }
}
```

---

#### Update Project

```http
PATCH /projects/:projectId
```

**Request Body**:
```json
{
  "name": "Updated Name",
  "description": "Updated description"
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": { ... }
}
```

---

#### Delete Project

```http
DELETE /projects/:projectId
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "id": "proj_abc123",
    "status": "archived"
  }
}
```

---

### Spec Generation

#### Get Clarifying Questions

```http
POST /projects/:projectId/clarify
```

Generates clarifying questions based on the project description.

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "questions": [
      {
        "id": "q1",
        "question": "What authentication method should users use?",
        "type": "single_choice",
        "options": ["Email/Password", "OAuth (Google, GitHub)", "Magic Link", "All of the above"]
      },
      {
        "id": "q2",
        "question": "How many team members should a workspace support?",
        "type": "single_choice",
        "options": ["1-5", "5-20", "20-100", "Unlimited"]
      },
      {
        "id": "q3",
        "question": "Should the app support real-time collaboration?",
        "type": "boolean"
      }
    ]
  }
}
```

---

#### Submit Answers

```http
POST /projects/:projectId/answers
```

**Request Body**:
```json
{
  "answers": [
    { "questionId": "q1", "answer": "Email/Password" },
    { "questionId": "q2", "answer": "5-20" },
    { "questionId": "q3", "answer": true }
  ]
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "status": "spec_review",
    "spec": {
      "version": 1,
      "documents": {
        "SPEC.md": "...",
        "ACCEPTANCE.md": "...",
        "ROUTES.md": "...",
        "DATA.md": "..."
      }
    }
  }
}
```

---

#### Approve Spec

```http
POST /projects/:projectId/approve
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "status": "approved",
    "spec": {
      "approvedAt": "2024-01-15T11:00:00Z"
    }
  }
}
```

---

#### Regenerate Spec

```http
POST /projects/:projectId/regenerate
```

**Request Body** (optional):
```json
{
  "feedback": "Add more detail about the notification system"
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "status": "spec_review",
    "spec": { ... }
  }
}
```

---

### Builds (Runs)

#### Start Build

```http
POST /projects/:projectId/runs
```

**Request Body** (optional):
```json
{
  "trigger": "manual"
}
```

**Response** (201 Created):
```json
{
  "success": true,
  "data": {
    "id": "run_xyz789",
    "projectId": "proj_abc123",
    "runNumber": 1,
    "status": "decomposing",
    "trigger": "manual",
    "createdAt": "2024-01-15T11:30:00Z"
  }
}
```

---

#### List Runs

```http
GET /projects/:projectId/runs
```

**Query Parameters**:
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | int | 1 | Page number |
| `pageSize` | int | 20 | Items per page |
| `status` | string | - | Filter by status |

**Response** (200 OK):
```json
{
  "success": true,
  "data": [
    {
      "id": "run_xyz789",
      "runNumber": 1,
      "status": "succeeded",
      "trigger": "manual",
      "tasksSummary": {
        "total": 6,
        "succeeded": 6,
        "failed": 0
      },
      "startedAt": "2024-01-15T11:30:00Z",
      "completedAt": "2024-01-15T11:45:00Z"
    }
  ],
  "pagination": { ... }
}
```

---

#### Get Run

```http
GET /projects/:projectId/runs/:runId
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "id": "run_xyz789",
    "projectId": "proj_abc123",
    "runNumber": 1,
    "status": "executing",
    "trigger": "manual",
    "specSnapshot": { ... },
    "tasksJson": {
      "tasks": [ ... ],
      "mergeOrder": [ ... ]
    },
    "tasks": [
      {
        "id": "task_001",
        "slug": "ui-shell",
        "status": "succeeded",
        "sequence": 1
      },
      {
        "id": "task_002",
        "slug": "auth",
        "status": "running",
        "sequence": 2
      }
    ],
    "verificationResult": null,
    "startedAt": "2024-01-15T11:30:00Z",
    "completedAt": null
  }
}
```

---

#### Cancel Run

```http
POST /projects/:projectId/runs/:runId/cancel
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "id": "run_xyz789",
    "status": "cancelled"
  }
}
```

---

#### Retry Run

```http
POST /projects/:projectId/runs/:runId/retry
```

Creates a new run with the same spec.

**Response** (201 Created):
```json
{
  "success": true,
  "data": {
    "id": "run_abc456",
    "runNumber": 2,
    "status": "decomposing",
    "trigger": "retry"
  }
}
```

---

### Tasks

#### List Tasks

```http
GET /projects/:projectId/runs/:runId/tasks
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": [
    {
      "id": "task_001",
      "slug": "ui-shell",
      "status": "succeeded",
      "sequence": 1,
      "definition": {
        "description": "Create Next.js app shell",
        "pathScope": ["src/app/**"]
      },
      "branchName": "feat/task-001-ui-shell",
      "result": {
        "filesCreated": ["src/app/layout.tsx"],
        "commitSha": "abc123"
      },
      "startedAt": "2024-01-15T11:31:00Z",
      "completedAt": "2024-01-15T11:35:00Z"
    }
  ]
}
```

---

#### Get Task

```http
GET /projects/:projectId/runs/:runId/tasks/:taskId
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "id": "task_001",
    "slug": "ui-shell",
    "status": "succeeded",
    "sequence": 1,
    "definition": {
      "id": "task-001",
      "slug": "ui-shell",
      "description": "Create Next.js app shell with layout",
      "instructions": "Create the base Next.js application structure...",
      "dependencies": [],
      "pathScope": ["src/app/**", "src/components/layout/**"],
      "expectedOutputs": ["src/app/layout.tsx", "src/app/page.tsx"]
    },
    "branchName": "feat/task-001-ui-shell",
    "worktreePath": "/data/workspaces/proj_abc123/wt/feat-task-001",
    "result": {
      "filesCreated": ["src/app/layout.tsx", "src/app/page.tsx"],
      "filesModified": [],
      "commitSha": "abc123def456"
    },
    "startedAt": "2024-01-15T11:31:00Z",
    "completedAt": "2024-01-15T11:35:00Z"
  }
}
```

---

### Events

#### List Events

```http
GET /projects/:projectId/runs/:runId/events
```

**Query Parameters**:
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `after` | timestamp | - | Events after this time |
| `type` | string | - | Filter by event type |
| `taskId` | uuid | - | Filter by task |
| `limit` | int | 100 | Max events to return |

**Response** (200 OK):
```json
{
  "success": true,
  "data": [
    {
      "id": "evt_001",
      "runId": "run_xyz789",
      "taskId": "task_001",
      "eventType": "FILE_CREATED",
      "payload": {
        "path": "src/app/layout.tsx",
        "size": 1234
      },
      "createdAt": "2024-01-15T11:32:00Z"
    }
  ]
}
```

---

#### Stream Events (SSE)

```http
GET /projects/:projectId/runs/:runId/events/stream
```

**Headers**:
```http
Accept: text/event-stream
```

**Response** (200 OK, text/event-stream):
```
event: TASK_STARTED
data: {"taskId":"task_001","slug":"ui-shell"}

event: FILE_CREATED
data: {"taskId":"task_001","path":"src/app/layout.tsx"}

event: TASK_COMPLETED
data: {"taskId":"task_001","status":"succeeded"}
```

---

### Export

#### Create Export

```http
POST /projects/:projectId/export
```

**Request Body**:
```json
{
  "runId": "run_xyz789",
  "format": "zip"
}
```

**Response** (202 Accepted):
```json
{
  "success": true,
  "data": {
    "id": "exp_123",
    "status": "processing",
    "format": "zip"
  }
}
```

---

#### Get Export Status

```http
GET /projects/:projectId/exports/:exportId
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "id": "exp_123",
    "status": "completed",
    "format": "zip",
    "sizeBytes": 5242880,
    "downloadUrl": "/projects/proj_abc123/exports/exp_123/download",
    "expiresAt": "2024-01-22T11:30:00Z"
  }
}
```

---

#### Download Export

```http
GET /projects/:projectId/exports/:exportId/download
```

**Response** (200 OK):
```http
Content-Type: application/zip
Content-Disposition: attachment; filename="my-saas-app-run-1.zip"
```

---

### Deployment

#### Deploy Preview (Hosted Only)

```http
POST /projects/:projectId/deploy
```

**Request Body**:
```json
{
  "runId": "run_xyz789",
  "provider": "railway"
}
```

**Response** (202 Accepted):
```json
{
  "success": true,
  "data": {
    "deploymentId": "dep_456",
    "status": "deploying",
    "provider": "railway"
  }
}
```

---

#### Get Deployment Status

```http
GET /projects/:projectId/deployments/:deploymentId
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "id": "dep_456",
    "status": "live",
    "provider": "railway",
    "url": "https://my-saas-app-preview.up.railway.app",
    "createdAt": "2024-01-15T12:00:00Z"
  }
}
```

---

### Files (Read-Only)

#### List Files

```http
GET /projects/:projectId/files
```

**Query Parameters**:
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `ref` | string | main | Branch or tag |
| `path` | string | / | Directory path |

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "ref": "main",
    "path": "/",
    "entries": [
      { "name": "src", "type": "directory" },
      { "name": "package.json", "type": "file", "size": 1234 },
      { "name": "README.md", "type": "file", "size": 567 }
    ]
  }
}
```

---

#### Get File Content

```http
GET /projects/:projectId/files/:path
```

**Query Parameters**:
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `ref` | string | main | Branch or tag |

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "path": "src/app/page.tsx",
    "ref": "main",
    "content": "export default function Home() { ... }",
    "encoding": "utf-8",
    "size": 1234
  }
}
```

---

### System (Admin)

#### Health Check

```http
GET /health
```

**Response** (200 OK):
```json
{
  "status": "healthy",
  "version": "1.0.0",
  "components": {
    "database": "healthy",
    "redis": "healthy",
    "workers": "healthy"
  }
}
```

---

#### Get System Stats

```http
GET /admin/stats
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "projects": {
      "total": 150,
      "active": 45
    },
    "runs": {
      "total": 500,
      "succeeded": 450,
      "failed": 50,
      "inProgress": 5
    },
    "queue": {
      "pending": 12,
      "processing": 4
    },
    "workers": {
      "total": 3,
      "active": 3
    }
  }
}
```

---

## WebSocket API

### Connection

```javascript
const ws = new WebSocket('wss://api.prazium.com/v1/ws');

// Authenticate
ws.send(JSON.stringify({
  type: 'auth',
  token: 'pzm_xxxxxxxxxxxx'
}));
```

### Subscribe to Run Events

```javascript
ws.send(JSON.stringify({
  type: 'subscribe',
  channel: 'run:run_xyz789'
}));
```

### Event Messages

```json
{
  "type": "event",
  "channel": "run:run_xyz789",
  "data": {
    "eventType": "TASK_STARTED",
    "taskId": "task_001",
    "payload": { ... },
    "timestamp": "2024-01-15T11:31:00Z"
  }
}
```

### Unsubscribe

```javascript
ws.send(JSON.stringify({
  type: 'unsubscribe',
  channel: 'run:run_xyz789'
}));
```

---

## Rate Limits

| Tier | Requests/min | Concurrent Builds |
|------|--------------|-------------------|
| Free | 60 | 1 |
| Pro | 300 | 5 |
| Enterprise | 1000 | Unlimited |

Rate limit headers:
```http
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 45
X-RateLimit-Reset: 1705320000
```

---

## API Flow Diagram

```mermaid
sequenceDiagram
    participant C as Client
    participant API as Orchestrator API
    participant DB as PostgreSQL
    participant Q as Job Queue
    participant W as Worker

    C->>API: POST /projects
    API->>DB: Create project
    API-->>C: Project created

    C->>API: POST /projects/:id/clarify
    API-->>C: Clarifying questions

    C->>API: POST /projects/:id/answers
    API->>DB: Store answers
    API-->>C: Generated spec

    C->>API: POST /projects/:id/approve
    API->>DB: Update status
    API-->>C: Spec approved

    C->>API: POST /projects/:id/runs
    API->>DB: Create run
    API->>Q: Enqueue tasks
    API-->>C: Run started

    C->>API: GET /runs/:id/events/stream
    
    loop Build Progress
        W->>Q: Claim task
        W->>DB: Update task status
        DB-->>API: Event trigger
        API-->>C: SSE event
    end

    W->>DB: Run completed
    API-->>C: Build complete event

    C->>API: POST /projects/:id/export
    API-->>C: Export URL
```

---

## Related Documentation

- [Architecture Overview](./ARCHITECTURE.md)
- [Database Schema](./DATABASE.md)
- [Event System](./EVENTS.md)
- [Security Model](./SECURITY.md)
