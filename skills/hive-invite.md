# hive-invite

Invite a colleague to collaborate on the team's hive repo.
- **Owner:** sends the invite directly and updates members.yml
- **Participant:** sends a permission request to the owner via a hive issue

## When to use
When the user says "invite [name] to hive", "add [colleague] to hive", or runs `/hive-invite`.

## Arguments
```
/hive-invite <github-username> <display-name>
```
Both arguments are required. Only these two are collected — use `/hive-update` to add role, team, and topics later.

## Steps

1. **Read local roster**
   Read `~/.hive/roster.yml`. Extract `repo` and `me`.

2. **Collect colleague details**
   Use `<github-username>` and `<display-name>` from the arguments.
   If either is missing, ask for only those two — nothing else.

3. **Check if user is owner**
   ```
   gh api /repos/<repo> --jq '.owner.login'
   ```
   - If `me` == owner → **Owner flow**
   - If `me` != owner → **Participant flow**

---

### Owner flow

4. **Confirm the invite**
   Show:
   - Repo: <repo>
   - Name: <display name>
   - GitHub: <github-username>
   - Permission: write
   Ask: "Send invite and add to members.yml? (yes / no)"

5. **Send the invite**
   ```
   gh api --method PUT /repos/<repo>/collaborators/<github-username> \
     --field permission=write
   ```

6. **Add to shared members.yml in repo**
   Fetch current file and sha:
   ```
   gh api /repos/<repo>/contents/members.yml --jq '.sha' > /tmp/hive-members-sha
   gh api /repos/<repo>/contents/members.yml --jq '.content' | base64 -d > /tmp/hive-members.yml
   ```
   Append new member (name and github only):
   ```yaml
     - name: <display name>
       github: <github-username>
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
   Tell the user: "Invite sent to <display name> (@<github-username>) and added to members.yml. Share these steps:"

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
   - A permission request will be sent to the repo owner
   Ask: "Send this request to the owner? (yes / no)"

5. **Create a permission-request issue assigned to owner**
   ```
   gh issue create \
     --repo <repo> \
     --title "[ASK] Permission to invite @<github-username>" \
     --body "## Request
   Requesting permission to invite a new member.

   - **Name:** <display name>
   - **GitHub:** @<github-username>
   - **Requested by:** @<me>

   ## Context
   Approve by running: /hive-invite <github-username> <display name>

   ## Topic
   general" \
     --assignee <owner-github-username> \
     --label "hive-ask,topic:general"
   ```

6. **Report back**
   Tell the user: "Permission request sent to the owner. They'll be notified via /hive-inbox."

---

## Notes
- Use `/hive-update` to add or change role, team, and topics for any member after invite
- Only the repo owner can send the actual GitHub invite — participants must request permission
- The invite expires after 7 days if not accepted