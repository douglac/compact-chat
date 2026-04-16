# compact-chat

**Your AI chat has a memory problem. This skill fixes it.**

Long Cursor sessions lose context. You've been there — 45 minutes of deep debugging, architecture decisions piling up, and suddenly the agent forgets what you decided 10 messages ago. You start a new chat and spend 15 minutes re-explaining everything.

`compact-chat` generates structured handoff documents so you can resume in a new chat **with zero context loss**.

---

## The Problem

```
Chat message #1:   "Let's refactor the auth module"
Chat message #15:  "Good, now let's add the middleware"  
Chat message #30:  "Wait, why did you change the User model?"
Chat message #45:  "You forgot everything we discussed about the JWT approach"
Chat message #60:  *starts new chat, spends 15 min re-explaining*
```

## The Solution

```
You:    "compact"
Agent:  *generates a structured handoff document*
You:    *copies it, opens new chat, pastes it*
Agent:  *picks up exactly where you left off*
```

---

## What You Get

The skill generates a structured document containing:

- **Current state** — what's working, what's broken, where you stopped
- **Decisions made** — with rationale and rejected alternatives
- **Reference documents** — auto-discovered plan files with absolute paths for instant access
- **Relevant files** — what was modified, created, or matters
- **Next steps** — prioritized pending tasks
- **Gotchas** — pitfalls the next chat needs to know about
- **Assumptions** — things taken as truth that should be re-validated

It also handles **resuming** — paste a compact into a new chat and the agent validates the current state, checks freshness, reads referenced plans, and picks up from item #1.

---

## Install

### Via `npx skills` (recommended)

```bash
npx skills add douglac/compact-chat
```

### Via Cursor Settings (native)

1. Open **Cursor Settings** → **Rules**
2. Click **+ Add Rule** → **Remote Rule (GitHub)**
3. Paste: `https://github.com/douglac/compact-chat`

### Manual

```bash
# Global (available in all projects)
mkdir -p ~/.cursor/skills/compact-chat
curl -o ~/.cursor/skills/compact-chat/SKILL.md \
  https://raw.githubusercontent.com/douglac/compact-chat/main/SKILL.md

# Or project-level
mkdir -p .cursor/skills/compact-chat
curl -o .cursor/skills/compact-chat/SKILL.md \
  https://raw.githubusercontent.com/douglac/compact-chat/main/SKILL.md
```

---

## Usage

### Creating a compact (saving context)

Just say **"compact"** in any Cursor chat. The agent will:

1. Collect git status, branch, recent commits
2. Auto-discover all plan/spec/doc files in the project
3. Generate a structured handoff document
4. Present it in a copyable Markdown block

### Resuming from a compact

1. Open a new chat (`Cmd+N`)
2. Paste the compact as the first message
3. The agent validates the current state, reads reference docs, and continues from the first pending task

### Proactive suggestion

After substantial work (5+ file edits, complex debugging, architecture decisions), the agent will suggest generating a compact before context degrades.

---

## Example Output

<details>
<summary>Click to expand a real compact example</summary>

```markdown
## 🔄 Compact: Auth module refactor with JWT middleware

### Metadata
- **Project:** my-saas-app
- **Branch:** feat/auth-refactor
- **Date:** 2026-04-16 14:30 UTC
- **Previous session:** N/A

### Recent commits (context)
- a1b2c3d Add JWT validation middleware
- e4f5g6h Refactor User model to include refresh tokens
- i7j8k9l Extract auth config to environment variables

### Chaining
- **Continues from:** Initial — first compact
- **Supersedes:** None

---

### Current State
Auth middleware is working for protected routes. JWT refresh token rotation 
is implemented but untested. The User model migration is pending — schema 
is updated in code but `prisma migrate` hasn't been run yet.

### What was done
- [x] Extracted auth logic from route handlers into middleware
- [x] Implemented JWT access + refresh token pair generation
- [x] Added token rotation on refresh
- [x] Updated User model with refreshToken and tokenFamily fields

### Decisions made

| Decision | Alternatives considered | Why this choice |
|----------|------------------------|-----------------|
| JWT with refresh rotation | Session-based auth, JWT without refresh | Stateless + secure against token theft via rotation |
| Store refresh token hash in DB | Redis, in-memory | Simpler infra, survives restarts, one less dependency |

### Reference documents (plans, specs, docs)

| Absolute path | What it contains |
|---------------|-----------------|
| `/Users/me/my-saas-app/docs/auth-architecture.plan.md` | Full auth flow diagram, token lifecycle, and security requirements |
| `/Users/me/my-saas-app/docs/database-schema.md` | Current DB schema including User model changes |

### Relevant files

| File | What it is/does | Status |
|------|----------------|--------|
| `src/middleware/auth.ts` | JWT validation middleware | created |
| `src/lib/tokens.ts` | Token generation and rotation logic | created |
| `prisma/schema.prisma` | User model with new token fields | modified |
| `src/routes/auth.ts` | Login/register/refresh endpoints | modified |

### Pending / Next steps
1. [ ] Run prisma migrate to apply User model changes
2. [ ] Write tests for token rotation edge cases
3. [ ] Add rate limiting to auth endpoints
4. [ ] Update API documentation

### Blockers / Open questions
- [ ] Should expired refresh tokens return 401 or 403? (affects frontend error handling)

### ⚠️ Gotchas
- The prisma migration MUST be run before testing — the code expects new fields that don't exist in the DB yet
- Token rotation invalidates the entire token family on reuse detection — this is intentional but will log out all devices

### Assumptions
- Access token TTL is 15 minutes (hardcoded in config, may need to be configurable)
- Single-tenant setup — multi-tenant would need per-tenant signing keys

### Important context
The frontend team is blocked waiting for the auth endpoints. They need the refresh flow working by EOD Thursday.
```

</details>

---

## How It Works

```
┌─────────────────────────────────────────────┐
│  COMPACT FLOW                               │
│                                             │
│  "compact" ──► Collect env info             │
│               ──► Discover plan files        │
│               ──► Generate handoff doc       │
│               ──► Security check (no secrets)│
│               ──► Present copyable block     │
│                                             │
├─────────────────────────────────────────────┤
│  RESUME FLOW                                │
│                                             │
│  Paste compact ──► Read full document       │
│                ──► Verify git state          │
│                ──► Validate assumptions      │
│                ──► Assess freshness          │
│                ──► Read reference docs       │
│                ──► Start from item #1        │
└─────────────────────────────────────────────┘
```

---

## Chaining

Compacts can reference previous compacts, creating a chain of context across multiple sessions:

```
Session 1 → Compact A (Initial)
Session 2 → Compact B (Continues from A, supersedes A)  
Session 3 → Compact C (Continues from B, supersedes A+B)
```

Each compact carries forward only what matters — no noise, no failed attempts, just decisions and state.

---

## Works With

| Agent | Status |
|-------|--------|
| Cursor (IDE + CLI) | ✅ Full support |
| Claude Code | ✅ Compatible (uses `~/.claude/skills/`) |
| Codex CLI | ✅ Compatible (uses `~/.codex/skills/`) |
| Windsurf | ✅ Compatible |
| Cline | ✅ Compatible |

---

## Contributing

Found a bug? Have an idea? Open an issue or PR. The skill is a single `SKILL.md` file — contributions are welcome.

## License

MIT — use it, fork it, share it.
