# hive Protocol Specification

This document defines the canonical schema for hive. All skill files and tool adapters must follow this spec. The protocol lives in GitHub — not in any specific AI tool.

---

## Roster

Each participant maintains a local config at `~/.hive/roster.yml` (personal only — `repo` and `me`).
The shared member list lives in `members.yml` in the team's private fork — visible to all collaborators.

### Local config (`~/.hive/roster.yml`)
```yaml
repo: <github-username>/<team>-hive   # team's private fork
me: <your-github-username>
```

### Shared member list (`members.yml` in repo)
```yaml
members:
  - name: <display-name>        # human-friendly name used in skills (required)
    github: <github-username>   # GitHub username for issue assignment (required)
    role: <job title>           # e.g. "Solution Architect" (optional)
    team: <team or squad>       # e.g. "DevOps", "Design" (optional)
    topics: [<topic>, ...]      # topic areas they know well — maps to hive topic labels (optional)
                                # valid values: auth, architecture, integration, devops, general
```

The `topics` field enables smart routing — `/hive-ask` can suggest the best assignee based on the request's topic.

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
| `hive-promoted` | Answer promoted to /knowledge archive |
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

## Knowledge Archive (`/knowledge` folder)

Promoted answers live in the `knowledge/` folder of the team's private fork — markdown files, readable on GitHub, searchable via `gh` CLI.

When an answer is worth keeping:

1. Create `knowledge/<slug>.md` in the repo
2. Add label `hive-promoted` to the issue and close it
3. Update `knowledge/README.md` index with a one-liner entry

### Knowledge page format

```markdown
# <Topic Title>

> Last updated: YYYY-MM-DD | Source: #<issue-number>

## Summary
<one paragraph — the answer in brief>

## Detail
<full answer>

## Related
- [Other Topic](other-topic.md)
- Issue #<n>
```

### Knowledge index (`knowledge/README.md`)

Serves as the archive index. Add a one-liner for each promoted page:

```markdown
## Index

- [Topic Title](topic-title.md) — one-line summary (from #<issue-number>)
```

---

## Lifecycle Management

To prevent unbounded issue growth:

1. **Archive pre-check** — before creating an issue, search `/knowledge` for an existing answer
2. **Duplicate detection** — search open issues for similar titles before creating
3. **Quarterly milestones** — every issue tagged with a quarter milestone (e.g. `2026-Q3`)
4. **`/hive-tidy`** — maintenance skill that closes stale answered issues, flags unanswered ones, and promotes answers to `/knowledge`

---

## Participation Model

| Tier | Can do |
|---|---|
| **Owner** | Invite, ask, answer, promote to /knowledge |
| **Participant** | Ask, answer, promote to /knowledge, request invites |

- No one can self-add. Join by invitation from the repo owner.
- Each person's local config reflects only `repo` and `me` — member list is shared via `members.yml`.
- The repo itself contains no central member list beyond `members.yml`.