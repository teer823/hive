# hive-inbox

Poll for help requests assigned to you and draft answers for your approval.

## When to use
When the user runs `/hive-inbox`, or when running as a `/loop` every 15 minutes.

**Recommended:** Run as a loop at the start of your session:
```
/loop 15m /hive-inbox
```
Stop it when you close your session. This polls every 15 minutes while you work without needing a persistent background process.

## Steps

1. **Read roster**
   Read `~/.hive/roster.yml`. Extract `repo` and `me` (your GitHub username).

2. **Fetch assigned open issues**
   ```
   gh issue list \
     --repo <repo> \
     --assignee <me> \
     --label hive-ask \
     --state open \
     --json number,title,body,labels,createdAt,url
   ```
   If no issues, report "No open requests" and stop.

3. **For each issue**

   a. **Check for hive-no-ai label**
      If the issue has label `hive-no-ai`, skip it and notify the user: "Issue #<n> is marked human-only — please reply manually."

   b. **Search knowledge archive for existing answer**
      ```
      gh api /repos/<repo>/contents/knowledge --jq '.[].name' 2>/dev/null
      ```
      If relevant pages exist, fetch and include as a source in your draft:
      ```
      gh api /repos/<repo>/contents/knowledge/<file> --jq '.content' | base64 -d
      ```

   c. **Search your own knowledge**
      Use any available vault, notes, or context to find a relevant answer.

   d. **Draft an answer**
      Write a clear, helpful answer in Markdown. Format it as:
      ```
      [claude] <answer body>

      ---
      Confidence: high | medium | low
      Promoted: yes | no
      ```
      Use `[copilot]`, `[cursor]`, etc. if you are a different tool.

   e. **Present draft to user for approval**
      Show:
      - Issue: #<number> — <title>
      - Requester: @<reporter>
      - Draft answer (full text)
      Ask: "Post this answer? (yes / edit / skip)"

   f. **On approval: post comment and close**
      ```
      gh issue comment <number> --repo <repo> --body "<approved answer>"
      gh issue edit <number> --repo <repo> --add-label hive-answered
      gh issue close <number> --repo <repo>
      ```

   g. **On low confidence or user skip**
      Add label `hive-needs-input`:
      ```
      gh issue edit <number> --repo <repo> --add-label hive-needs-input
      ```
      Notify the user: "Issue #<n> flagged as needs-input — please answer manually or ask the requester to clarify."

## Notes
- Never post without user approval — always show the draft first
- Handle each issue one at a time, waiting for user decision before moving to the next
- Do not modify issues with `hive-no-ai` label