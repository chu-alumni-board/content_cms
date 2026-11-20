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

## Multilingual content

`admin/config.yml` uses Sveltia's `single_file_default_root` i18n structure: everything for one announcement lives in a single file. `locales` is `[zh-TW, en-US, vi-VN]`, `default_locale` is `zh-TW`, and `initial_locales` is `[zh-TW]` — a new announcement starts Chinese-only, and editors opt in to English and/or Vietnamese per entry from the CMS UI. The default locale can't be disabled. `title` and `body` are per-locale (`i18n: true`); `publishedAt` is not — there is one shared `publishedAt` per announcement, never a per-locale copy, and no per-locale filenames.

Under this structure, the default locale's fields stay at the file root, unnested, and only non-default locales get nested under their own top-level key. The CMS also maintains a root-level `lang` list recording which locales are currently enabled for that entry; editors don't edit it directly. It's not a signal of translation completeness or human acceptance — just which locale keys exist in the file.

Chinese-only entry (no translations enabled):

```markdown
---
lang:
  - zh-TW
title: CMS 中文單語驗收 20261103
publishedAt: 2026-11-03T01:59:00Z
---

這是隔離驗收公告，不需填寫英文或越南文。![](/content/uploads/%E9%A9%97%E6%94%B6%20%E6%96%B0%E5%9C%96%E7%89%87.png)
```

Multilingual entry (English and Vietnamese enabled):

```markdown
---
lang:
  - zh-TW
  - en-US
  - vi-VN
title: CMS 圖片顯示測試
publishedAt: 2026-09-17T02:10:00Z
en-US:
  title: CMS multilingual verification
  body: English content preserved with the Chinese original.
vi-VN:
  title: Kiểm tra nội dung đa ngôn ngữ
  body: Nội dung tiếng Việt được lưu cùng thông báo tiếng Trung.
---

這是一篇用來測試 CMS 圖片在前端顯示效果的公告。

...(root Markdown body continues in zh-TW, same as a Chinese-only entry)
```

The root-level `title`/`body` (no locale key) are zh-TW's; `en-US`/`vi-VN` keys only appear once that locale has been enabled for that entry.

Announcements created before this change have no `lang` field at all — that's also a valid zh-TW-only entry under this structure, so none of them need a bulk rewrite or migration.

## Editing a multilingual announcement

1. Create or open an announcement under **公告** as usual.
2. Fill in the Chinese (zh-TW) 標題/內容 directly — no extra step needed.
3. To add a translation, use the entry editor's locale switcher to enable English and/or Vietnamese, then fill in that locale's 標題/內容.
4. Save, then reopen the entry to confirm each enabled locale kept its own title/body, and that the Chinese original and `publishedAt` are unchanged.
5. A locale that's never enabled writes no fields for it — there's no auto-translation, and an entry doesn't need every locale filled in.

Opening and saving an old Chinese-only announcement without touching English/Vietnamese leaves its existing body, `publishedAt`, and image paths untouched. Images stay under `/content/uploads/...`; a filename with non-ASCII characters or spaces (e.g. `校友會 測試圖片.png`) is URL-encoded in the generated Markdown reference so it still loads correctly.

## Content delivery to frontend

This repo's CMS produces the announcement files described above; it doesn't decide how `frontend_web` selects a locale to display, falls back when a translation is missing, or renders the result — that's `frontend_web` issue #3, not implemented by this CMS config.

As currently implemented, `frontend_web` (confirmed at its HEAD `939acfe`) bundles `content_cms`'s Markdown at build time — `build/announcements-plugin.ts` reads it into a `virtual:announcements` module, not fetched at runtime — and its own README's "Manual content update flow" is `git pull` in `content_cms`, then `npm run build` and `npm run deploy` in `frontend_web`. `documents/待討論清單.md` item A13 still records "how content reaches `frontend_web`" (runtime fetch vs. build-time bundling) as formally undecided. That's a discrepancy between the existing implementation and the authoritative spec, noted here as-is — resolving A13 or editing `documents` is out of scope for this README.

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

## Branches

There are two separate workflows on two separate branches:

