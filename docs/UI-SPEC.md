# Prazium UI Specification

## Overview

The Prazium Web UI provides a user-friendly interface for creating, monitoring, and managing AI-generated applications. The UI abstracts all technical complexity, including Git operations, from end users.

## Design Principles

1. **No Code Exposure** - Users never need to see or understand code
2. **No Git Concepts** - Version control is completely hidden
3. **Real-time Feedback** - Live updates during builds
4. **Progressive Disclosure** - Show complexity only when needed
5. **Mobile Responsive** - Works on all device sizes

---

## Technology Stack

| Component | Technology |
|-----------|------------|
| Framework | Next.js 15 (App Router) |
| React | React 19 |
| Styling | Tailwind CSS |
| Components | shadcn/ui |
| State | Zustand |
| Data Fetching | TanStack Query |
| Real-time | SSE / WebSocket |
| Forms | React Hook Form + Zod |
| Icons | Lucide React |

---

## Page Structure

```mermaid
graph TD
    subgraph Public Pages
        LANDING[/ - Landing Page]
        LOGIN[/login - Login]
        REGISTER[/register - Register]
    end

    subgraph Dashboard
        DASH[/dashboard - Project List]
    end

    subgraph Project Flow
        NEW[/new - New Project Wizard]
        PROJ[/projects/id - Project Detail]
        RUN[/projects/id/runs/runId - Build Watcher]
        EXPORT[/projects/id/export - Export]
    end

    subgraph Settings
        SETTINGS[/settings - User Settings]
        KEYS[/settings/api-keys - API Keys]
    end

    LANDING --> LOGIN
    LANDING --> REGISTER
    LOGIN --> DASH
    REGISTER --> DASH
    DASH --> NEW
    DASH --> PROJ
    PROJ --> RUN
    PROJ --> EXPORT
    DASH --> SETTINGS
    SETTINGS --> KEYS
```

---

## Page Specifications

### Landing Page (/)

**Purpose**: Marketing page for new visitors

**Components**:
- Hero section with value proposition
- Feature highlights
- How it works (3-step process)
- Pricing (hosted mode)
- CTA buttons

**Layout**:
```
┌─────────────────────────────────────────┐
│              Navigation                  │
├─────────────────────────────────────────┤
│                                         │
│              Hero Section               │
│     "Build apps with AI in minutes"     │
│         [Get Started] [Demo]            │
│                                         │
├─────────────────────────────────────────┤
│                                         │
│           Feature Grid (3x2)            │
│                                         │
├─────────────────────────────────────────┤
│                                         │
│            How It Works                 │
│     1. Describe → 2. Review → 3. Ship   │
│                                         │
├─────────────────────────────────────────┤
│              Footer                     │
└─────────────────────────────────────────┘
```

---

### Dashboard (/dashboard)

**Purpose**: List and manage all projects

**Components**:
- Project cards with status indicators
- Search and filter controls
- New project button
- Empty state for new users

**Layout**:
```
┌─────────────────────────────────────────┐
│  Logo    Dashboard    Settings   User ▼ │
├─────────────────────────────────────────┤
│                                         │
│  Your Projects              [+ New App] │
│                                         │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐   │
│  │ App 1   │ │ App 2   │ │ App 3   │   │
│  │ ● Live  │ │ ◐ Build │ │ ○ Draft │   │
│  │ Updated │ │ 45% ... │ │ Created │   │
│  └─────────┘ └─────────┘ └─────────┘   │
│                                         │
│  ┌─────────┐ ┌─────────┐               │
│  │ App 4   │ │ App 5   │               │
│  │ ✗ Failed│ │ ● Live  │               │
│  └─────────┘ └─────────┘               │
│                                         │
└─────────────────────────────────────────┘
```

**Project Card States**:
| State | Icon | Color | Description |
|-------|------|-------|-------------|
| Draft | ○ | Gray | Not yet built |
| Building | ◐ | Blue | Build in progress |
| Live | ● | Green | Successfully built |
| Failed | ✗ | Red | Build failed |

---

### New Project Wizard (/new)

