---
name: majr-services
description: Use the MAJR Agent Services from Claude Code — AI SEO (audit a live page for search and AI answer engines, generate robots.txt / sitemap / head tags / llms.txt), Dispatch (turn a dev log into customer-facing release notes), and Media Encoding (video to HLS, audio to AAC, images to WebP). Use when the user asks for an SEO audit, "make this page findable / citable", robots.txt or sitemap or llms.txt, release notes or a changelog from commits, or to encode or transcode a video, audio file, or image. Also use when a majr-* MCP tool call fails with 401, 403, or 503, or when MAJR_API_KEY is missing.
---

# MAJR Agent Services

Three hosted services your MCP tools already point at. One API key works on all of them.
Free during beta, no card.

| Server (this plugin) | Does | Contract |
|---|---|---|
| `majr-seo` | Audit one live page (`audit_page`) or HTML you have not deployed yet (`audit_html` sends that HTML to the service); generate `robots.txt`, `sitemap.xml`, the `<head>` block, `llms.txt` | https://seo.majr.app/llms.txt |
| `majr-dispatch` | `draft_dispatch`: a raw dev log in, release-notes Markdown out; `get_voice`, `get_usage` | https://dispatch.majr.app/llms.txt |
| `majr-media-encoding` | `create_upload` → PUT the bytes → `encode_media` → poll `get_encoding_job` | https://encoding.majr.app/llms.txt |

## Get a key (once, one minute)

The servers read the key from the `MAJR_API_KEY` environment variable.

1. Ask the user to open https://majr.app/keys, sign in with their email (a 6-digit code),
   name their product, and copy the key. It is shown once.
2. Have them set it where Claude Code starts, then restart Claude Code:

   ```bash
   export MAJR_API_KEY=rn_...
   ```

3. Check with `/mcp` that the three `majr-*` servers are connected.

An agent can also mint the key for its owner: the loop (identity sign-in → verify the code →
`POST /v1/self-serve/keys`) is spelled out at https://majr-platform.fly.dev/llms.txt. The owner
still reads the 6-digit code from their inbox.

## What each error means

- `401` — the key is wrong. Do not guess: ask the user to check `MAJR_API_KEY`.
- `403` with `insufficient_service_scope` — the key is valid but scoped to other services. Keys
  from majr.app/keys are fleet-wide; a hand-issued key may not be. Do not rotate; use a
  fleet-wide key.
- `503` with `Retry-After` — the service could not judge the key right now (cold start or a
  control-plane blip). Wait and retry. This is never a verdict on the key.
- `429` on Dispatch — the account's draft ceiling for this window (60 an hour). Wait.

## The loops that work

**AI SEO.** `audit_page` the live URL → read `findable` (can an engine reach and index it — fix
these first) and `citable` (once reached, can it be extracted and attributed) → fix the template
(`generate_head`, `generate_robots`, `generate_sitemap`, `generate_llms_txt` render the
artifacts) → `audit_html` the render (the HTML leaves the machine) to verify before deploying → deploy → `audit_page` again.
`score` is passed/13 with equal weights; a `0.0` means unreachable, not bad.

**Dispatch.** Collect commit subjects and merged PR titles for the release, call
`draft_dispatch` with `changes` (and `render_target: "standalone_document"` when the notes are
the whole page). `no_news: true` with empty markdown means nothing user-facing shipped — say so,
do not invent notes. The account's brand-voice floor is composed server-side; send persona only.

**Media Encoding.** `create_upload` → PUT the file to the returned URL with the right
`Content-Type` (bytes never transit MCP) → `encode_media` with a fresh UUID `jobId` and the
`source` from the upload; omit `destination` and `callback` → poll `get_encoding_job` until
`SUCCEEDED` → download every `outputs[]` URL and copy it into the user's own storage before
`objectExpiresAt` (~72h). HLS playlists reference segments by relative path: re-host the whole
prefix, never hand out the presigned master URL.

## Quickstarts and reference

- https://majr.app/quickstart/seo · https://majr.app/quickstart/dispatch ·
  https://majr.app/quickstart/media-encoding
- Fleet catalog (machine-readable): https://dispatch.majr.app/catalog.json
- Usage per product: https://majr.app/keys (or `get_usage` on Dispatch for the whole fleet)
