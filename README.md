# montage-plugin

The **Montage** plugin for Claude Code and ChatGPT / Codex. Import recordings
into Montage, create editable moments, or use transcripts and visual analysis
for evidence-backed writing and research in the assistant.

## Install

**Claude Code**

```bash
claude plugin marketplace add ukumi-ai/montage-plugin
claude plugin install montage@montage
```

Or in a session: `/plugin marketplace add ukumi-ai/montage-plugin` then
`/plugin install montage@montage`.

**Codex / ChatGPT desktop**

```bash
codex plugin marketplace add ukumi-ai/montage-plugin
codex plugin add montage@montage
```

Both hosts read the same `.claude-plugin/marketplace.json`.

## What you get

| Component | Name |
|---|---|
| Skill | `ingest-video` — pull a video in from a Google Drive or Dropbox link and follow the pipeline |
| Skill | `create-moment` — turn a segment into an editable moment with an editor link |
| Skill | `repurpose-recordings` — write summaries, posts, and case studies or research across recordings with timestamped evidence |
| Agent | `clip-scout` — read-only scout that proposes clip segments |
| MCP server | `montage` → `https://mcp.montage.app/mcp` |

No credentials to configure. The MCP server speaks OAuth 2.1 with dynamic client
registration, so the host prompts you to sign in on the first tool call.

## Relationship to montage-skills

| Repo | Holds | For |
|---|---|---|
| **montage-plugin** (this one) | the plugin: manifests, `skills/`, `agents/`, `.mcp.json` | Claude Code and Codex plugin installs |
| [montage-skills](https://github.com/ukumi-ai/montage-skills) | the plain skills collection | `npx skills add ukumi-ai/montage-skills` |

Both repositories carry byte-for-byte copies of the canonical Montage MCP
skills. Updates require a commit in each repository. Plugin manifests, agents,
and MCP config belong here.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for adding a skill or an agent, the MCP
auth model, local testing, and releases.

## Publishing to the ChatGPT app directory

Not done yet. It needs the MCP server registered in ChatGPT developer mode and
the resulting `asdk_app_…` ID pasted into `.app.json` — see the "ChatGPT wiring"
section of CONTRIBUTING.md.
