# Plan: Update Documentation to Use Claude Code (Claude Agent SDK)

## Overview

This plan outlines the changes needed to update Prazium's documentation to use **Claude Code** (Claude Agent SDK) for all AI/code generation tasks. We will use the **TypeScript SDK**, maintain the existing **Docker sandbox architecture**, and update all code examples to show Claude Agent SDK patterns.

## Key Changes Summary

| Current State | New State |
|---------------|-----------|
| Generic "LLM Provider" / "OpenAI / Anthropic" | Claude Code (Claude Agent SDK) |
| Custom agent runtime with manual LLM calls | Claude Agent SDK with built-in tools |
| Manual file/command tool implementations | SDK-provided tools (Bash, Read, Write, Edit, Glob) |
| `LLM_API_KEY` environment variable | `ANTHROPIC_API_KEY` environment variable |

## Claude Agent SDK Overview

The Claude Agent SDK provides:
- **`query()`** - Async function for simple one-off requests
- **`ClaudeSDKClient`** - Bidirectional interactive conversations with custom tools and hooks
- **Built-in tools**: Bash, Read, Write, Edit, Glob, Grep, LS, etc.
- **Custom MCP tools** - Define additional tools as TypeScript functions
- **Streaming responses** - Real-time output for UI updates

**Documentation**: https://platform.claude.com/docs/en/agent-sdk/overview
**TypeScript SDK**: https://github.com/anthropics/claude-agent-sdk-typescript

---

## Files to Update

### 1. docs/AGENTS.md (Major Rewrite)

**Current**: Custom agent runtime with manual LLM client, file operations, and command execution.

**Changes**:
- Replace custom `Agent` class with Claude Agent SDK `ClaudeSDKClient`
- Replace manual `LLMClient` with SDK's built-in Claude integration
- Replace custom file/command tools with SDK's built-in tools
- Update Docker container to include Claude Agent SDK
- Update event emission to use SDK hooks
- Update error handling to use SDK error types

**Key Code Changes**:

```typescript
// OLD: Custom agent
const agent = new Agent({
  taskDefinition: taskDef,
  workspacePath: '/workspace',
  llmApiKey: process.env.LLM_API_KEY!,
});

// NEW: Claude Agent SDK
import { ClaudeSDKClient, ClaudeAgentOptions } from '@anthropic-ai/claude-agent-sdk';

const client = new ClaudeSDKClient({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const result = await client.query({
  prompt: buildTaskPrompt(taskDef),
  options: {
    allowedTools: ['Bash', 'Read', 'Write', 'Edit', 'Glob', 'Grep'],
    workingDirectory: '/workspace',
    maxTurns: 50,
  },
  hooks: {
    onToolStart: (tool, input) => emit('TOOL_STARTED', { tool, input }),
    onToolEnd: (tool, output) => emit('TOOL_COMPLETED', { tool, output }),
  },
});
```

**Sections to Update**:
- Agent Architecture diagram - update to show Claude Agent SDK
- Agent Lifecycle sequence diagram
- Agent Runtime section - complete rewrite
- LLM Integration section - replace with Claude Agent SDK patterns
- Command Execution - reference SDK's Bash tool
- Error Handling - update error types

---

### 2. docs/ARCHITECTURE.md (Moderate Changes)

**Changes**:
- Update Technology Stack table: `LLM | OpenAI / Anthropic API` → `AI Agent | Claude Code (Claude Agent SDK)`
- Update diagrams to show "Claude Agent SDK" instead of "LLM Provider"
- Update failure modes table for Claude API

**Specific Updates**:

Line 400 - Technology Stack:
```markdown
| AI Agent | Claude Code (Claude Agent SDK) | Agentic code generation with built-in tools |
```

Line 468 - Network note:
```markdown
2. Sandboxes cannot reach external services (except Claude API via proxy)
```

Line 482 - Failure modes:
```markdown
| Claude API down | API error | Retry with backoff |
```

---

### 3. docs/SDLC.md (Minor Changes)

**Changes**:
- Update sequence diagrams: "LLM Provider" → "Claude Agent SDK"
- Update parallel execution description to mention Claude agents

**Specific Updates**:

Lines 134-140, 176-191, 305-311 - Sequence diagrams:
- Change `participant LLM as LLM Provider` to `participant CLAUDE as Claude Agent SDK`
- Update arrow labels accordingly

Line 408:
```markdown
**Purpose**: Execute tasks concurrently using Claude Code agents
```

---

### 4. docs/SECURITY.md (Moderate Changes)

**Changes**:
- Update secret management: `LLM_API_KEY` → `ANTHROPIC_API_KEY`
- Update LLM API Proxy section for Claude-specific considerations
- Update network diagrams to show "Claude API"

**Specific Updates**:

Line 25 - Assets table:
```markdown
| Anthropic API keys | Critical | Injected at runtime only |
```

Lines 302-325 - LLM API Proxy section:
```typescript
// Worker-side Claude API proxy
class ClaudeAPIProxy {
  private rateLimiter: RateLimiter;
  private validator: RequestValidator;
  
  async forward(request: ClaudeRequest): Promise<ClaudeResponse> {
    // Validate request
    this.validator.validate(request);
    
    // Rate limit per Anthropic guidelines
    await this.rateLimiter.acquire();
    
    // Forward to Claude API
    const response = await this.claudeClient.messages.create(request);
    
    return response;
  }
}
```

