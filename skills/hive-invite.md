# hive-invite

Invite a colleague to collaborate on the team's hive repo.
- **Owner:** sends the invite directly and updates members.yml
- **Participant:** sends a permission request to the owner via a hive issue

## When to use
When the user says "invite [name] to hive", "add [colleague] to hive", or runs `/hive-invite`.

## Steps

1. **Read local roster**
   Read `~/.hive/roster.yml`. Extract `repo` and `me`.

2. **Collect colleague details**
   Ask the user for:
   - Display name (e.g. "Somchai") — required
   - GitHub username — required
   - Role (e.g. "Solution Architect") — optional, press Enter to skip
   - Team (e.g. "DevOps", "Design") — optional, press Enter to skip
   - Topics they know well — optional, comma-separated from: `auth`, `architecture`, `integration`, `devops`, `general`

   If the user already provided name and GitHub username in their message, use those directly and only ask for the optional metadata fields.

3. **Check if user is owner**
   Run:
   ```
   gh api /repos/<repo> --jq '.owner.login'
   ```
   Compare to `me` from the local roster.

   - If `me` == owner → follow **Owner flow** below
   - If `me` != owner → follow **Participant flow** below

---

### Owner flow

4. **Confirm the invite**
   Show:
   - Repo: <repo>
   - Name: <display name>
   - GitHub: <github-username>
   - Role: <role or "—">
   - Team: <team or "—">
   - Topics: <topics or "—">
   - Permission: write
   Ask: "Send invite and add to members.yml? (yes / no)"

5. **Send the invite**
   ```
   gh api \
     --method PUT \
     /repos/<repo>/collaborators/<github-username> \
     --field permission=write
   ```

6. **Add to shared members.yml in repo**
   Fetch current file and sha:
   ```
   gh api /repos/<repo>/contents/members.yml --jq '.sha' > /tmp/hive-members-sha
   gh api /repos/<repo>/contents/members.yml --jq '.content' | base64 -d > /tmp/hive-members.yml
   ```
   Append new member (include only fields that were provided):
   ```yaml
     - name: <display name>
       github: <github-username>
       role: <role>        # omit if not provided
       team: <team>        # omit if not provided
       topics: [<topics>]  # omit if not provided
   ```
   Push updated file:
   ```
   gh api --method PUT /repos/<repo>/contents/members.yml \
     --field message="Add <display name> to members" \
     --field content="$(base64 < /tmp/hive-members.yml)" \
     --field sha="$(cat /tmp/hive-members-sha)"
   rm /tmp/hive-members.yml /tmp/hive-members-sha
   ```

7. **Share onboarding instructions**
   Tell the user: "Invite sent to <display name> (@<github-username>) and added to members.yml. Share these steps with them:"

   ---
   **hive setup for new participants:**

   **Prerequisites:**
   - Install GitHub CLI: `brew install gh` (Mac) or https://cli.github.com
   - Authenticate: `gh auth login`

   **Steps:**
   1. Accept the GitHub repo invite (check your email)
   2. Open `ONBOARDING.md`: `https://github.com/<repo>/blob/main/ONBOARDING.md`
   3. Copy the bootstrap prompt and paste it into your AI assistant
   ---

---

### Participant flow

4. **Confirm the request**
   Show:
   - Requesting invite for: <display name> (@<github-username>)
   - Role / Team / Topics if provided
   - A permission request will be sent to the repo owner
   Ask: "Send this request to the owner? (yes / no)"

5. **Create a permission-request issue assigned to owner**
   Fetch owner username:
   ```
   gh api /repos/<repo> --jq '.owner.login'
   ```
   Create issue:
   ```
   gh issue create \
     --repo <repo> \
     --title "[ASK] Permission to invite @<github-username>" \
     --body "## Request
   Requesting permission to invite a new member to ibmdt-hive.

   - **Name:** <display name>
   - **GitHub:** @<github-username>
   - **Role:** <role or not provided>
   - **Team:** <team or not provided>
   - **Topics:** <topics or not provided>
   - **Requested by:** @<me>

   ## Context
   Please approve by running: /hive-invite <github-username> <display name>

   ## Topic
   general" \
     --assignee <owner-github-username> \
     --label "hive-ask,topic:general"
   ```

6. **Report back**
   Tell the user: "Permission request sent to the owner. They'll be notified via /hive-inbox and can approve by running /hive-invite."

---

## Notes
- Only the repo owner can send the actual GitHub invite — participants must request permission
- The invite expires after 7 days if not accepted
- members.yml in the repo is the source of truth — all collaborators see it automatically
- Optional metadata fields (role, team, topics) can be added later by editing members.yml directly