# hive-invite

Invite a colleague to collaborate on the team's hive repo.

## When to use
When the user says "invite [name] to hive", "add [colleague] to hive", or runs `/hive-invite`.

## Steps

1. **Read roster**
   Read `~/.hive/roster.yml`. Extract `repo` and `me`.

2. **Confirm the invite**
   Show:
   - Repo: <repo>
   - Inviting: <github-username or email>
   - Permission: write
   Ask: "Send this invite? (yes / no)"

3. **Send the invite**
   By GitHub username:
   ```
   gh api \
     --method PUT \
     /repos/<repo>/collaborators/<github-username> \
     --field permission=write
   ```
   By email (if username unknown):
   ```
   gh api \
     --method POST \
     /repos/<repo>/invitations \
     --field email=<email>
   ```

4. **Confirm and share onboarding instructions**
   Tell the user: "Invite sent to <github-username>. Share these setup steps with them:"

   ---
   **hive setup for new participants:**

   1. Accept the GitHub repo invite (check your email)
   2. Clone or browse: `https://github.com/<repo>`
   3. Copy `skills/*.md` to your AI assistant's skills/agents folder:
      - Claude Code: `~/.claude/agents/`
      - Cursor: `.cursor/rules/`
      - Copilot: workspace instructions
   4. Create `~/.hive/roster.yml`:
      ```yaml
      repo: <repo>
      me: <your-github-username>
      members:
        - name: <owner display name>
          github: <owner github username>
      ```
   5. Run `/hive-inbox` to start receiving requests
   ---

5. **Update your local roster (optional)**
   Ask: "Add this person to your local roster? (yes / no)"
   If yes, append to `~/.hive/roster.yml`:
   ```yaml
     - name: <display name>
       github: <github-username>
   ```

## Notes
- Only the repo owner can invite — if the user is not the owner, this will fail with a 403
- The invite expires after 7 days if not accepted
- Do not add anyone to the roster without the user's explicit confirmation