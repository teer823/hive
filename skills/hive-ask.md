# hive-ask

Send a help request to a colleague via a GitHub Issue on the team's hive repo.

## When to use
When the user says "ask [colleague]", "send a request to [colleague]", or "hive ask".
If no colleague is named, use the `topics` field in members.yml to suggest the best person.

## Steps

1. **Read local roster**
   Read `~/.hive/roster.yml`. Extract `repo` and `me`.

2. **Read shared member list**
   Fetch `members.yml` from the repo:
   ```
   gh api /repos/<repo>/contents/members.yml --jq '.content' | base64 -d
   ```

3. **Resolve assignee**
   - If the user named a colleague: look up their `github` username from members.yml. Stop if not found — suggest `/hive-sync` to refresh.
     - **Topic mismatch guardrail:** If the colleague's `topics` list does not include the request's topic, warn the user:
       ```
       ⚠️  Joke's topics are [devops, integration] — this looks like a design question.
       Better match: Lookchin (Designer, topics: design, ux-ui, figma)
       Still send to Joke? (yes / reassign to Lookchin)
       ```
       Let the user decide — do not override their choice.
   - If no colleague named: look at the request topic and match against each member's `topics` list. Suggest the best match and ask the user to confirm.
   - If multiple members match the topic: list them with their `role` and `team` and let the user choose.

4. **Search knowledge archive for existing answer**
   Fetch the index first (one API call):
   ```
   gh api /repos/<repo>/contents/knowledge/README.md --jq '.content' | base64 -d
   ```
   Scan the index entries for relevance to the user's request. If any entry looks relevant, fetch only that page:
   ```
   gh api /repos/<repo>/contents/knowledge/<file> --jq '.content' | base64 -d
   ```
   Surface any relevant pages to the user and ask if they still want to create an issue. If no match in the index, skip fetching individual pages.

5. **Check for duplicate open issues**
   ```
   gh issue list --repo <repo> --label hive-ask --state open --json number,title
   ```
   If a similar title exists, show it and ask if the user wants to proceed anyway.

6. **Confirm request details with user**
   Show:
   - Assignee: <display name> (<role>, <team>) @<github-username>
   - Topic: <topic label>
   - Summary of the request body
   Ask the user to confirm before creating.

7. **Create the issue**
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

8. **Report back**
   Tell the user the issue URL and number.

## Notes
- Keep the issue body tool-agnostic — no Claude-specific formatting
- If no topic is specified, use `topic:general`
- Always read members from the repo's `members.yml`, not a local file
- Smart routing is a suggestion only — the user always confirms the assignee