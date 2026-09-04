# hive-invite

Invite a colleague to collaborate on the team's hive repo.

## When to use
When the user says "invite [name] to hive", "add [colleague] to hive", or runs `/hive-invite`.

## Steps

1. **Read roster**
   Read `~/.hive/roster.yml`. Extract `repo` and `me`.

2. **Collect colleague details**
   Ask the user for both:
   - Display name (e.g. "Somchai") — used in the roster
   - GitHub username or email — used for the invite

   If the user already provided these in their message, use those values directly without asking again.

3. **Confirm the invite**
   Show:
   - Repo: <repo>
   - Name: <display name>
   - GitHub: <github-username or email>
   - Permission: write
   Ask: "Send invite and add to roster? (yes / no)"

4. **Send the invite**
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

5. **Add to local roster**
   Append to `~/.hive/roster.yml`:
   ```yaml
     - name: <display name>
       github: <github-username>
   ```
   Do this automatically on approval — no separate prompt needed.

6. **Confirm and share onboarding instructions**
   Tell the user: "Invite sent to <display name> (@<github-username>) and added to your roster. Share these setup steps with them:"

   ---
   **hive setup for new participants:**

   **Prerequisites:**
   - Install GitHub CLI: `brew install gh` (Mac) or https://cli.github.com
   - Authenticate: `gh auth login`

   **Steps:**
   1. Accept the GitHub repo invite (check your email)
   2. Open `ONBOARDING.md` in the repo: `https://github.com/<repo>/blob/main/ONBOARDING.md`
   3. Copy the bootstrap prompt and paste it into your AI assistant — it will set everything up automatically
   ---

## Notes
- Always collect display name AND GitHub username before proceeding — both are needed
- Only the repo owner can invite — if the user is not the owner, this will fail with a 403
- The invite expires after 7 days if not accepted
- Roster update happens automatically on confirmation — no separate prompt