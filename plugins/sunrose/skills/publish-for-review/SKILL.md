---
name: publish-for-review
description: Publish a built static website to Sunrose as a preview or a client review, and make a review link to send. Use when the user wants to share a site, send it to a client, get feedback on it, or put a new version up.
---

# Publish a site for client review

Sunrose hosts a **built static site** (HTML, CSS, JavaScript and assets, with `index.html` at the root). It does not build projects or run server code, and the MCP tools cannot publish designs (images or PDFs); those are dropped on the Sunrose website.

## 1. Get the site ready

- Publish the **build output**, not the source. For Vite or Astro that is usually `dist`; build it first (`npm run build`) and check the folder has `index.html` at its top level.
- Never include `node_modules`, `.env` files, source maps you did not mean to ship, or anything holding secrets. Executables are refused.
- Paths are relative to the site root (`index.html`, `css/site.css`). Every version needs the **full** set of files, including unchanged ones: a version is a snapshot, not a patch.

## 2. Decide new project or new version

- Call `list_projects` (add `includeArchived` if you expect an archived one). To add a version, use that project's `projectId`. For a new project, pass a `name` (up to 80 characters) and no `projectId`.
- An archived project must be restored first with `unarchive_project`, which fails if the plan's project limit is full.
- For a large site, `get_usage` shows the plan, storage and files left this month.

## 3. Publish

Choose `mode`: `review` when a client will comment, `show` for a plain preview (default is `review`).

**Up to 5 MB in total** (decoded): call `publish_site` with `files`, each `{ path, text }` for text files or `{ path, base64 }` for binary ones.

**Larger than 5 MB, or more files than fits comfortably:**
1. Work out each file's exact size in bytes (for example `wc -c < file`).
2. Call `begin_publish` with `files: [{ path, size }]` (up to 200 files).
3. For every file, `PUT` its raw bytes to the returned URL with **exactly** the headers listed, from a shell (for example `curl -T file -H "…" "URL"`). Uploads expire after a few minutes.
4. Call `finish_publish` with the `sessionId`. **Nothing is live until this succeeds.** If it reports files that are not ready, upload those again and call it again.

Over 200 files needs the REST API instead (see `https://sunrose.dev/docs/api`).

## 4. Read the result

- `held: true` means the upload is saved but waiting on a safety check. It is **not** being served, so do not give out an address for it.
- Otherwise give the user the preview address.

## 5. Share it for feedback

Call `create_review_link` with `audience: "client"`:
- `approvalEnabled` (default true) lets the reviewer approve the version.
- `expiresInDays` (1–90, default 30) is how long the link stays open **without a visit**; each visit renews it.
- `feedbackDueAt` (an ISO date) closes comments and approval on a date, on Studio and Agency only.
- `allowDownload` is for image and PDF versions only; leave it off for websites.

Treat the returned URL as a secret: anyone who holds it can open that version and comment, and it **bypasses the preview password**. Give it only to the user, never paste it into a public place. A review link is tied to the version it was made for, so after publishing a new version, make a new link.

If the user wants a teammate to look instead of a client, use `audience: "team"`: it works once and must be opened within 15 minutes.