**Purpose**: Guide users through project creation

**Steps**:
1. **Describe** - Enter product description
2. **Clarify** - Answer questions
3. **Review** - Approve spec
4. **Build** - Watch progress

**Step 1: Describe**
```
┌─────────────────────────────────────────┐
│  ← Back                    Step 1 of 4  │
├─────────────────────────────────────────┤
│                                         │
│         What do you want to build?      │
│                                         │
│  App Name                               │
│  ┌─────────────────────────────────┐   │
│  │ My SaaS App                      │   │
│  └─────────────────────────────────┘   │
│                                         │
│  Describe your app                      │
│  ┌─────────────────────────────────┐   │
│  │ A project management tool with   │   │
│  │ team collaboration, task boards, │   │
│  │ and time tracking...             │   │
│  │                                   │   │
│  │                                   │   │
│  └─────────────────────────────────┘   │
│                                         │
│  💡 Tip: Be specific about features     │
│                                         │
│                          [Continue →]   │
│                                         │
└─────────────────────────────────────────┘
```

**Step 2: Clarify**
```
┌─────────────────────────────────────────┐
│  ← Back                    Step 2 of 4  │
├─────────────────────────────────────────┤
│                                         │
│      A few questions to get started     │
│                                         │
│  1. What authentication method?         │
│     ○ Email/Password                    │
│     ○ OAuth (Google, GitHub)            │
│     ● Magic Link                        │
│     ○ All of the above                  │
│                                         │
│  2. How many team members per workspace?│
│     ○ 1-5                               │
│     ● 5-20                              │
│     ○ 20-100                            │
│     ○ Unlimited                         │
│                                         │
│  3. Real-time collaboration?            │
│     [Yes] [No]                          │
│                                         │
│  Question 3 of 5                        │
│                                         │
│                          [Continue →]   │
│                                         │
└─────────────────────────────────────────┘
```

**Step 3: Review**
```
┌─────────────────────────────────────────┐
│  ← Back                    Step 3 of 4  │
├─────────────────────────────────────────┤
│                                         │
│         Review Your App Spec            │
│                                         │
│  ┌─────────────────────────────────┐   │
│  │ Features                         │   │
│  │ ├─ User Authentication          │   │
│  │ │   └─ Magic link login         │   │
│  │ ├─ Team Workspaces              │   │
│  │ │   └─ Up to 20 members         │   │
│  │ ├─ Task Boards                  │   │
│  │ │   └─ Kanban-style             │   │
│  │ └─ Real-time Updates            │   │
│  │     └─ Live collaboration       │   │
│  └─────────────────────────────────┘   │
│                                         │
│  [View Full Spec]                       │
│                                         │
│  ┌─────────────────────────────────┐   │
│  │ 💬 Request changes (optional)    │   │
│  └─────────────────────────────────┘   │
│                                         │
│  [← Make Changes]    [Approve & Build →]│
│                                         │
└─────────────────────────────────────────┘
```

---

### Build Watcher (/projects/:id/runs/:runId)

**Purpose**: Real-time build progress visualization

**Components**:
- Overall progress indicator
- Phase timeline
- Task list with status
- Live log stream
- File change summary

**Layout**:
```
┌─────────────────────────────────────────┐
│  ← My SaaS App              Run #1      │
├─────────────────────────────────────────┤
│                                         │
│  Building your app...           67%     │
│  ████████████████░░░░░░░░░░░░░░░░░░░   │
│                                         │
│  ┌─ Timeline ─────────────────────────┐ │
│  │ ✓ Spec      ✓ Tasks    ◐ Build    │ │
│  │ ○─────────○─────────●─────────○   │ │
│  │                      ↑             │ │
│  │                   Current          │ │
│  └────────────────────────────────────┘ │
│                                         │
│  Tasks                                  │
│  ┌────────────────────────────────────┐ │
│  │ ✓ Project Setup          00:12    │ │
│  │ ✓ Database Schema        00:23    │ │
│  │ ✓ Authentication         00:45    │ │
│  │ ◐ API Routes             01:12... │ │
│  │ ○ UI Components          --:--    │ │
│  │ ○ Tests                  --:--    │ │
│  └────────────────────────────────────┘ │
│                                         │
│  Live Activity                          │
│  ┌────────────────────────────────────┐ │
│  │ Creating src/app/api/tasks/route.ts │ │
│  │ Running pnpm install...             │ │
│  │ ✓ Dependencies installed            │ │
│  └────────────────────────────────────┘ │
│                                         │
└─────────────────────────────────────────┘
```

