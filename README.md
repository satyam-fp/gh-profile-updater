# gh-profile-updater

A Claude Code plugin that automatically generates impact-driven summaries from your git activity and updates your GitHub profile README — so your profile always reflects what you're *actually* shipping.

---

## What It Does

`gh-profile-updater` scans your recent git commits and current session context, classifies your progress, and writes a polished summary into your GitHub profile README — via pull request, so you always get a chance to review before it goes live.

Every item is categorized into one of three buckets:

| Category | Description |
|---|---|
| **Shipping** | New features, completed projects, deployed services |
| **Learning** | New libraries, languages, or research domains explored |
| **Refining** | Optimizations, bug fixes, architectural improvements |

The output focuses on **impact**, not activity — *"Cut API latency by 40%"* instead of *"Changed some database queries."*

---

## Installation

### 1. Add the marketplace

```
/plugin marketplace add satyam-fp/gh-profile-updater
```

### 2. Install the plugin

```
/plugin install gh-profile-updater@satyam-tools
```

### 3. Run the skill

```
/update-profile
```

That's it. The plugin will scan your recent work, present a draft for your review, and open a PR to update your profile README.

---

## How It Works

1. **Scans** the last 20 git commits and the current session's conversation context.
2. **Classifies** each item as Shipping, Learning, or Refining.
3. **Generates** a high-impact summary (bulleted list or 3-sentence narrative).
4. **Presents the draft** for your review — you can request edits before anything is pushed.
5. **Reads** your profile `README.md` via the GitHub API (no local clone needed).
6. **Archives** the previous velocity snapshot to a "Past Projects" section with a date header.
7. **Patches** the README between `<!-- RECENT-VELOCITY:START -->` / `<!-- RECENT-VELOCITY:END -->` sentinel tags — or appends a new section if the tags don't exist yet.
8. **Opens a pull request** with the updated README on a new branch.

---

## Before & After

### Before

A typical GitHub profile README with no recent activity context:

```markdown
# Hi, I'm Satyam

Software engineer. I like building things.

## Tech Stack
- TypeScript, Python, Go
- React, Node.js, PostgreSQL
```

### After

After running `/update-profile` and merging the PR:

```markdown
# Hi, I'm Satyam

Software engineer. I like building things.

## Tech Stack
- TypeScript, Python, Go
- React, Node.js, PostgreSQL

## 🛠️ Recent Velocity

<!-- RECENT-VELOCITY:START -->
- 🚀 Shipped a Claude Code plugin that auto-generates profile summaries from git history.
- 📚 Explored Claude's plugin/skill architecture and marketplace registration.
- 🔧 Refined commit-scanning logic to extract impact-first descriptions from raw diffs.
<!-- RECENT-VELOCITY:END -->

## 📦 Past Projects

<!-- PAST-PROJECTS:START -->
### Jan 2026
- 🚀 Built an initial prototype of the profile updater with local git clone workflow.
- 📚 Learned Claude Code's plugin system and skill registration.
<!-- PAST-PROJECTS:END -->
```

The Recent Velocity section updates each time you run the skill, and previous snapshots are automatically archived under Past Projects with a date header.

---

## Sentinel Tags

To control exactly where the summary appears in your README, add these HTML comments:

```html
<!-- RECENT-VELOCITY:START -->
<!-- RECENT-VELOCITY:END -->
```

The plugin will replace the content between them. If the tags aren't present, it appends a new `## 🛠️ Recent Velocity` section at the end of the file.

For archival, the plugin uses a second pair of tags:

```html
<!-- PAST-PROJECTS:START -->
<!-- PAST-PROJECTS:END -->
```

If these don't exist, they're created automatically when the first snapshot is archived.

---

## Configuration

No configuration needed. The plugin infers your GitHub username from the current repo's remote URL and interacts with your profile repo entirely through the GitHub API.

- **Review gate**: You always see the draft summary before any changes are pushed.
- **Pull request workflow**: Changes are pushed to a new branch and a PR is opened — never pushed directly to main.
- **Archival**: Previous velocity snapshots are preserved in a Past Projects section with month/year headers.

---

## Requirements

- Claude Code CLI
- [GitHub CLI (`gh`)](https://cli.github.com/) — install via `brew install gh` (macOS), `winget install GitHub.cli` (Windows), or [see all options](https://github.com/cli/cli#installation). Then authenticate with `gh auth login`.
- A GitHub profile repo (`<your-username>/<your-username>`) with a `README.md`

---

## License

MIT
