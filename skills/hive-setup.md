# hive-setup

One-time setup for a new team fork. Run this once as the repo owner after forking hive.

## When to use
When the user says "setup hive", "initialize hive", or runs `/hive-setup`.
Only the repo owner needs to run this — once, on a fresh fork.

## Steps

1. **Read roster**
   Read `~/.hive/roster.yml`. Extract `repo` and `me`.

2. **Confirm**
   Show: "This will initialize `<repo>` with hive labels and milestones. Run as owner once only."
   Ask: "Proceed? (yes / no)"

3. **Create labels**
   Run each of the following:
   ```
   gh label create "hive-ask"          --repo <repo> --color "0075ca" --description "All help requests"
   gh label create "hive-answered"     --repo <repo> --color "0e8a16" --description "Has a usable answer"
   gh label create "hive-promoted"     --repo <repo> --color "6f42c1" --description "Answer promoted to wiki"
   gh label create "hive-needs-input"  --repo <repo> --color "e4e669" --description "Requester needs to clarify"
   gh label create "hive-no-ai"        --repo <repo> --color "d93f0b" --description "Human-only, do not auto-answer"
   gh label create "topic:auth"        --repo <repo> --color "bfd4f2" --description "Authentication / authorization"
   gh label create "topic:architecture"--repo <repo> --color "bfd4f2" --description "System / solution architecture"
   gh label create "topic:integration" --repo <repo> --color "bfd4f2" --description "API / system integration"
   gh label create "topic:devops"      --repo <repo> --color "bfd4f2" --description "CI/CD, infrastructure, deployment"
   gh label create "topic:general"     --repo <repo> --color "bfd4f2" --description "General topic"
   ```
   If a label already exists, skip it (add `2>/dev/null || true` to each command).

4. **Create current quarter milestone**
   Determine the current quarter (e.g. `2026-Q3` for July–September 2026).
   ```
   gh api --method POST /repos/<repo>/milestones \
     --field title="<quarter>" \
     --field description="hive requests for <quarter>"
   ```
   If the milestone already exists, skip.

5. **Initialize Wiki Home page**
   Clone the wiki repo and create `Home.md`:
   ```
   git clone https://github.com/<repo>.wiki.git /tmp/hive-wiki-setup
   ```
   Create `/tmp/hive-wiki-setup/Home.md`:
   ```markdown
   # hive Knowledge Archive

   Promoted answers from the team's help requests.
   Add a one-liner here when a new page is promoted.

   ## Index

   <!-- entries added by /hive-tidy as answers are promoted -->
   ```
   Push:
   ```
   cd /tmp/hive-wiki-setup && git add Home.md && git commit -m "Initialize hive wiki" && git push
   rm -rf /tmp/hive-wiki-setup
   ```
   If the wiki clone fails (wiki not yet enabled), tell the user: "Enable the wiki on GitHub repo settings first, then re-run `/hive-setup`."

6. **Report**
   Tell the user: "hive setup complete for `<repo>`. Labels, milestone, and wiki initialized. You're ready to invite colleagues with `/hive-invite`."

## Notes
- This is owner-only and runs once — it is safe to re-run (all steps are idempotent)
- Do not delete existing labels if the repo was already partially set up
- The wiki must be enabled in GitHub repo Settings → Features → Wikis before step 5 will work