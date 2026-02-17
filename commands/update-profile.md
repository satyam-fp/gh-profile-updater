---
allowed-tools: Bash(git log:*), Bash(git remote:*), Bash(git branch:*), Bash(gh api:*), Bash(gh pr:*)
description: Generate an impact-driven summary from git activity and update your GitHub profile README
---

## Context

- Recent commits: !`git log --oneline -20`
- Current repo remote: !`git remote get-url origin`
- Current branch: !`git branch --show-current`

## Instructions

Follow these steps **exactly**:

### Step 1 — Gather Data

1. **Git commits**: Review the 20 recent commits provided above. Parse each commit message for what was accomplished.
2. **Session context**: Review the current conversation history for any tasks completed, bugs fixed, features built, or concepts explored during this session.
3. **Extended context** (optional): If the user explicitly asks for a broader scan, also consider open PRs, recent branches, and any other signals of recent work visible in the conversation.

### Step 2 — Classify Progress

Categorize every meaningful item from Step 1 into exactly one of these three buckets:

| Category | Icon | What belongs here |
|---|---|---|
| **Shipping** | :rocket: | New features, completed projects, first releases, shipped endpoints, deployed services |
| **Learning** | :books: | New libraries adopted, languages explored, research papers read, unfamiliar domains entered |
| **Refining** | :wrench: | Optimizations, bug fixes, performance improvements, architectural refactors, test coverage gains |

### Step 3 — Write the Summary

Compose the summary following these rules:

- **Tone**: Professional, high-energy, and technical — but accessible to a non-expert reader.
- **Focus on impact, not activity.** Write *"Optimized 3D mesh generation speed by 20%"* instead of *"Worked on mesh code."*
- **Format**: Choose whichever fits the volume of material better:
  - **Bulleted list** (when there are 3+ distinct items):
    ```
    - :rocket: Shipped a real-time WebSocket notification service handling 10k concurrent connections.
    - :books: Picked up Rust's `tokio` async runtime for the first time.
    - :wrench: Cut Docker image size from 1.2 GB to 340 MB by switching to a multi-stage build.
    ```
  - **3-sentence narrative** (when items are closely related):
    ```
    Shipped v2.0 of the analytics dashboard with sub-100ms query latency. Learned BigQuery's optimized JOIN strategies along the way. Refactored the ETL pipeline to reduce daily compute costs by 35%.
    ```

### Step 4 — Review & Edit

**Hard stop — do not proceed past this step until the user explicitly approves.**

1. Present the draft summary to the user inside a fenced code block.
2. Ask the user to review it. They may request edits, rewording, additions, or removals.
3. If the user requests changes, revise the summary and present it again.
4. Only proceed to Step 5 once the user gives explicit approval (e.g., "looks good", "approved", "go ahead").

### Step 5 — Read Profile README via API

1. Determine the GitHub username by extracting the owner from the repo remote URL above.
2. Fetch the current profile README via the GitHub API:
   ```bash
   gh api repos/{username}/{username}/contents/README.md
   ```
3. Extract the `content` field (base64-encoded) and decode it to get the current README text.
4. Extract the `sha` field — this is required for updating the file later.
5. If the profile repo or README doesn't exist, inform the user and stop.

### Step 6 — Archive Old Velocity

1. Look for existing content between `<!-- RECENT-VELOCITY:START -->` and `<!-- RECENT-VELOCITY:END -->` in the decoded README.
2. If there is existing content between the tags **and** it differs from the new summary:
   - Extract the old content.
   - Look for a `<!-- PAST-PROJECTS:START -->` / `<!-- PAST-PROJECTS:END -->` section in the README.
   - If the PAST-PROJECTS section **does not exist**, create it immediately after the `<!-- RECENT-VELOCITY:END -->` tag:
     ```markdown

     ## 📦 Past Projects

     <!-- PAST-PROJECTS:START -->
     <!-- PAST-PROJECTS:END -->
     ```
   - Insert the archived content at the **top** of the PAST-PROJECTS section (newest first), formatted with a month/year header:
     ```markdown
     ### Feb 2026
     {old velocity content}
     ```
