# hive-tidy

Clean up stale issues and promote valuable answers to the knowledge archive.

## When to use
When the user says "tidy hive", "clean up hive", or runs `/hive-tidy`. Run at end of quarter or when issue volume feels high.

## Steps

### Part 1: Close stale answered issues

1. **Read local roster**
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

### Part 3: Promote to knowledge archive

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
   "Promote this answer to /knowledge? (yes / no)"
   Show: issue title, first comment body.

3. **On approval: create knowledge page**
   Slugify the title: lowercase, spaces → hyphens, remove `[ASK]` prefix and special chars.

   Fetch the top comment:
   ```
   gh issue view <number> --repo <repo> --json comments \
     --jq '.comments[0].body'
   ```

   Create the file locally at `/tmp/hive-knowledge-<slug>.md`:
   ```markdown
   # <Title without [ASK] prefix>

   > Last updated: <today YYYY-MM-DD> | Source: #<issue-number>

   ## Summary
   <one paragraph summary>

   ## Detail
   <full answer from comment>

   ## Related
   - Issue #<number>
   ```

   Push to `/knowledge/<slug>.md` in the repo:
   ```
   gh api --method PUT /repos/<repo>/contents/knowledge/<slug>.md \
     --field message="Promote #<number> to knowledge archive" \
     --field content="$(base64 < /tmp/hive-knowledge-<slug>.md)"
   rm /tmp/hive-knowledge-<slug>.md
   ```

4. **Update issue labels**
   ```
   gh issue edit <number> --repo <repo> --add-label hive-promoted
   ```

5. **Update knowledge/README.md index**
   Fetch current README (create if missing):
   ```
   gh api /repos/<repo>/contents/knowledge/README.md \
     --jq '.content' | base64 -d > /tmp/hive-knowledge-index.md 2>/dev/null \
     || echo "# hive Knowledge Archive\n\n## Index\n" > /tmp/hive-knowledge-index.md
   ```
   Append one-liner:
   ```
   echo "- [<Title>](/<slug>.md) — <one-line summary> (from #<number>)" >> /tmp/hive-knowledge-index.md
   ```
   Push updated index:
   ```
   SHA=$(gh api /repos/<repo>/contents/knowledge/README.md --jq '.sha' 2>/dev/null || echo "")
   gh api --method PUT /repos/<repo>/contents/knowledge/README.md \
     --field message="Update knowledge index" \
     --field content="$(base64 < /tmp/hive-knowledge-index.md)" \
     $([ -n "$SHA" ] && echo "--field sha=$SHA")
   rm /tmp/hive-knowledge-index.md
   ```

## Notes
- Never promote without user approval — always show the draft first
- Slugify the title for the filename: lowercase, spaces → hyphens, remove special chars
- Keep knowledge pages generic — remove personal names or org-specific references before promoting