**Task States**:
| State | Icon | Animation |
|-------|------|-----------|
| Pending | ○ | None |
| Running | ◐ | Spinning |
| Succeeded | ✓ | Checkmark |
| Failed | ✗ | Red X |

---

### Project Detail (/projects/:id)

**Purpose**: Project overview and actions

**Components**:
- Project header with status
- Quick actions (rebuild, export, deploy)
- Build history
- Preview link (if deployed)

**Layout**:
```
┌─────────────────────────────────────────┐
│  ← Dashboard                            │
├─────────────────────────────────────────┤
│                                         │
│  My SaaS App                   ● Live   │
│  Last built 2 hours ago                 │
│                                         │
│  ┌──────────┐ ┌──────────┐ ┌─────────┐ │
│  │ Rebuild  │ │ Export   │ │ Deploy  │ │
│  └──────────┘ └──────────┘ └─────────┘ │
│                                         │
│  Preview                                │
│  ┌────────────────────────────────────┐ │
│  │  🔗 https://my-app.prazium.app    │ │
│  └────────────────────────────────────┘ │
│                                         │
│  Build History                          │
│  ┌────────────────────────────────────┐ │
│  │ Run #3  ● Succeeded   2h ago  [→] │ │
│  │ Run #2  ✗ Failed      5h ago  [→] │ │
│  │ Run #1  ● Succeeded   1d ago  [→] │ │
│  └────────────────────────────────────┘ │
│                                         │
│  Danger Zone                            │
│  ┌────────────────────────────────────┐ │
│  │ [Archive Project]                  │ │
│  └────────────────────────────────────┘ │
│                                         │
└─────────────────────────────────────────┘
```

---

### Export Page (/projects/:id/export)

**Purpose**: Download built application

**Components**:
- Format selector
- Download button
- Deployment instructions

**Layout**:
```
┌─────────────────────────────────────────┐
│  ← My SaaS App                          │
├─────────────────────────────────────────┤
│                                         │
│         Export Your Application         │
│                                         │
│  Format                                 │
│  ┌────────────────────────────────────┐ │
│  │ ● ZIP Archive (.zip)               │ │
│  │ ○ Tarball (.tar.gz)                │ │
│  └────────────────────────────────────┘ │
│                                         │
│  Build                                  │
│  ┌────────────────────────────────────┐ │
│  │ Run #3 (Latest) - 2 hours ago      │ │
│  └────────────────────────────────────┘ │
│                                         │
│           [⬇ Download Export]           │
│                                         │
│  ─────────────────────────────────────  │
│                                         │
│  What's Included                        │
│  • Full source code                     │
│  • Dockerfile                           │
│  • docker-compose.yml                   │
│  • Deployment guide                     │
│  • Environment template                 │
│                                         │
│  Deploy Anywhere                        │
│  [Railway] [Vercel] [Docker] [Manual]   │
│                                         │
└─────────────────────────────────────────┘
```

---

## Components Library

### Core Components

#### Button

```tsx
interface ButtonProps {
  variant: 'primary' | 'secondary' | 'ghost' | 'danger';
  size: 'sm' | 'md' | 'lg';
  loading?: boolean;
  disabled?: boolean;
  icon?: ReactNode;
  children: ReactNode;
}
```

#### Card

```tsx
interface CardProps {
  title?: string;
  description?: string;
  footer?: ReactNode;
  children: ReactNode;
}
```

#### Progress

```tsx
interface ProgressProps {
  value: number;  // 0-100
  variant: 'default' | 'success' | 'error';
  showLabel?: boolean;
  animated?: boolean;
}
```

