# hive-sync

Show the current team member list from the shared repo.

## When to use
When the user says "sync hive", "show hive members", "who's in hive", or runs `/hive-sync`.
Also suggest running this when a colleague is not found during `/hive-ask`.

## Steps

1. **Read local roster**
   Read `~/.hive/roster.yml`. Extract `repo` and `me`.

2. **Fetch members.yml from repo**
   ```
   gh api /repos/<repo>/contents/members.yml --jq '.content' | base64 -d
   ```

3. **Display team directory**
   Show a rich member list:
   ```
   hive team — teer823/ibmdt-hive

   Name        GitHub           Role                  Team      Topics
   ─────────────────────────────────────────────────────────────────────
   Tle         @teer823         Portfolio Architect   IBMDT     architecture, auth, integration
   Lookchin    @lukeatdesign     Designer             Design    general
   Joke        @wsaikliang      DevOps Engineer       DevOps    devops, integration
   ```
   For members with no optional fields, show only name and GitHub.

4. **Report**
   Tell the user: "Use these names with /hive-ask. Topic-based routing will suggest the best match automatically."

## Notes
- This skill is read-only — it does not modify any files
- members.yml in the repo is always up to date for collaborators
- Run this when a colleague name isn't recognized in `/hive-ask`
- The `topics` column shows what each person knows well — used by `/hive-ask` for smart routing