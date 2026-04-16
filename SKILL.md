---
name: compact-chat
description: "Generate structured handoff documents to transfer full context between AI chat sessions. Use when: (1) user says 'compact', 'checkpoint', 'save progress', 'handoff', (2) context is getting long or slow, (3) user wants to resume from a previous handoff with 'resume', 'continue where I left off', 'load handoff'. Proactively suggest after substantial work (5+ file edits, complex debugging, architecture decisions)."
---

# Compact Chat

Generate structured handoff documents to transfer context between chat sessions with zero ambiguity. Solves the context exhaustion problem in long-running AI coding sessions.

## Mode Selection

Determine which mode applies:

**COMPACT (create handoff)?** User wants to save state, pause, or chat is getting long.
→ Follow the COMPACT flow below

**RESUME (restore)?** User wants to continue previous work, pasted a handoff into chat.
→ Follow the RESUME flow below

**Proactive suggestion?** After substantial work (5+ file edits, complex debugging, architecture decisions), suggest:
> "We've made good progress. Want me to generate a compact to preserve context? Say 'compact' when ready."

---

## COMPACT Flow

### Step 1: Collect environment info

Before generating the document, collect project data:

```bash
# Branch and git status
git branch --show-current 2>/dev/null
git log --oneline -5 --no-decorate 2>/dev/null
git diff --name-only 2>/dev/null
git diff --name-only --cached 2>/dev/null
pwd
```

Then search for all plan/spec files in the workspace:

```bash
# Find plan files (*.plan.md, *.plan, plan-*.md, etc.)
find . -type f \( -name "*.plan.md" -o -name "*.plan" -o -name "plan-*.md" -o -name "plan.md" -o -name "*_plan_*.md" -o -name "*-plan-*.md" \) -not -path "*/node_modules/*" -not -path "*/.git/*" 2>/dev/null | sort

# Also check docs/ and common documentation folders
find . -type f -name "*.md" -path "*/docs/*" -not -path "*/node_modules/*" 2>/dev/null | sort
```

**Important:** Include ALL plan files found in the "Reference documents" section of the compact, with full absolute paths. These documents contain architecture decisions, flows, and critical context the next chat needs.

### Step 2: Generate the handoff document

Using collected info + all context from the current chat, generate the document following the template below. **Fill ALL sections** — no placeholders. If a section doesn't apply, write "N/A" with a brief reason.

### Step 3: Present to the user

Deliver the document in a copyable Markdown block. Tell the user:
1. Copy the block with Cmd+C (or Ctrl+C)
2. Open a new chat with Cmd+N
3. Paste as the first message
4. Continue where you left off

### Step 4: Security check

Before delivering, mentally verify:
- [ ] No API keys, tokens, passwords, or secrets in the document
- [ ] No connection strings with credentials
- [ ] File paths are relative to the project (not absolute system paths)

If any secret is detected, remove it and warn the user.

---

## Compact Template

Generate EXACTLY in this format, filled with real data from the chat:

````markdown
## 🔄 Compact: [descriptive task title]

### Metadata
- **Project:** [project name or path]
- **Branch:** [current git branch]
- **Date:** [timestamp]
- **Previous session:** [chat link if available, or "N/A"]

### Recent commits (context)
- [short hash] [commit message]
- ...

### Chaining
- **Continues from:** [reference to previous compact, or "Initial — first compact"]
- **Supersedes:** [old compacts this one makes obsolete, or "None"]

---

### Current State
[One paragraph describing: what was being done, where it stopped, what's working and what isn't]

### What was done
- [x] [Completed task 1 — brief description]
- [x] [Completed task 2]
- ...

### Decisions made

| Decision | Alternatives considered | Why this choice |
|----------|------------------------|-----------------|
| [Decision 1] | [Options A, B, C] | [Rationale] |

### Reference documents (plans, specs, docs)

Planning and documentation files found in the project. **Use absolute paths** so the next chat can read them directly.

| Absolute path | What it contains |
|---------------|-----------------|
| `/full/path/to/docs/example.plan.md` | [brief description of the plan's content/purpose] |

> **Note:** These files are the source of truth for architecture decisions, flows, schemas, and specs. The next chat MUST read the relevant ones before continuing.

### Relevant files

| File | What it is/does | Status |
|------|----------------|--------|
| `relative/path/to/file` | [description] | [modified/created/unchanged] |

### Pending / Next steps
1. [ ] [Most critical action — what to do first]
2. [ ] [Second priority]
3. [ ] [Third priority]

### Blockers / Open questions
- [ ] [Blocker or question — what's needed to resolve]

### ⚠️ Gotchas
- [Thing that can go wrong if the next chat doesn't know]
- [Non-obvious side effect, hidden dependency, etc.]

### Assumptions
- [Assumption 1 that was taken as truth]
- [Assumption 2 — validate if still true when resuming]

### Important context
[Any critical information that doesn't fit the categories above but the next chat NEEDS to know to continue without asking]
````

---

## RESUME Flow

When the user pastes a compact into chat or asks to resume:

### Step 1: Read the entire compact

Read the full document before taking any action.

### Step 2: Verify current state

Run the commands below and compare with what's in the compact:

```bash
git branch --show-current
git status
git log --oneline -5
```

### Step 3: Validation checklist

Before starting work, verify:

- [ ] Current branch matches the compact (or understand why it changed)
- [ ] Listed files still exist
- [ ] Assumptions are still valid
- [ ] Blockers have been resolved or are still pending
- [ ] Read the "Gotchas" section to avoid known pitfalls

### Step 4: Assess freshness

Compare the compact's date with the current state:
- **Same day, few commits:** FRESH — can resume directly
- **1-2 days, some commits:** SLIGHTLY STALE — review changes first
- **3+ days or many commits:** STALE — consider re-exploring before continuing

### Step 5: Read reference documents

If the compact contains a "Reference documents" section, read the most relevant plan files for the next pending task. These contain critical architecture context and decisions.

### Step 6: Start with item #1

Begin with the first pending item in "Next steps". Reference the "Decisions made", "Reference documents", and "Gotchas" sections while working.

### Step 7: Update or chain

Throughout the session:
- Mark completed items in the pending list
- If the session gets long, generate a new compact referencing this one as previous

---

## Rules

1. **Fill everything** — the goal is that the next chat understands everything without needing to ask
2. **Relative paths** — use paths relative to the project for code files. **Exception:** reference documents (plans, specs, docs) MUST use full absolute paths so the next chat can read them directly
3. **Don't include code** — only file references. The new chat can read the files
4. **Filter noise** — ignore failed intermediate attempts and tangents. Focus on what worked and what's left
5. **No secrets** — never include API keys, tokens, passwords. Only environment variable names
6. **Decisions with rationale** — not just "chose X", but "chose X because Y and Z didn't work for this reason"
7. **Gotchas are mandatory** — always list at least one pitfall, even if minor. If none, write "None identified so far"
8. **Copyable format** — output must be a Markdown block the user copies with Cmd+C and pastes into a new chat
9. **Plans are mandatory** — always search and list ALL plan/spec/doc files found in the project, with absolute paths. If none found, write "No plan files found in the project"
