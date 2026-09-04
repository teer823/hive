# hive-tidy

Clean up stale issues and promote valuable answers to the GitHub Wiki.

## When to use
When the user says "tidy hive", "clean up hive", or runs `/hive-tidy`. Run at end of quarter or when issue volume feels high.

## Steps

### Part 1: Close stale answered issues

1. **Read roster**
   Read `~/.hive/roster.yml`. Extract `repo`.

2. **Fetch old answered issues**
   ```
   gh issue list \
     --repo <repo> \
     --label hive-answered \
     --state open \
     --json number,title,updatedAt,url
   ```
   Filter for issues not updated in the last 30 days.

3. **Show list to user for confirmation**
   Show the list and ask: "Close these answered issues? (yes / review each / skip)"

4. **On approval: close in batch**
   For each confirmed issue:
   ```
   gh issue close <number> --repo <repo>
   ```

---

### Part 2: Flag unanswered issues

1. **Fetch open issues with no hive-answered label**
   ```
   gh issue list \
     --repo <repo> \
     --label hive-ask \
     --state open \
     --json number,title,assignee,createdAt,url
   ```
   Filter for issues older than 7 days with no `hive-answered` label.

2. **Report to user**
   Show the list: "These issues have no answer yet — consider following up or reassigning."

---

### Part 3: Promote to Wiki

1. **Fetch closed issues with comments but no hive-promoted label**
   ```
   gh issue list \
     --repo <repo> \
     --label hive-answered \
     --state closed \
     --json number,title,body,url \
     --limit 20
   ```

2. **For each issue, show the answer and ask**
   "Promote this answer to the Wiki? (yes / no)"
   Show: issue title, first comment body.

3. **On approval: create Wiki page**
   Create a file locally:
   ```markdown
   # <Issue Title without [ASK] prefix>

   > Last updated: <today YYYY-MM-DD> | Source: #<issue-number>

   ## Summary
   <one paragraph summary>

   ## Detail
   <full answer from comment>

   ## Related
   - Issue #<number>
   ```

   Push to wiki:
   ```
   gh api --method PUT \
     /repos/<repo>/contents/wiki/<slug>.md \
     --field message="Promote #<number> to wiki" \
     --field content="$(base64 < <file>)"
   ```
   Or clone the wiki repo and push:
   ```
   git clone https://github.com/<repo>.wiki.git /tmp/hive-wiki
   cp <file> /tmp/hive-wiki/<slug>.md
   cd /tmp/hive-wiki && git add . && git commit -m "Promote #<number>" && git push
   ```

4. **Update issue labels**
   ```
   gh issue edit <number> --repo <repo> --add-label hive-promoted
   ```

5. **Update Wiki Home.md**
   Add a one-liner to the index:
   ```
   - [[<Title>]] — <one-line summary> (from #<number>)
   ```

## Notes
- Never promote without user approval — always show the draft first
- Slugify the title for the wiki filename: lowercase, spaces → hyphens, remove special chars
- Keep wiki pages generic — remove any personal names or org-specific references before promoting