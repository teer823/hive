# hive

> Async help-request and collective knowledge protocol for teams using AI coding assistants.

**hive** lets you send help requests to colleagues through GitHub Issues, get AI-drafted answers, and build a shared knowledge archive — all using portable skill files that work with Claude Code, Copilot, Cursor, or any AI assistant that can run shell commands.

No real-time connection needed. Everyone works at their own pace.

---

## How it works

```
You                          Colleague
 │                               │
 ├─ /hive-ask @colleague         │
 │   └─ GitHub Issue created ────┤
 │                               ├─ /hive-inbox (polls every 15 min)
 │                               ├─ AI drafts answer
 │                               ├─ Colleague approves
 │                               └─ Comment posted, issue closed
 │                               │
 ├─ /hive-check ─────────────────┤
 │   └─ Reply surfaced           │
 │                               │
 └─ Valuable answers promoted to GitHub Wiki (shared archive)
```

---

## Quick start (team owner)

### 1. Prerequisites
- Install [GitHub CLI](https://cli.github.com/): `brew install gh`
- Authenticate: `gh auth login`

### 2. Fork for your team
```
gh repo fork teer823/hive --fork-name <your-team>-hive
gh repo clone <your-github-username>/<your-team>-hive
```

### 3. Install skills
Copy `skills/*.md` to your AI assistant's commands folder:

| Tool | Folder |
|---|---|
| Claude Code | `~/.claude/commands/` |
| Cursor | `.cursor/rules/` |
| Copilot | Workspace instructions |

### 4. Create your roster
```yaml
# ~/.hive/roster.yml
repo: <your-github-username>/<your-team>-hive
me: <your-github-username>

members:
  - name: Alice
    github: alice-gh
  - name: Bob
    github: bob-gh
```

### 5. Initialize the repo (owner only, once)
```
/hive-setup
```
Creates labels, milestones, and wiki. Safe to re-run.

### 6. Invite colleagues
```
/hive-invite
```
Sends a GitHub collaborator invite and outputs ready-to-share onboarding instructions.

---

## Skills

| Skill | Who | What it does |
|---|---|---|
| `/hive-setup` | Owner, once | Initialize labels, milestones, wiki on a fresh fork |
| `/hive-invite` | Owner | Invite a colleague + generate onboarding instructions |
| `/hive-ask` | Anyone | Send a help request to a colleague |
| `/hive-inbox` | Anyone | Poll for requests assigned to you, draft answers |
| `/hive-check` | Anyone | Check replies on requests you sent |
| `/hive-tidy` | Anyone | Clean up stale issues, promote answers to wiki |

---

## For new participants

If you were invited to a team's hive repo, see the `ONBOARDING.md` file in that repo — it has a single copy-paste prompt you can give your AI assistant to get set up automatically.

---

## Two-repo model

| Repo | Visibility | What lives here |
|---|---|---|
| **`hive`** (this repo) | Public | Protocol spec, generic skill files |
| **`<team>-hive`** (your fork) | Private | Issues, Wiki, your team's data |

Team data stays private. Protocol improvements go back upstream via PR.

---

## Contributing

PRs welcome for:
- Skill improvements and portability fixes
- Support for new AI tools
- Protocol improvements (labels, status flows, etc.)

Keep skill files tool-agnostic — no Claude-specific syntax.

---

## License

MIT