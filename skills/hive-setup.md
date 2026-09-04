# hive-setup

One-time setup for a new team fork. Run this once as the repo owner after forking hive.

## When to use
When the user says "setup hive", "initialize hive", or runs `/hive-setup`.
Only the repo owner needs to run this — once, on a fresh fork.

## Steps

1. **Read roster**
   Read `~/.hive/roster.yml`. Extract `repo` and `me`.

2. **Confirm**
   Show: "This will initialize `<repo>` with hive labels, milestones, and knowledge archive. Run as owner once only."
   Ask: "Proceed? (yes / no)"

3. **Create labels**
   Run each of the following (add `2>/dev/null || true` to skip if already exists):
   ```
   gh label create "hive-ask"           --repo <repo> --color "0075ca" --description "All help requests"
   gh label create "hive-answered"      --repo <repo> --color "0e8a16" --description "Has a usable answer"
   gh label create "hive-promoted"      --repo <repo> --color "6f42c1" --description "Answer promoted to knowledge archive"
   gh label create "hive-needs-input"   --repo <repo> --color "e4e669" --description "Requester needs to clarify"
   gh label create "hive-no-ai"         --repo <repo> --color "d93f0b" --description "Human-only, do not auto-answer"
   gh label create "topic:auth"         --repo <repo> --color "bfd4f2" --description "Authentication / authorization"
   gh label create "topic:architecture" --repo <repo> --color "bfd4f2" --description "System / solution architecture"
   gh label create "topic:integration"  --repo <repo> --color "bfd4f2" --description "API / system integration"
   gh label create "topic:devops"       --repo <repo> --color "bfd4f2" --description "CI/CD, infrastructure, deployment"
   gh label create "topic:general"      --repo <repo> --color "bfd4f2" --description "General topic"
   ```

4. **Create current quarter milestone**
   Determine the current quarter (e.g. `2026-Q3` for July–September 2026).
   ```
   gh api --method POST /repos/<repo>/milestones \
     --field title="<quarter>" \
     --field description="hive requests for <quarter>" 2>/dev/null || true
   ```

5. **Initialize knowledge archive**
   Create `knowledge/README.md` in the repo:
   ```
   gh api --method PUT /repos/<repo>/contents/knowledge/README.md \
     --field message="Initialize knowledge archive" \
     --field content="$(echo '# hive Knowledge Archive

Promoted answers from the team'"'"'s help requests.

## Index

<!-- entries added by /hive-tidy as answers are promoted -->
' | base64)"
   ```

6. **Report**
   Tell the user: "hive setup complete for `<repo>`. Labels, milestone, and knowledge archive initialized. You're ready to invite colleagues with `/hive-invite`."

## Notes
- This is owner-only and runs once — it is safe to re-run (all steps are idempotent)
- Do not delete existing labels if the repo was already partially set up