# montage-plugin

The plugin marketplace for **Montage** — install the Montage skills and MCP
server into Claude Code or Codex with two commands.

This repo is the *front door*. It holds one file: a marketplace manifest that
points at the plugin itself, which lives in
[ukumi-ai/montage-skills](https://github.com/ukumi-ai/montage-skills).

## Install

**Claude Code**

```bash
claude plugin marketplace add ukumi-ai/montage-plugin
claude plugin install montage@montage
```

Or from inside a session: `/plugin marketplace add ukumi-ai/montage-plugin`
then `/plugin install montage@montage`.

**Codex / ChatGPT desktop**

```bash
codex plugin marketplace add ukumi-ai/montage-plugin
codex plugin add montage@montage
```

Both hosts read the same `.claude-plugin/marketplace.json`. Codex supports it
as a first-class source; there is no separate Codex marketplace file.

## What you get

| Component | Name |
|---|---|
| Skill | `ingest-video` — pull a video in from Google Drive or a public URL and follow the pipeline |
| Skill | `create-moment` — turn a segment into an editable moment with an editor link |
| Agent | `clip-scout` — scouts a corpus for clip-worthy segments |
| MCP server | `montage` → `https://mcp.montage.app/mcp` |

No credentials to configure. The MCP server speaks OAuth 2.1 with dynamic
client registration, so the host prompts you to sign in on the first tool call.

## Two repos, one plugin

| Repo | Holds |
|---|---|
| **montage-plugin** (this one) | the marketplace manifest only |
| [montage-skills](https://github.com/ukumi-ai/montage-skills) | the plugin — `skills/`, `agents/`, `.mcp.json`, both plugin manifests |

The split keeps `npx skills add ukumi-ai/montage-skills` working for people who
want the raw skills without a plugin host, and keeps exactly one copy of each
skill.

**To add or change a skill, an agent, or the MCP config, work in
montage-skills** — see its `CONTRIBUTING.md`. Nothing in this repo needs to
change when the plugin gains a skill; the marketplace tracks the default branch
of montage-skills, so a merge there is live here immediately.

You only touch this repo to rename the marketplace, add a second plugin
alongside `montage`, or pin to a specific commit (add `"sha"` next to `"url"`
in the source block).

## Publishing to the ChatGPT app directory

Not done yet, and it does not happen from this repo. The remaining steps live
on the montage-skills side: register the MCP server in ChatGPT developer mode,
paste the resulting `asdk_app_…` ID into `.app.json`, and allowlist the ChatGPT
OAuth redirect on `api.ukumi.ai`. See `montage-skills/CONTRIBUTING.md`.
