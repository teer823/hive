# hive

> Async help-request and collective knowledge protocol for teams using AI coding assistants.

**hive** lets you send help requests to colleagues through GitHub Issues, get AI-drafted answers, and build a shared knowledge archive — all using portable skill files that work with Claude Code, Copilot, Cursor, or any AI assistant that can run shell commands.

---

## How it works

1. You ask a colleague for help → a GitHub Issue is created in your team's private fork
2. Their AI assistant picks it up, drafts an answer, and presents it for approval
3. They post the answer as a comment → issue closed
4. Valuable answers get promoted to the GitHub Wiki → shared knowledge archive

No real-time connection needed. Everyone participates at their own pace.

---

## Setup

### Prerequisites
- [GitHub CLI (`gh`)](https://cli.github.com/) installed and authenticated (`gh auth login`)
- Access to a forked team repo (see **Forking for your team** below)

### 1. Fork for your team
Fork this repo to a **private** repo for your team:
```
gh repo fork teer823/hive --clone --fork-name <your-team>-hive
```
Or create a new private repo and copy the `skills/` folder.

### 2. Copy skill files to your AI tool
Copy the files from `skills/` to your AI assistant's skills/agents folder:

| Tool | Folder |
|---|---|
| Claude Code | `~/.claude/agents/` |
| Cursor | `.cursor/rules/` or agent config |
| Copilot | Workspace instructions or agent config |

### 3. Set up your local roster
Create `~/.hive/roster.yml`:
```yaml
# ~/.hive/roster.yml
repo: <github-username>/<your-team>-hive   # your team's private fork
me: <your-github-username>

members:
  - name: Alice
    github: alice-gh
  - name: Bob
    github: bob-gh
```

### 4. Start receiving requests
Run `/hive-inbox` in your AI assistant. It will poll for issues assigned to you every 15 minutes.

---

## Skills

| Skill | What it does |
|---|---|
| `hive-ask` | Send a help request to a colleague |
| `hive-inbox` | Poll for and answer requests assigned to you |
| `hive-check` | Check replies on your open requests |
| `hive-tidy` | Clean up stale issues, promote answers to wiki |
| `hive-invite` | Invite a colleague as a collaborator |

---

## Forking for your team

The **public `hive` repo** contains only the generic protocol and skill files — no team data, no usernames.

Your **private team fork** is where:
- Issues (help requests) live
- GitHub Wiki (knowledge archive) lives
- Team-specific skill customizations live (optional)

Protocol improvements should be PRed back to this repo. Team data stays private.

---

## Contributing

PRs welcome for:
- Improving skill instructions for better portability
- Adding support for new AI tools
- Protocol improvements (new labels, status flows, etc.)

Please keep skill files tool-agnostic — no Claude-specific syntax.

---

## License

MIT