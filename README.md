# Sunrose plugin

Connects Claude Code and Codex to [Sunrose](https://sunrose.dev): publish a built static site for client review, send a review link, and work through the feedback.

It bundles the Sunrose MCP server (`https://sunrose.dev/mcp`) and two skills:

- `publish-for-review`: publish a site as a new project or version, and make a review link.
- `triage-feedback`: read reviewer comments, turn them into changes, and track each one's status.

API and MCP access need a Sunrose plan that includes them (Personal and above). The tools, and what a connection can and cannot do, are listed at <https://sunrose.dev/docs/mcp>.

## Claude Code

```
/plugin marketplace add coshge/sunrose-plugin
/plugin install sunrose@sunrose
```

Then run `/mcp` (or `claude mcp login sunrose`) and sign in. You choose Read only or Read and write on Sunrose's consent screen. The skills are `/sunrose:publish-for-review` and `/sunrose:triage-feedback`.

To try it from a checkout without installing: `claude --plugin-dir ./plugins/sunrose`, then `/reload-plugins` after edits.

## Codex

```
codex plugin marketplace add coshge/sunrose-plugin
codex            # then /plugins, choose Sunrose, and install it
codex mcp login sunrose
```

To add only the server, without the plugin: `codex mcp add sunrose --url https://sunrose.dev/mcp`, then `codex mcp login sunrose`.

## Layout

One folder serves both tools, so the skills and the MCP file are written once.

```
plugins/sunrose/
  .claude-plugin/plugin.json   Claude Code manifest
  .codex-plugin/plugin.json    Codex manifest
  .mcp.json                    the remote MCP server, shared
  skills/                      shared skills (SKILL.md)
  assets/logo.png              the Codex plugin icon
.claude-plugin/marketplace.json     Claude Code marketplace
.agents/plugins/marketplace.json    Codex marketplace
```

Check it with `claude plugin validate ./plugins/sunrose` and `claude plugin validate .` (the marketplace).

## Licence

[MIT](LICENSE).
