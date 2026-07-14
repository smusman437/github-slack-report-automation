# GitHub Activity → Slack Summary (n8n Workflow)

🎥 **[Watch the demo](https://github.com/user-attachments/assets/666bb776-09d4-463d-8d79-2968ba9d1737)** — see it fetch activity and post the summary to Slack in real time.

Tracks your GitHub pull requests and commits (across organization and personal repos) and posts a weekly summary to Slack, optionally polished by an AI-generated write-up.

## What this workflow does

- Fetches PRs from your organization repos
- Fetches commits directly from `main`/`master` for personal repos that don't use a PR workflow
- Auto-detects each repo's default branch (no hardcoded `main` vs `master` assumptions)
- Combines everything into one message, with an AI-generated summary on top
- Sends the result to a Slack channel of your choice
- Runs on a schedule (weekly by default, configurable)

## Setup Instructions

### 1. Import the workflow
Import the exported `.json` file into your n8n instance (Workflows → Import from File).

### 2. Create credentials
n8n will prompt you for these on each node that needs them — credentials are **not** included in the export, by design:

| Credential | Where to get it | Scopes needed |
|---|---|---|
| GitHub API | [github.com/settings/tokens](https://github.com/settings/tokens) | `repo`, `read:org` |
| Slack (OAuth2) | Your Slack app config | `chat:write` (bot must be invited to the target channel) |
| OpenAI API | [platform.openai.com](https://platform.openai.com) | Standard API key |

To create each: click the node with a red warning icon → credential dropdown → **Create New** → paste your token/key.

### 3. Configure your details in one place
Open the **`Set Report Config`** node and set:

- `githubUsername` → your GitHub username
- `sinceDate` → day count for the lookback window (`7` = weekly, `1` = daily)
- `reportLabel` → display label, e.g. `"Weekly"` / `"Daily"`

This is the main node you'll touch for personal config — most downstream logic reads from it. The Slack recipient is set separately (see step 5).

### 4. Point the repo list at your own repos
Wherever repos are listed (e.g. in `Loop Repos` or a repo-list Set node), replace the test values with your own org/personal repos. Nothing downstream should reference a hardcoded repo name — all repo-specific logic uses `{{ $json.full_name }}` dynamically.

### 5. Set the Slack recipient
Open the **`Send Slack Message`** node. **Send Message To** is set to `User`, with the message delivered via Slackbot DM. Reselect the **User** field to point at yourself (or whoever should receive the summary) in your own Slack workspace — the imported workflow will still reference the previous owner's user ID until you change it.

### 6. Test before activating
- Click **Execute Workflow** to run it once manually
- Check the Slack message lands correctly in your channel
- Only then enable the **Schedule Trigger** to activate on autopilot

## Notes

- GitHub's PR/commit endpoints are paginated at 100 results/page — the workflow handles this automatically, but very high-volume repos may take a few extra seconds per run.
- The commit-author filter matches on GitHub username; if a personal repo's local git config uses a different name/email, switch that filter to your commit email instead.
- Nothing in this workflow stores your token, key, or channel ID outside of n8n's own credential store — safe to share the JSON + this README with others.
