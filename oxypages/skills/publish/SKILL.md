---
name: publish
description: Publish a static website (HTML, CSS, JS, images or a ZIP) to OxyPages and give the user a live HTTPS link. Use when the user asks to host, publish, deploy, put online, or get a link for a page or site. Uses the bundled OxyPages MCP tools when an API key is configured, or a no-account 30-minute link otherwise.
---

# Publish to OxyPages

OxyPages hosts static sites on `*.myoxypages.com` with HTTPS, live in seconds. This plugin bundles the OxyPages MCP server, so when the user configured an API key you have these tools:

- `list_sites` - the sites on the account: id, name, subdomain, live URL, status.
- `create_site` - a new site with a generated subdomain. Optional `name`.
- `deploy_site` - replace a site's files: `site_id` plus `files[]` of `{path, content, encoding}`. Text as `utf8` (default), binaries as `base64`. Must include `index.html` at the top level. Returns the live URL.
- `list_deployments` - version history for a site.

If those tools are missing, the key is not configured; use the no-account path below.

## Consent rule

Publishing makes content public. Before you publish:

1. The user must have asked for it to go live. "Make me a page" is not "publish it". Show the result and ask, unless they already said host, publish, deploy, put online or give me a link.
2. Do not publish pages that impersonate a brand or ask for credentials. OxyPages moderates every publish and refuses phishing, brand impersonation, gambling funnels and illegal content. Never work around a refusal.
3. Always tell the user the URL. For an unclaimed link, always add the claim link and the 30-minute expiry.

## With an API key: MCP tools

1. Call `list_sites`. If a site clearly matches what the user is updating, deploy into it - a deploy replaces the whole site, which is what "update my site" means. Otherwise, or when they say "new site", call `create_site`.
2. Gather every file the site needs. Read them from disk; relative links only (`css/style.css`, never absolute machine paths). Hostable types: HTML, CSS, JS, JSON, text, images, fonts, PDF. Encode images and fonts as `base64`.
3. Call `deploy_site` with the complete file list. Report the returned URL and version number. Report any `warnings` (skipped files).

Sizes are the account's plan. Do not quote numbers from memory; if it matters, fetch `https://api.oxypages.com/limits` with WebFetch or curl.

## Without an API key: a 30-minute unclaimed link

One request, no account. A single `.html` file or a `.zip` with `index.html` at its top level. 5 MB and 10 files at most; there is also an hourly publish cap per address, and a `429` is how you learn you have reached it.

```bash
zip -r site.zip index.html css js images   # skip this for a single page
curl -sS -X POST https://api.oxypages.com/drops \
  -H "X-Oxy-Agent: claude-code" \
  -F "file=@site.zip"
```

The JSON reply has `url`, `claim_url` and `expires_at`. Tell the user:

> It's live at {url} for the next 30 minutes. To keep it, claim it here: {claim_url}. That puts it in a free OxyPages account, and you'll get an API key - add it to this plugin (`/plugin`, OxyPages, API key) and I can publish permanent sites for you.

A `429` means the hourly cap; relay the reply's `error`, which explains that a free account and an API key lift it. It quotes no number - do not invent one. A `422` is a moderation refusal; relay the `error` and stop.

## What the user gets back

- Owned deploy: "Deployed to {url} (version {n})."
- Unclaimed: the URL, the 30-minute expiry, the claim link, and the API-key offer.
- Refusal: the reason, verbatim, and no retry with cosmetic edits.
