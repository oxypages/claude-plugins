# OxyPages for Claude Code

Publish websites and built apps to [OxyPages](https://oxypages.com) without
leaving Claude Code. Ask Claude to "put this page online", "publish this
folder to OxyPages" or "build this app and deploy it to OxyPages" and you get
a live HTTPS link. The plugin bundles the OxyPages MCP server and a
`/oxypages:publish` skill.

## Install

```text
/plugin marketplace add oxypages/claude-plugins
/plugin install oxypages@oxypages
```

When the plugin is enabled you are asked for an **OxyPages API key**. Create
one at app.oxypages.com under Profile, API Keys. It starts with `oxy_`, and
free accounts include one. Claude Code stores it as a sensitive value. Leave
it empty and the skill can still publish 30-minute unclaimed links that you
can claim into a free account; add the key later from `/plugin`.

## What you get

| | |
|---|---|
| MCP tools (with an API key) | `list_sites`, `create_site`, `deploy_site`, `read_site_files`, `update_site_files`, `list_deployments`, `deploy_app`, plus tools for project settings, custom domains, builds and Git-linked projects |
| MCP tools (no key) | `publish_page` and `deploy_app`, which publish a 30-minute link with a claim link |
| `/oxypages:publish` | the skill: when to publish (only when you ask), which tool to use, the no-account fallbacks for an HTML page or a built React, Vue or Vite app, and what to tell you afterwards |

Tools that replace or delete something live are marked destructive, and the
server asks Claude to confirm with you before it runs the ones that remove
content or change a site's address or password.

## What the plugin sends and where

- **MCP server:** `https://mcp.oxypages.com/mcp`, with your API key as a
  Bearer token when you set one.
- **No-account publishing:** the skill sends the files you choose to publish
  to `https://api.oxypages.com/drops` or `https://api.oxypages.com/drops/app`.
- **Limits:** the skill may read `https://api.oxypages.com/limits`, which is
  public and sends nothing about you.

It only sends the files you ask it to publish. It doesn't read other
credentials from your machine, and it runs no hooks or local programs. Every
publish goes through OxyPages moderation; a refusal is relayed, not retried.
Published sites are public unless you password-protect them on a paid plan.

- Privacy policy: https://oxypages.com/privacy-policy
- Terms: https://oxypages.com/terms-of-service
- Help: https://oxypages.com/help/automation/publish-from-an-ai-agent
- Support: https://oxypages.com/contact

## Develop

```bash
claude --plugin-dir ./oxypages     # load this folder without installing
claude plugin validate ./oxypages  # check the manifest and files
```
