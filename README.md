# content_cms

Content management repo for Chu Alumni Association's **public content**. Built on [Sveltia CMS](https://github.com/sveltia/sveltia-cms) (a git-based CMS): content lives as files in this repo, with no CMS backend database.

## Boundary

This repo contains **public content only**. It must never contain personal data or secrets.

## Repository structure

```
admin/
  index.html      # Admin UI (static page, loads Sveltia CMS)
  config.yml      # CMS config: backend and collections
content/
  announcements/  # Announcements, one .md per entry
```

## Content format

Announcement files are Markdown with YAML frontmatter:

```markdown
---
title: 標題
publishedAt: 2026-09-08T05:30:00Z
---

內文（Markdown）
```

`publishedAt` is a full timestamp (ISO 8601, UTC). Editors enter Taiwan local time in the admin UI; the CMS converts it to UTC on save.

## Admin UI

[https://chu-alumni-board.github.io/content_cms/admin/](https://chu-alumni-board.github.io/content_cms/admin/)

Login is currently by **PAT (Personal Access Token)**:

1. Open the admin UI, click **Sign In with Token**
2. The on-screen link takes you to GitHub's fine-grained token creation page, with **Contents: write** pre-selected
3. **Under Repository access, select only `chu-alumni-board/content_cms`** — not All repositories
4. Set an expiry date, generate the token, and paste it back into the CMS's input field
5. Saving writes an announcement back to this repository as a commit

> **A PAT is itself a sensitive credential** — never write it into any file, message, or repo, and always set an expiry date when generating one.

Editors need **write** access to this repo; a PAT cannot grant more than the holder already has.

## Maintenance

`admin/index.html` **pins the Sveltia CMS version**. Sveltia CMS is currently beta; pinning keeps deploys reproducible. Before bumping the pinned version, check the [Sveltia CMS releases](https://github.com/sveltia/sveltia-cms/releases).
