# hive-update

Update a member's metadata in the shared members.yml.

## When to use
When the user says "update [name]'s info", "set [name]'s role", "add topics for [name]", or runs `/hive-update`.

## Arguments
```
/hive-update <name> [field=value ...]
```
Examples:
```
/hive-update Joke role="DevOps Engineer" team=DevOps topics=devops,integration
/hive-update Lookchin team=Design topics=general
/hive-update Tle topics=architecture,auth,integration
```

## Steps

1. **Read local roster**
   Read `~/.hive/roster.yml`. Extract `repo`.

2. **Fetch members.yml from repo**
   ```
   gh api /repos/<repo>/contents/members.yml --jq '.sha' > /tmp/hive-members-sha
   gh api /repos/<repo>/contents/members.yml --jq '.content' | base64 -d > /tmp/hive-members.yml
   ```

3. **Find the member**
   Match `<name>` (case-insensitive) against the `name` field in members.yml.
   If not found, show the current member list and stop.

4. **Show current values and proposed changes**
   Show a before/after diff:
   ```
   Updating Joke (@wsaikliang):
     role:   — → DevOps Engineer
     team:   — → DevOps
     topics: — → [devops, integration]
   ```
   Ask: "Apply these changes? (yes / no)"

5. **Apply changes to members.yml**
   Update the matched member's entry with the new fields.
   Valid fields: `role`, `team`, `topics` (comma-separated list).
   Omit fields not provided — do not clear existing values unless explicitly set to empty.

6. **Push updated file**
   ```
   gh api --method PUT /repos/<repo>/contents/members.yml \
     --field message="Update <name> metadata" \
     --field content="$(base64 < /tmp/hive-members.yml)" \
     --field sha="$(cat /tmp/hive-members-sha)"
   rm /tmp/hive-members.yml /tmp/hive-members-sha
   ```

7. **Confirm**
   Tell the user: "<name>'s profile updated in members.yml."

## Notes
- Any collaborator can update member metadata — not owner-only
- To clear a field, set it to empty: `role=""`
- Valid topic values: `auth`, `architecture`, `integration`, `devops`, `general`
- Changes are visible to all collaborators immediately after push