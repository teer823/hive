# hive-ask

Send a help request to a colleague via a GitHub Issue on the team's hive repo.

## When to use
When the user says "ask [colleague]", "send a request to [colleague]", or "hive ask".

## Steps

1. **Read local roster**
   Read `~/.hive/roster.yml`. Extract `repo` and `me`.

2. **Read shared member list**
   Fetch `members.yml` from the repo:
   ```
   gh api /repos/<repo>/contents/members.yml --jq '.content' | base64 -d
   ```
   Parse the `members` list to find the named colleague's `github` username.
   If the colleague is not found, stop and tell the user. Suggest running `/hive-sync` to refresh.

3. **Search Wiki for existing answer**
   ```
   gh api /repos/<repo>/wiki/pages 2>/dev/null
   ```
   If relevant wiki pages exist, surface them and ask if the user still wants to create an issue.

4. **Check for duplicate open issues**
   ```
   gh issue list --repo <repo> --label hive-ask --state open --json number,title
   ```
   If a similar title exists, show it and ask if the user wants to proceed anyway.

5. **Confirm request details with user**
   Show:
   - Assignee: <colleague display name> (@<github-username>)
   - Topic: <topic label>
   - Summary of the request body
   Ask the user to confirm before creating.

6. **Create the issue**
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
   If the current quarter milestone exists, add `--milestone <quarter>`.

7. **Report back**
   Tell the user the issue URL and number.

## Notes
- Keep the issue body tool-agnostic — no Claude-specific formatting
- If no topic is specified, use `topic:general`
- Always read members from the repo's `members.yml`, not local file