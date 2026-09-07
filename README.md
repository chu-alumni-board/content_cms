# content_cms

Content management repo for Chu Alumni Association's **public content**. Built on [Sveltia CMS](https://github.com/sveltia/sveltia-cms) (a git-based CMS): content lives as files in this repo, with no CMS backend database.

The admin UI has two modes, with different write behavior:

- **GitHub backend** (online use): saves are written back to the GitHub repository as a commit
- **Local Repository mode** (local use): only modifies local files, does **not** auto-commit or push

## Boundary

This repo is **fully separated** from the alumni system (`backend_cloud` / the alumni information system): no connection to the alumni database, no shared account mechanism, no path to any personal data. The repository access (PAT or OAuth scope) used by the CMS should be limited to `content_cms`.

## Directory structure

```
admin/
  index.html      # Admin UI (static page, loads Sveltia CMS)
  config.yml      # CMS config: backend and collections
content/
  announcements/  # Announcements, one .md per entry
```

Announcement files are Markdown with YAML frontmatter:

```markdown
---
title: 標題
publishedAt: 2026-09-08T05:30:00Z
---

內文（Markdown）
```

`publishedAt` stores a full timestamp (ISO 8601, UTC), not just a date — same-day announcements are possible, and a date alone can't determine their order. Editors enter Taiwan local time in the admin UI; the CMS always converts it to UTC on save.

The filename is `<publishedAt's UTC timestamp, YYYYMMDDTHHmmss>-<title>.md`. This prefix **only exists to avoid same-title collisions — it carries no display-date meaning** and no sort meaning either. Sorting always reads the `publishedAt` frontmatter field, never the filename.

## Editors: how to publish an announcement

Admin UI URL:

```
https://chu-alumni-board.github.io/content_cms/admin/
```

Login is currently by **PAT (Personal Access Token)**:

1. Open the admin UI, click **Sign In with Token**
2. The on-screen link takes you to GitHub's fine-grained token creation page, with **Contents: write** pre-selected
3. **Under Repository access, select only `chu-alumni-board/content_cms`** — not All repositories
4. Set an expiry date, generate the token, and paste it back into the CMS's input field
5. You can then add/edit announcements; saving writes back to the GitHub repository (one commit)

> **A PAT is itself a sensitive credential** — never write it into any file, message, or repo, and set an expiry date when you generate one.

Editors need **write** access to this repo; a PAT cannot grant more than the holder already has.

## Local editing and development

Sveltia CMS supports operating directly against a local clone, no proxy server needed:

```bash
python -m http.server 8080          # run from the repo root
```

Open <http://127.0.0.1:8080/admin/index.html> in a **Chromium-based browser** (Chrome / Edge / Brave), choose **Work with Local Repository**, and point it at this clone. This mode uses the File System Access API, so it's limited to Chromium-based browsers, and the CMS does no git operations for you — commit and push are on you.

## Deployment

The admin UI is just two static files, `admin/index.html` and `admin/config.yml`, with no secrets in either.

The Sveltia GitHub backend points at `develop` (see `admin/config.yml`).

## Maintenance

`admin/index.html` **pins the Sveltia CMS version** (currently `0.208.0`). Sveltia CMS is currently beta; pinning keeps deploys reproducible and prevents upstream changes from directly affecting the live admin UI.

**The cost is this isn't a long-term zero-maintenance solution.** Someone needs to periodically (suggested: quarterly, or on security advisories) check <https://github.com/sveltia/sveltia-cms/releases>, evaluate, and manually bump the version in `index.html`. Not updating means continuing to miss bug and security fixes.
