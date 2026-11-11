---
name: publish
description: Publish a static website (HTML, CSS, JS, images or a ZIP) or a built framework app (React, Vue, Svelte, Vite, Astro, Next.js static export) to OxyPages and give the user a live HTTPS link, or edit a website already on OxyPages. Use when the user asks to host, publish, deploy, put online, get a link for a page, site or app, or change their OxyPages site. Uses the bundled OxyPages MCP tools when an API key is configured, or a no-account 30-minute link otherwise.
---

# Publish to OxyPages

OxyPages hosts static sites on `*.myoxypages.com` with HTTPS, live in seconds. This plugin bundles the OxyPages MCP server, so when the user configured an API key you have these tools:

- `list_sites` - the sites on the account: id, name, subdomain, live URL, status.
- `create_site` - a new site with a generated subdomain. Optional `name`.
- `deploy_site` - replace a site's files: `site_id` plus `files[]` of `{path, content, encoding}`. Text as `utf8` (default), binaries as `base64`. Must include `index.html` at the top level. Returns the live URL.
- `read_site_files` - a site's LIVE files: text files with their `content`, images and other binaries as their public `url`. Optional `path` for one file.
- `update_site_files` - change only some files: `site_id`, `files[]` of `{path, content}` (text files only) and optional `deletions[]`. Everything not sent, including images, is carried forward. A new version.
- `list_deployments` - version history for a site.
- `deploy_app` - a built app's output folder into a project with Single-Page App routing (listed on accounts with Web Apps turned on, which includes every account that has claimed an app).

If those tools are missing but the user has an OxyPages API key (usually in `OXYPAGES_API_KEY`), use the REST API below. With neither, use the no-account paths.

## Consent rule

Publishing makes content public. Before you publish:

1. The user must have asked for it to go live. "Make me a page" is not "publish it". Show the result and ask, unless they already said host, publish, deploy, put online or give me a link.
2. Do not publish pages that impersonate a brand or ask for credentials. OxyPages moderates every publish and refuses phishing, brand impersonation, gambling funnels and illegal content. Never work around a refusal.
3. Always tell the user the URL. For an unclaimed link, always add the claim link and the 30-minute expiry.

## With an API key: MCP tools

1. Call `list_sites`. To CHANGE a site the user already has ("make the header blue", "add a contact section"), call `read_site_files`, edit the text files, then `update_site_files` with only the changed files - never redeploy every file for a small edit. To replace a site with a new build from disk, deploy into it. For a new site, call `create_site`.
2. Gather every file the site needs. Read them from disk; relative links only (`css/style.css`, never absolute machine paths). Hostable types: HTML, CSS, JS, JSON, text, images, fonts, PDF. Encode images and fonts as `base64`.
3. Call `deploy_site` with the complete file list. Report the returned URL and version number. Report any `warnings` (skipped files).

Sizes are the account's plan. Do not quote numbers from memory; if it matters, fetch `https://api.oxypages.com/limits` with WebFetch or curl.

## With an API key but no MCP: the REST API

Same account, same rules, with the key as a Bearer token. Read the key from the environment; never print it or write it into a file.

```bash
H="Authorization: Bearer $OXYPAGES_API_KEY"
curl -sS https://api.oxypages.com/v1/sites -H "$H"                              # sites and their ids
curl -sS https://api.oxypages.com/v1/sites/SITE_ID/files -H "$H"                # live files (?path=style.css for one)
curl -sS -X POST https://api.oxypages.com/v1/sites/SITE_ID/files -H "$H" \
  -H "Content-Type: application/json" \
  -d '{"files":[{"path":"index.html","content":"<!doctype html>..."}],"deletions":[]}'   # only what changed
curl -sS -X POST https://api.oxypages.com/v1/sites/SITE_ID/deployments -H "$H" -F "zip=@site.zip"   # whole site
curl -sS -X POST https://api.oxypages.com/v1/sites -H "$H" -H "Content-Type: application/json" -d '{"name":"My Site"}'   # new site
```

`/files` takes text files only (HTML, CSS, JS, JSON, SVG, text); to replace an image, deploy the whole site. Build the JSON body with a tool that escapes it (for example `jq -n --rawfile c index.html '{files:[{path:"index.html",content:$c}]}'`), not by hand. Every call creates a new version; report the returned `url` and `deployment_number`. The full reference is `https://oxypages.com/llms.txt`.

