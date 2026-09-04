# hive-check

Check replies on help requests you sent.

## When to use
When the user says "check my hive requests", "any replies?", or runs `/hive-check`.

## Steps

1. **Read roster**
   Read `~/.hive/roster.yml`. Extract `repo` and `me` (your GitHub username).

2. **Fetch your issues with recent activity**
   ```
   gh issue list \
     --repo <repo> \
     --author <me> \
     --label hive-ask \
     --state all \
     --json number,title,state,labels,updatedAt,url \
     --limit 20
   ```

3. **For each issue, check for comments**
   ```
   gh issue view <number> --repo <repo> --json comments,state,labels
   ```

4. **Surface issues with new comments**
   Show a summary for each issue that has comments:
   ```
   #<number> — <title> [<state>]
   <commenter>: <first line of latest comment>
   <url>
   ```
   Group by state: open first, then closed.

5. **If no new activity**
   Report: "No new replies on your hive requests."

6. **Optional: open in browser**
   If the user wants to see the full thread, run:
   ```
   gh issue view <number> --repo <repo> --web
   ```

## Notes
- Show at most 20 recent issues to avoid noise
- Closed issues with `hive-promoted` label are worth highlighting — they have a wiki page