3. If the tags don't exist yet, or the section is empty, skip archiving — there's nothing to archive.

### Step 7 — Patch README in Memory

1. **If sentinel tags exist:** Replace everything between `<!-- RECENT-VELOCITY:START -->` and `<!-- RECENT-VELOCITY:END -->` (exclusive of the tags) with the approved summary from Step 4.
2. **If sentinel tags do NOT exist:** Append the following block to the end of the README:

   ```markdown

   ## 🛠️ Recent Velocity

   <!-- RECENT-VELOCITY:START -->
   {approved summary}
   <!-- RECENT-VELOCITY:END -->
   ```

3. Keep the full updated README content in memory — do not write any files to disk.

### Step 8 — Push via API & Create PR

Before creating anything, check if there's already an open PR from a previous run:

```bash
# 8a. Check for an existing open PR from this plugin
gh pr list --repo {username}/{username} --head "profile-update-" --state open --json number,headRefName --jq '.[0]'
```

#### If an open PR exists — update it in place

Reuse the existing branch. This avoids merge conflicts from competing PRs that touch the same block.

```bash
# Get the current file SHA from the existing branch (not main)
gh api repos/{username}/{username}/contents/README.md?ref={existing_branch} --jq '.sha'

# Update the file on the existing branch
gh api --method PUT repos/{username}/{username}/contents/README.md \
  -f message="Update recent velocity snapshot" \
  -f sha="{file_sha_from_branch}" -f branch="{existing_branch}" \
  -f content="$(base64 <<'READMEEOF'
{full updated README content}
READMEEOF
)"
```

Skip to Step 9, using the existing PR's URL.

#### If no open PR exists — create a new branch and PR

```bash
# 8b. Get the SHA of the main branch
gh api repos/{username}/{username}/git/ref/heads/main --jq '.object.sha'

# 8c. Create a new branch
gh api --method POST repos/{username}/{username}/git/refs \
  -f ref="refs/heads/profile-update-{YYYY-MM-DD}" -f sha="{main_sha}"

# 8d. Update the file on the new branch (base64-encode the full README via heredoc)
gh api --method PUT repos/{username}/{username}/contents/README.md \
  -f message="Update recent velocity snapshot" \
  -f sha="{file_sha}" -f branch="profile-update-{YYYY-MM-DD}" \
  -f content="$(base64 <<'READMEEOF'
{full updated README content}
READMEEOF
)"

# 8e. Create a pull request
gh pr create --repo {username}/{username} --base main --head "profile-update-{YYYY-MM-DD}" \
  --title "Update profile velocity snapshot" --body "Auto-generated by gh-profile-updater.

Updates the Recent Velocity section with latest activity and archives previous snapshot to Past Projects."
```

**Notes:**
- Use a single-quoted heredoc (`<<'READMEEOF'`) so that special characters in the markdown content are passed through literally.
- If branch creation fails with "Reference already exists" (422), the old branch may be from a merged/closed PR. Delete it first with `gh api --method DELETE repos/{username}/{username}/git/refs/heads/profile-update-{YYYY-MM-DD}` and retry.

### Step 9 — Report Back

Print a short confirmation to the user:

```
Profile README updated via pull request.
Categories: {list categories that had entries}
PR: {URL of the created pull request}
Archived previous velocity: yes/no
```

## Constraints

- Never fabricate commits or activity. Only summarize what is verifiable from git log output or the current session.
- If there are zero meaningful items to report, inform the user instead of writing an empty section.
- Do not remove or alter any content in the README outside of the sentinel tags, the PAST-PROJECTS section, or the appended section.
- Do not proceed past Step 4 without explicit user approval of the summary.
- Never push directly to main — always create a PR.
