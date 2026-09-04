# hive-sync

Sync the shared member list from the repo into your local knowledge.

## When to use
When the user says "sync hive", "update hive members", or runs `/hive-sync`.
Also suggest running this when a colleague is not found during `/hive-ask`.

## Steps

1. **Read local roster**
   Read `~/.hive/roster.yml`. Extract `repo` and `me`.

2. **Fetch members.yml from repo**
   ```
   gh api /repos/<repo>/contents/members.yml --jq '.content' | base64 -d
   ```

3. **Display current member list**
   Show a clean list:
   ```
   hive members (teer823/ibmdt-hive):
   - Tle (@teer823)
   - Lookchin (@lukeatdesign)
   ```

4. **Report**
   Tell the user: "Member list is up to date. Use these names with /hive-ask."

## Notes
- This skill is read-only — it does not modify any files
- The member list lives in the repo's `members.yml` and is always up to date for collaborators
- Run this when a colleague name isn't recognized in `/hive-ask`