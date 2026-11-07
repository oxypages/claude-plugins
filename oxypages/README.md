# OxyPages for Claude Code

Publish static sites to [OxyPages](https://oxypages.com) without leaving
Claude Code. The plugin bundles the OxyPages MCP server and a `/oxypages:publish`
skill.

## Install

```text
/plugin marketplace add oxypages/claude-plugins
/plugin install oxypages@oxypages
```

When the plugin is enabled you are asked for an **OxyPages API key**. Create
one at app.oxypages.com under Profile, API Keys - it starts with `oxy_` and
free accounts include one. Leave it empty and the skill can still publish
30-minute unclaimed links; add it later from `/plugin`.

## What you get

| | |
|---|---|
| MCP tools | `list_sites`, `create_site`, `deploy_site`, `list_deployments` on your account (`https://mcp.oxypages.com/mcp`) |
| `/oxypages:publish` | the skill: consent rule, which tool to use, the no-account fallbacks (an HTML page, or a built React/Vue/Vite app's output folder), what to tell the user |

Ask Claude to "publish this folder to OxyPages", "put this page online" or
"build this app and deploy it to OxyPages" and the skill takes over. Every publish goes through OxyPages moderation; a
refusal is relayed, not retried.

## Develop

```bash
claude --plugin-dir ./oxypages     # load this folder without installing
claude plugin validate ./oxypages  # the same check the community review runs
```
