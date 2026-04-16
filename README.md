# compact-chat

**Every token your AI re-reads is money you're burning.**

Long Cursor and Claude Code sessions accumulate massive context. The model re-processes every previous message on each turn — and you pay for every token. A 60-message session doesn't cost 60x the first message. It costs **1 + 2 + 3 + ... + 60 = 1,830x**. That's quadratic cost growth for linear work.

At some point the context window fills up, the model starts hallucinating, and you open a new chat — only to spend 15 minutes (and more tokens) re-explaining everything.

`compact-chat` compresses an entire session into a structured handoff document. Paste it into a fresh chat and **resume with full context at a fraction of the token cost**.

---

## The Problem

```
Message #1:    50 tokens    →  you pay 50
Message #10:   50 tokens    →  you pay 500  (model re-reads all 10)
Message #30:   50 tokens    →  you pay 1,500
Message #60:   50 tokens    →  you pay 3,000

Total session: ~91,500 tokens consumed for 3,000 tokens of actual content
```

Eventually the context fills, quality drops, and you start over — paying again to re-establish what the model already knew.

## The Fix

```
You:    "compact"
Agent:  *distills 60 messages into ~800 tokens of structured context*
You:    *opens new chat, pastes the compact*
Agent:  *resumes instantly — fresh context window, full knowledge, minimal tokens*
```

**One compact replaces thousands of tokens of stale conversation.** Your next session starts lean and accurate instead of bloated and confused.

---

## Why This Matters

| Metric | Without compact-chat | With compact-chat |
|--------|---------------------|-------------------|
| Context after 60 messages | ~90k tokens (full history) | ~800 tokens (structured summary) |
| Resume cost in new chat | ~5k tokens re-explaining | ~800 tokens (paste compact) |
| Model accuracy | Degrades as context grows | Fresh — peak accuracy |
| Architecture decisions | Buried in conversation noise | Explicit table with rationale |
| Plan files & specs | Forgotten | Auto-discovered, absolute paths |

---

## What You Get

The skill generates a structured handoff document containing:

- **Current state** — what's working, what's broken, where you stopped
- **Decisions made** — with rationale and rejected alternatives
- **Reference documents** — auto-discovered plan/spec files with absolute paths for instant access
- **Relevant files** — what was modified, created, or matters
- **Next steps** — prioritized pending tasks
- **Gotchas** — pitfalls the next session needs to know
- **Assumptions** — things taken as truth that should be re-validated

It also handles **resuming** — paste a compact into a new chat and the agent validates the current state, checks freshness, reads referenced plans, and picks up from item #1.

---

## Install

### One-liner (works for both Cursor and Claude Code)

```bash
npx skills add douglac/compact-chat
```

This installs to `~/.agents/skills/compact-chat` and copies to the right directory for your agent.

---

### Cursor

**Option A — via Settings (native import)**

1. Open **Cursor Settings** → **Rules**
2. Click **+ Add Rule** → **Remote Rule (GitHub)**
3. Paste: `https://github.com/douglac/compact-chat`

**Option B — manual**

```bash
mkdir -p ~/.cursor/skills/compact-chat
curl -o ~/.cursor/skills/compact-chat/SKILL.md \
  https://raw.githubusercontent.com/douglac/compact-chat/main/SKILL.md
```

After installing, type `/compact-chat` or just say "compact" in any Agent chat.

---

### Claude Code

**Option A — via `npx skills`**

```bash
npx skills add douglac/compact-chat --agent claude-code
```

**Option B — manual**

```bash
mkdir -p ~/.claude/skills/compact-chat
curl -o ~/.claude/skills/compact-chat/SKILL.md \
  https://raw.githubusercontent.com/douglac/compact-chat/main/SKILL.md
```

After installing, invoke with `/compact-chat` or just say "compact" in any session. Claude Code auto-discovers skills from `~/.claude/skills/` — no extra config needed.

**Option C — project-level (share with your team)**

```bash
mkdir -p .claude/skills/compact-chat
curl -o .claude/skills/compact-chat/SKILL.md \
  https://raw.githubusercontent.com/douglac/compact-chat/main/SKILL.md
```

Commit to git and everyone on the team gets it.

---

## Usage

### Creating a compact (saving context)

Say **"compact"** in any chat. The agent will:

1. Collect git status, branch, recent commits
2. Auto-discover all plan/spec/doc files in the project
3. Generate a structured handoff document
4. Present it in a copyable Markdown block

### Resuming from a compact

1. Open a new chat (`Cmd+N`)
2. Paste the compact as the first message
3. The agent validates the current state, reads reference docs, and continues from the first pending task

### Proactive suggestion

After substantial work (5+ file edits, complex debugging, architecture decisions), the agent suggests generating a compact before context degrades and costs spiral.

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
| `/Users/me/my-saas-app/docs/auth-architecture.plan.md` | Full auth flow diagram, token lifecycle, security requirements |
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
┌─────────────────────────────────────────────────┐
│  COMPACT FLOW                                   │
│                                                 │
│  "compact" ──► Collect git + env info           │
│             ──► Discover plan/spec files         │
│             ──► Distill session into structured  │
│                 handoff (~800 tokens)            │
│             ──► Security check (strip secrets)   │
│             ──► Present copyable Markdown block  │
│                                                 │
├─────────────────────────────────────────────────┤
│  RESUME FLOW                                    │
│                                                 │
│  Paste compact ──► Parse full document          │
│                ──► Verify git state matches      │
│                ──► Validate assumptions           │
│                ──► Assess freshness (stale?)      │
│                ──► Read referenced plan files     │
│                ──► Start from pending item #1     │
└─────────────────────────────────────────────────┘
```

---

## Chaining

Compacts reference previous compacts, creating a chain of context across sessions:

```
Session 1 → Compact A (Initial)
Session 2 → Compact B (Continues from A, supersedes A)  
Session 3 → Compact C (Continues from B, supersedes A+B)
```

Each compact carries forward only what matters — no noise, no failed attempts, just decisions and state. The chain never grows because each new compact supersedes the old ones.

---

## Compatibility

| Agent | Install path | Status |
|-------|-------------|--------|
| **Cursor** (IDE + CLI) | `~/.cursor/skills/` | ✅ Full support |
| **Claude Code** | `~/.claude/skills/` | ✅ Compatible |
| Codex CLI | `~/.codex/skills/` | ✅ Compatible |
| Windsurf | `.cursor/skills/` | ✅ Compatible |
| Cline | `.cursor/skills/` | ✅ Compatible |

---

## Contributing

Found a bug? Have an idea? Open an issue or PR. The skill is a single `SKILL.md` file — contributions are welcome.

## License

MIT — use it, fork it, share it.