- **Editor content saves** go through the CMS, which commits the content source directly to `main` (`backend.branch: main` in `admin/config.yml`). That commit by itself doesn't update the deployed public website — it only updates this repo. Getting it live still requires `frontend_web`'s manual content update flow (`git pull` here, then `npm run build` and `npm run deploy` there); there's no CI/CD connecting the two yet.
- **Code, config, and doc changes** to this repo (like `admin/config.yml` or this README) use a separate Issue branch from `develop` (for example `feat/3-announcement-locales`). Open a PR to `develop` and assign the issue back to Enos for review. After integration, a separate PR takes `develop` to `main`; Enos reviews, merges, and closes the issue. AI agents do not merge, close issues, or deploy on their own.

## Maintenance

`admin/index.html` **pins the Sveltia CMS version**. Sveltia CMS is currently beta; pinning keeps deploys reproducible. Before bumping the pinned version, check the [Sveltia CMS releases](https://github.com/sveltia/sveltia-cms/releases).

## Verification (2026-11-03)

The multilingual behavior above was checked against commit `1338c72` using the real, pinned Sveltia CMS 0.208.0 UI, pointed at an isolated local `test-repo` backend (browser-local storage only — not a live authenticated GitHub write, and not this repo or GitHub Pages). Two pre-existing announcements and images from this repo were seeded into that test backend first; the schema under test was unchanged.

Checked and passed:

- Opening an existing Chinese-only announcement, enabling English and Vietnamese, saving, and reopening preserved all three locales' title/body.
- The original Chinese body, image paths, and the shared `publishedAt` were unchanged.
- A new Chinese-only announcement saved without writing empty `en-US`/`vi-VN` fields.
- A new image with a Chinese-and-space filename uploaded with its stored bytes matching the input file (checked by SHA-256), and the generated Markdown reference was URL-encoded.

Not covered by this check: authenticated writes to the real GitHub backend, GitHub Pages deployment, and `frontend_web`'s locale selection/rendering (see "Content delivery to frontend" above).

## Follow-up verification (2026-11-20)

The same pinned Sveltia CMS 0.208.0 and multilingual schema were checked again in an isolated `test-repo`/OPFS session. An existing Chinese announcement saved and reopened with English and Vietnamese titles/bodies; its Chinese body, shared `publishedAt`, image paths, and original image bytes were preserved. A new Chinese-only announcement saved with no empty translation keys; entering 13:00 in the Taipei time field produced 05:00 UTC in the file. The desktop editor's Chinese, English, Vietnamese, and image-preview panels were visually checked.

The new-image upload/URL-encoding check remains the 2026-11-03 evidence above; it was not repeated in this follow-up. Real GitHub authentication/writes, native local-folder selection, and deployment were not exercised.

## Manual acceptance check (local, reviewer-run)

The 2026-11-03 check above ran against an automation-friendly `test-repo`/OPFS backend. The same multilingual behavior can be checked by hand using the CMS's built-in local-folder mode, which checks for Git metadata (`.git` directory or linked-worktree file) — a plain file copy without it won't do. The commit under test (`1338c72`) hasn't been pushed yet, so a fresh clone from GitHub won't include it. Use a disposable **Git clone of the local Issue branch**, running the following commands from its checkout:

1. From this repo's root, make a disposable clone of the local Issue branch: `git clone --no-hardlinks --branch feat/3-announcement-locales . ../content_cms-i18n-check` (use a new destination if it already exists).
2. Change into that disposable clone: `cd ../content_cms-i18n-check`.
3. From the disposable clone's root, run `python -m http.server 8080`.
4. Open `http://localhost:8080/admin/` in Chromium.
5. On the sign-in screen, choose **Work with Local Repository** (not Sign In with Token) and point it at the disposable clone's root directory. This mode writes only to that selected disposable clone's files on disk; it does not commit or push to GitHub.
6. Open an existing announcement that already has an image (e.g. `20260914t093500-123`). Enable English and/or Vietnamese via the locale switcher, fill in that locale's 標題/內容, and save.
7. Reopen the entry, then read the saved file directly: confirm Chinese title/body are still at the file root, the enabled locale(s) are nested under their own `en-US`/`vi-VN` key, `publishedAt` is one shared value, and the root `lang` list reflects the locales you enabled.
8. Create a new Chinese-only announcement, save it, and confirm the file has no `en-US`/`vi-VN` keys.
9. Upload an image (ideally with a Chinese character or space in the filename), and confirm it's written under `/content/uploads/` and referenced with a URL-encoded path in the Markdown.

This exercises the same save/reopen behavior as the isolated checks above, but against a disposable local clone. These native local-folder steps have not been run as part of this README edit; they remain reviewer instructions.