#### Timeline

```tsx
interface TimelineProps {
  steps: {
    id: string;
    label: string;
    status: 'pending' | 'active' | 'completed' | 'error';
  }[];
}
```

### Build Watcher Components

#### TaskList

```tsx
interface TaskListProps {
  tasks: {
    id: string;
    slug: string;
    status: 'pending' | 'running' | 'succeeded' | 'failed';
    duration?: number;
  }[];
  onTaskClick?: (taskId: string) => void;
}
```

#### ActivityFeed

```tsx
interface ActivityFeedProps {
  events: {
    id: string;
    type: string;
    message: string;
    timestamp: Date;
    taskId?: string;
  }[];
  maxItems?: number;
  autoScroll?: boolean;
}
```

#### FileChanges

```tsx
interface FileChangesProps {
  changes: {
    path: string;
    type: 'created' | 'modified' | 'deleted';
    size?: number;
  }[];
  collapsed?: boolean;
}
```

---

## Real-time Updates

### SSE Connection

```typescript
// hooks/useRunEvents.ts
import { useEffect, useState } from 'react';

interface RunEvent {
  id: string;
  eventType: string;
  taskId?: string;
  payload: any;
  createdAt: string;
}

export function useRunEvents(runId: string) {
  const [events, setEvents] = useState<RunEvent[]>([]);
  const [connected, setConnected] = useState(false);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    const eventSource = new EventSource(
      `/api/v1/projects/${projectId}/runs/${runId}/events/stream`
    );

    eventSource.onopen = () => {
      setConnected(true);
      setError(null);
    };

    eventSource.onmessage = (e) => {
      const event = JSON.parse(e.data);
      setEvents(prev => [...prev, event]);
    };

    eventSource.onerror = (e) => {
      setConnected(false);
      setError(new Error('Connection lost'));
    };

    return () => {
      eventSource.close();
    };
  }, [runId]);

  return { events, connected, error };
}
```

### State Aggregation

```typescript
// hooks/useRunProgress.ts
interface RunProgress {
  status: string;
  phase: string;
  percentage: number;
  tasks: TaskProgress[];
  currentTask?: string;
}

interface TaskProgress {
  id: string;
  slug: string;
  status: string;
  duration?: number;
}

export function useRunProgress(runId: string): RunProgress {
  const { events } = useRunEvents(runId);
  
  return useMemo(() => {
    return aggregateProgress(events);
  }, [events]);
}

function aggregateProgress(events: RunEvent[]): RunProgress {
  const tasks = new Map<string, TaskProgress>();
  let status = 'pending';
  let phase = 'initializing';
  let currentTask: string | undefined;

  for (const event of events) {
    switch (event.eventType) {
      case 'RUN_STARTED':
        status = 'running';
        break;
      case 'RUN_DECOMPOSING':
        phase = 'decomposing';
        break;
      case 'RUN_EXECUTING':
        phase = 'executing';
        break;
      case 'TASK_STARTED':
        tasks.set(event.taskId!, {
          id: event.taskId!,
          slug: event.payload.slug,
          status: 'running',
        });
        currentTask = event.taskId;
        break;
      case 'TASK_SUCCEEDED':
        const task = tasks.get(event.taskId!);
        if (task) {
          task.status = 'succeeded';
          task.duration = event.payload.duration;
        }
        break;
      case 'RUN_SUCCEEDED':
        status = 'succeeded';
        phase = 'complete';
        break;
    }
  }

  const taskList = Array.from(tasks.values());
  const completed = taskList.filter(t => t.status === 'succeeded').length;
  const percentage = taskList.length > 0 
    ? Math.round((completed / taskList.length) * 100)
    : 0;

  return {
    status,
    phase,
    percentage,
    tasks: taskList,
    currentTask,
  };
}
```

---

## Responsive Design

### Breakpoints

| Breakpoint | Width | Target |
|------------|-------|--------|
| sm | 640px | Mobile landscape |
| md | 768px | Tablet |
| lg | 1024px | Desktop |
| xl | 1280px | Large desktop |

### Mobile Adaptations

