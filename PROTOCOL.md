# hive Protocol Specification

This document defines the canonical schema for hive. All skill files and tool adapters must follow this spec. The protocol lives in GitHub — not in any specific AI tool.

---

## Roster

Each participant maintains a local roster at `~/.hive/roster.yml`. The repo contains no central member list.

```yaml
repo: <github-username>/<team>-hive   # team's private fork
me: <your-github-username>

members:
  - name: <display-name>              # human-friendly name used in skills
    github: <github-username>         # GitHub username for issue assignment
```

---

## Issue Schema

Every help request is a GitHub Issue on the team's private fork.

### Title
```
[ASK] <short description>
```
Always prefixed with `[ASK]`.

### Body
```markdown
## Request
<what you need — be specific>

## Context
<background, links, relevant files>

## Topic
<topic tag — matches a label below>
```

### Fields

| Field | Value |
|---|---|
| **Assignee** | Target colleague's GitHub username |
| **Labels** | `hive-ask` + one topic label (see below) |
| **Milestone** | Current quarter, e.g. `2026-Q3` |

---

## Label Taxonomy

### Status labels

| Label | Meaning |
|---|---|
| `hive-ask` | All help requests — primary filter |
| `hive-answered` | Has a usable answer in comments |
| `hive-promoted` | Answer promoted to GitHub Wiki |
| `hive-needs-input` | Requester needs to clarify before answer is possible |
| `hive-no-ai` | Human-only — do not auto-draft answer |

### Topic labels

| Label | Meaning |
|---|---|
| `topic:auth` | Authentication / authorization |
| `topic:architecture` | System / solution architecture |
| `topic:integration` | API / system integration |
| `topic:devops` | CI/CD, infrastructure, deployment |
| `topic:general` | Does not fit other topics |

---

## Status Workflow

```
Open
  └─ Assigned, label: hive-ask
       ├─ Answered → add hive-answered → Close
       ├─ Promoted → add hive-promoted → Close
       └─ Needs clarification → add hive-needs-input → stay Open
```

---

## Comment Format

All AI-generated comments must follow this format:

```
[<tool>] <answer body>

---
Confidence: high | medium | low
Promoted: yes | no
```

Where `<tool>` is one of: `claude`, `copilot`, `cursor`, `human`.

Example:
```
[claude] The recommended approach is to use JWT tokens stored in httpOnly cookies...

---
Confidence: high
Promoted: no
```

---

## Knowledge Archive (GitHub Wiki)

When an answer is worth keeping:

1. Create a Wiki page: `<Topic Title>.md`
2. Add label `hive-promoted` to the issue and close it
3. Link the Wiki page in the issue body

### Wiki page format

```markdown
# <Topic Title>

> Last updated: YYYY-MM-DD | Source: #<issue-number>

## Summary
<one paragraph — the answer in brief>

## Detail
<full answer>

## Related
- [[Other Topic]]
- Issue #<n>
```

### Wiki Home page

`Home.md` serves as the archive index. Add a one-liner entry for each promoted page:

```markdown
## Archive Index

- [[Topic Title]] — one-line summary (from #<issue-number>)
```

---

## Lifecycle Management

To prevent unbounded issue growth:

1. **Archive pre-check** — before creating an issue, search the Wiki for an existing answer
2. **Duplicate detection** — search open issues for similar titles before creating
3. **Quarterly milestones** — every issue tagged with a quarter milestone (e.g. `2026-Q3`)
4. **`hive-tidy`** — maintenance skill that closes stale answered issues and flags unanswered ones

---

## Participation Model

| Tier | Can do |
|---|---|
| **Owner** | Invite, ask, answer, promote to wiki |
| **Participant** | Ask, answer, promote to wiki |

- No one can self-add. Join by invitation from the repo owner.
- Each person's local roster reflects only who they've chosen to work with.
- The repo itself contains no central member list.