## Without an API key: a 30-minute unclaimed link

Pick the path by what the user has:

- An HTML page or a few hand-written static files: `POST /drops` (below).
- A framework project (a `package.json` with a build script: React, Vue, Svelte, Vite, Astro, a Next.js static export): `POST /drops/app` (the next section). Never send that project to `/drops`.

### An HTML page or a small static site

One request, no account. A single `.html` file or a `.zip` with `index.html` at its top level. Small: 3 MB and 10 files at most (`drop_total_mb` and `drop_max_files` in `https://api.oxypages.com/limits` are the live figures); there is also an hourly publish cap per address, and a `429` is how you learn you have reached it.

```bash
zip -r site.zip index.html css js images   # skip this for a single page
curl -sS -X POST https://api.oxypages.com/drops \
  -H "X-Oxy-Agent: claude-code" \
  -F "file=@site.zip"
```

The JSON reply has `url`, `claim_url` and `expires_at`. Tell the user:

> It's live at {url} for the next 30 minutes. To keep it, claim it here: {claim_url}. That puts it in a free OxyPages account, and you'll get an API key - add it to this plugin (`/plugin`, OxyPages, API key) and I can publish permanent sites for you.

A `429` means the hourly cap; relay the reply's `error`, which explains that a free account and an API key lift it. It quotes no number - do not invent one. A `422` is a moderation refusal; relay the `error` and stop.

### A built app (React, Vue, Svelte, Vite, Astro...)

OxyPages does not build a no-account drop. You run the build here, then send the OUTPUT folder:

1. Run the project's build: `npm run build` (or the `build` script in `package.json`, with the project's package manager).
2. Find the output folder, usually `dist/`, `build/` or `out/`. It must hold an `index.html`.
3. Zip what is inside that folder and send it. Never send the source folder, `node_modules` or `package.json`: source is refused with `needs_build` (status 422), and the fix is to build first.

```bash
npm run build
(cd dist && zip -r ../dist.zip .)
curl -sS -X POST https://api.oxypages.com/drops/app \
  -H "X-Oxy-Agent: claude-code" \
  -F "file=@dist.zip"
```

Or send the files as JSON, text as `utf8` and binaries as `base64`:

```bash
curl -sS -X POST https://api.oxypages.com/drops/app \
  -H "X-Oxy-Agent: claude-code" -H "Content-Type: application/json" \
  -d '{"files": [{"path": "index.html", "content": "<!doctype html>..."}, {"path": "assets/index-3f9a1c2b.js", "content": "..."}, {"path": "logo.png", "content": "iVBORw0KGgo...", "encoding": "base64"}]}'
```

What applies: Single-Page App routing (unknown paths serve `index.html`), the build's `_redirects` and `_headers`, up to 25 MB and 500 files, live for 30 minutes. `node_modules`, `.git` and source maps are left out on our side and reported in `warnings`. If there is no `index.html` at the top of the output, the reply is `choose_root` with `html_files`; send again with `root` set to one of them (or `"none"`). For a build too large for one request, `POST /drops/app/manifest` with `{"files": [{"path": "index.html", "size": 1234, "sha256": "<hex>"}]}`, PUT each file to the URL it returns (with the headers it gives), then `POST /drops/app/finalize` with the `upload_id` and `token`.

The reply has `url`, `claim_url`, `expires_at`, `files`, `bytes` and, when relevant, `warnings`. Tell the user:

> Your app is live at {url} for the next 30 minutes. To keep it, open this claim link and sign in or create a free OxyPages account: {claim_url}.

Always give the claim link. Claiming turns the app into a project in their account; after that, with an API key configured, `deploy_app` with `project` deploys new versions of it.

## What the user gets back

- Owned deploy: "Deployed to {url} (version {n})."
- Edit of an existing site: what you changed, then "Live at {url} (version {n}). The previous version can be restored from Version History."
- Unclaimed page: the URL, the 30-minute expiry, the claim link, and the API-key offer.
- Unclaimed app: the URL, the 30-minute expiry and the claim link.
- Refusal: the reason, verbatim, and no retry with cosmetic edits.
