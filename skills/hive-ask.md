# hive-ask

Send a help request to a colleague via a GitHub Issue on the team's hive repo.

## When to use
When the user says "ask [colleague]", "send a request to [colleague]", or "hive ask".

## Steps

1. **Read roster**
   Read `~/.hive/roster.yml`. Extract `repo` and the `github` username for the named colleague.
   If the colleague is not in the roster, stop and tell the user.

2. **Search Wiki for existing answer**
   Run: `gh api /repos/<repo>/git/trees/HEAD:wiki --jq '.tree[].path' 2>/dev/null`
   If relevant wiki pages exist, surface them to the user and ask if they still want to create an issue.

3. **Check for duplicate open issues**
   Run: `gh issue list --repo <repo> --label hive-ask --state open --json number,title`
   If a similar title exists, show it to the user and ask if they want to proceed anyway.

4. **Confirm request details with user**
   Show:
   - Assignee: <colleague display name> (@<github-username>)
   - Topic: <topic label>
   - Summary of the request body
   Ask the user to confirm before creating.

5. **Create the issue**
   ```
   gh issue create \
     --repo <repo> \
     --title "[ASK] <short description>" \
     --body "## Request
   <request body>

   ## Context
   <context provided by user>

   ## Topic
   <topic>" \
     --assignee <github-username> \
     --label "hive-ask,topic:<topic>"
   ```
   If the current quarter milestone exists, add `--milestone <quarter>` (e.g. `2026-Q3`).

6. **Report back**
   Tell the user the issue URL and number.

## Notes
- Keep the issue body tool-agnostic — no Claude-specific formatting
- If no topic is specified, use `topic:general`
- Do not hardcode the repo — always read from `~/.hive/roster.yml`