**Dashboard**:
- Single column card layout
- Bottom navigation
- Swipe gestures for actions

**Build Watcher**:
- Collapsible sections
- Tab-based navigation (Timeline / Tasks / Activity)
- Simplified progress indicator

---

## Accessibility

### Requirements

- WCAG 2.1 AA compliance
- Keyboard navigation
- Screen reader support
- Color contrast ratios
- Focus indicators

### Implementation

```tsx
// Accessible button example
<Button
  aria-label="Start build"
  aria-busy={isLoading}
  aria-disabled={isDisabled}
  role="button"
  tabIndex={0}
  onKeyDown={(e) => {
    if (e.key === 'Enter' || e.key === ' ') {
      onClick();
    }
  }}
>
  {isLoading ? <Spinner aria-hidden /> : null}
  Start Build
</Button>
```

### Focus Management

```tsx
// Auto-focus on new content
useEffect(() => {
  if (buildComplete) {
    successMessageRef.current?.focus();
  }
}, [buildComplete]);
```

---

## Error States

### Error Boundaries

```tsx
// components/ErrorBoundary.tsx
class ErrorBoundary extends React.Component {
  state = { hasError: false, error: null };

  static getDerivedStateFromError(error: Error) {
    return { hasError: true, error };
  }

  render() {
    if (this.state.hasError) {
      return (
        <ErrorCard
          title="Something went wrong"
          message={this.state.error?.message}
          action={<Button onClick={() => window.location.reload()}>Reload</Button>}
        />
      );
    }
    return this.props.children;
  }
}
```

### Error Messages

| Error Type | User Message | Action |
|------------|--------------|--------|
| Network | "Connection lost. Retrying..." | Auto-retry |
| Auth | "Session expired. Please log in." | Redirect to login |
| Not Found | "Project not found." | Back to dashboard |
| Build Failed | "Build failed. See details below." | Show error details |
| Rate Limit | "Too many requests. Please wait." | Show countdown |

---

## Loading States

### Skeleton Screens

```tsx
// components/ProjectCardSkeleton.tsx
function ProjectCardSkeleton() {
  return (
    <Card>
      <Skeleton className="h-6 w-3/4 mb-2" />
      <Skeleton className="h-4 w-1/2 mb-4" />
      <Skeleton className="h-8 w-full" />
    </Card>
  );
}
```

### Progress Indicators

| Context | Indicator |
|---------|-----------|
| Page load | Skeleton screen |
| Button action | Spinner in button |
| Build progress | Progress bar |
| Background task | Toast notification |

---

## Theme

### Colors

```css
:root {
  /* Primary */
  --primary: 222.2 47.4% 11.2%;
  --primary-foreground: 210 40% 98%;
  
  /* Status */
  --success: 142.1 76.2% 36.3%;
  --warning: 38 92% 50%;
  --error: 0 84.2% 60.2%;
  --info: 199 89% 48%;
  
  /* Neutral */
  --background: 0 0% 100%;
  --foreground: 222.2 47.4% 11.2%;
  --muted: 210 40% 96.1%;
  --muted-foreground: 215.4 16.3% 46.9%;
  
  /* Border */
  --border: 214.3 31.8% 91.4%;
  --ring: 222.2 47.4% 11.2%;
}

.dark {
  --background: 222.2 84% 4.9%;
  --foreground: 210 40% 98%;
  /* ... dark mode overrides */
}
```

### Typography

```css
:root {
  --font-sans: 'Inter', system-ui, sans-serif;
  --font-mono: 'JetBrains Mono', monospace;
}

/* Scale */
.text-xs { font-size: 0.75rem; }
.text-sm { font-size: 0.875rem; }
.text-base { font-size: 1rem; }
.text-lg { font-size: 1.125rem; }
.text-xl { font-size: 1.25rem; }
.text-2xl { font-size: 1.5rem; }
.text-3xl { font-size: 1.875rem; }
```

---

## Related Documentation

- [Architecture Overview](./ARCHITECTURE.md)
- [API Specification](./API.md)
- [Event System](./EVENTS.md)
