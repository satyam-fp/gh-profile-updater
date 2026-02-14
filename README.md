# gh-profile-updater

A Claude Code plugin that automatically generates impact-driven summaries from your git activity and updates your GitHub profile README — so your profile always reflects what you're *actually* shipping.

---

## What It Does

`gh-profile-updater` scans your recent git commits and current session context, classifies your progress, and writes a polished summary directly into your GitHub profile README.

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

That's it. The plugin will scan your recent work and update your profile README automatically.

---

## How It Works

1. **Reads** the last 5 git commits and the current session's conversation context.
2. **Classifies** each item as Shipping, Learning, or Refining.
3. **Generates** a high-impact summary (bulleted list or 3-sentence narrative).
4. **Locates** your GitHub profile repo — checks for a local clone first, otherwise **auto-clones** your `<username>/<username>` repo to a temp directory.
5. **Patches** the profile `README.md` between `<!-- RECENT-VELOCITY:START -->` and `<!-- RECENT-VELOCITY:END -->` sentinel tags — or appends a new section if the tags don't exist yet.
6. **Pushes** the update directly to GitHub (when using the auto-clone path).

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

After running `/update-profile`, the plugin appends a live-updated section:

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
```

The section updates in-place every time you run the skill — your profile stays fresh without any manual editing.

---

## Sentinel Tags

To control exactly where the summary appears in your README, add these HTML comments:

```html
<!-- RECENT-VELOCITY:START -->
<!-- RECENT-VELOCITY:END -->
```

The plugin will replace the content between them. If the tags aren't present, it appends a new `## 🛠️ Recent Velocity` section at the end of the file.

---

## Configuration

No configuration needed. The plugin infers your GitHub username from the current repo's remote URL and automatically handles the profile repo:

- **Local clone exists?** Updates the file in place (no auto-commit).
- **No local clone?** Clones the profile repo to a temp directory, patches, commits, pushes, and cleans up.

---

## Requirements

- Claude Code CLI
- A GitHub profile repo (`<your-username>/<your-username>`) with a `README.md`

---

## License

MIT