Lines 367-378 - Environment variables:
```bash
# API Server
ANTHROPIC_API_KEY=<from-vault>

# Worker
ANTHROPIC_API_KEY=<from-vault>

# Sandbox (minimal)
# ANTHROPIC_API_KEY injected at runtime only
```

Line 388 - Key rotation:
```typescript
anthropicApiKey: {
  rotationInterval: '30d';
  gracePeriod: '24h';
};
```

---

### 5. docs/DEPLOYMENT.md (Minor Changes)

**Changes**:
- Update environment variable examples
- Update docker-compose configuration

**Specific Updates**:

Lines 91-93, 122-124 - Docker compose:
```yaml
- ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
```

Lines 218-227 - Environment template:
```bash
# ===================
# Claude Code Configuration
# ===================
ANTHROPIC_API_KEY=sk-ant-your-anthropic-key

# Optional: Claude model selection
CLAUDE_MODEL=claude-sonnet-4-20250514
```

---

### 6. docs/PRD.md (Minor Changes)

**Changes**:
- Update AI agent references to mention Claude Code

**Specific Updates**:

Line 17:
```markdown
* **parallel Claude Code agents** operating in isolated sandboxes
```

Line 29:
```markdown
* Scale horizontally using parallel Claude Code agents
```

Line 180:
```markdown
* One Claude Code agent per worktree
```

Line 336:
```markdown
* System runs offline except for Claude API access
```

---

### 7. docs/README.md (Minor Changes)

**Changes**:
- Update technology stack overview
- Update architecture diagram

**Specific Updates**:

Line 51 - Diagram:
```mermaid
subgraph External
    CLAUDE[Claude Agent SDK]
end
```

Line 62:
```markdown
AGENT --> CLAUDE
```

Line 136 - Technology table:
```markdown
| AI | Claude Code (Claude Agent SDK) |
```

Lines 151-152:
```markdown
### Agents
Claude Code agents execute tasks in isolated Docker sandboxes. They use the Claude Agent SDK's built-in tools for file operations and command execution within their scope.
```

---

## New Content to Add

### Claude Agent SDK Integration Guide (in AGENTS.md)

Add a new section explaining the SDK integration:

```markdown
## Claude Agent SDK Integration

Prazium uses the [Claude Agent SDK](https://platform.claude.com/docs/en/agent-sdk/overview) for all code generation tasks.

### SDK Configuration

```typescript
import { ClaudeSDKClient } from '@anthropic-ai/claude-agent-sdk';

const client = new ClaudeSDKClient({
  apiKey: process.env.ANTHROPIC_API_KEY,
  model: process.env.CLAUDE_MODEL || 'claude-sonnet-4-20250514',
});
```

### Built-in Tools

The SDK provides these tools out of the box:

| Tool | Description | Usage |
|------|-------------|-------|
| `Bash` | Execute shell commands | Running builds, tests, git operations |
| `Read` | Read file contents | Analyzing existing code |
| `Write` | Write file contents | Creating new files |
| `Edit` | Edit existing files | Modifying code with search/replace |
| `Glob` | Find files by pattern | Discovering project structure |
| `Grep` | Search file contents | Finding code patterns |
| `LS` | List directory contents | Exploring directories |

### Custom Tools

Additional tools can be defined for Prazium-specific operations:

```typescript
const customTools = [
  {
    name: 'emit_event',
    description: 'Emit a task event for the UI',
    parameters: {
      type: 'object',
      properties: {
        eventType: { type: 'string' },
        payload: { type: 'object' },
      },
    },
    execute: async (params) => {
      emitEvent(params.eventType, params.payload);
      return { success: true };
    },
  },
];
```

### Hooks for Observability

```typescript
const hooks = {
  onMessageStart: (message) => {
    logger.info('Claude started response', { id: message.id });
  },
  onToolStart: (tool, input) => {
    emitEvent('TOOL_STARTED', { tool: tool.name, input });
  },
  onToolEnd: (tool, output) => {
    emitEvent('TOOL_COMPLETED', { tool: tool.name, output });
  },
  onError: (error) => {
    logger.error('Claude error', { error });
  },
};
```
```

---

## Implementation Order

1. **docs/AGENTS.md** - Major rewrite (highest impact, most complex)
2. **docs/ARCHITECTURE.md** - Update technology stack and diagrams
3. **docs/SECURITY.md** - Update secret management and proxy docs
4. **docs/DEPLOYMENT.md** - Update environment variables
5. **docs/SDLC.md** - Update sequence diagrams
6. **docs/PRD.md** - Minor terminology updates
7. **docs/README.md** - Update overview and tech stack

---

## Validation Checklist

After updates, verify:

- [ ] All references to "LLM" are updated to "Claude" or "Claude Agent SDK"
- [ ] All references to "OpenAI" are removed
- [ ] Environment variable `LLM_API_KEY` is replaced with `ANTHROPIC_API_KEY`
- [ ] Code examples use Claude Agent SDK TypeScript patterns
- [ ] Diagrams show "Claude Agent SDK" instead of "LLM Provider"
- [ ] Docker sandbox architecture is preserved
- [ ] Built-in SDK tools are documented
- [ ] Custom tool extension pattern is documented
- [ ] Hooks for observability are documented

---

## Questions Resolved

1. **SDK Choice**: TypeScript SDK (`@anthropic-ai/claude-agent-sdk`)
2. **Architecture**: Keep Docker sandbox, use Claude Agent SDK inside
3. **Code Examples**: Update to show actual SDK patterns (`query()`, `ClaudeSDKClient`, hooks)
