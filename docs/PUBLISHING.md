# Publishing

Two hosts, two very different bars. Claude Code needs nothing but a public repo. ChatGPT/Codex needs a registered connector and a review.

This repo (`ukumi-ai/montage-plugin`) is the whole plugin: manifests, `skills/`, `agents/`, and `.mcp.json`. [`ukumi-ai/montage-skills`](https://github.com/ukumi-ai/montage-skills) stays a plain skills collection for `npx skills add ukumi-ai/montage-skills` and carries no plugin files.

A plugin manifest has to sit in the same repo as the skills it ships, so the two repos each hold their own copy of `skills/` and **nothing syncs them**. A skill change that should reach both audiences needs a commit in each. Nothing enforces that today.

## Claude Code — live as soon as the marketplace manifest is on `main`

A Claude Code marketplace is just a git repo containing `.claude-plugin/marketplace.json`:

```bash
claude plugin marketplace add ukumi-ai/montage-plugin
claude plugin install montage@montage
```

No registry, no submission, no review. Users refresh with `claude plugin marketplace update montage`.

Pin a release if you want installs to stop tracking `main`:

```bash
claude plugin marketplace add ukumi-ai/montage-plugin@v0.2.0
```

To distribute across an org on a Team or Enterprise plan, the marketplace repo must be **private or internal** and gets wired up under Organization settings → Plugins. Org sync packages relative-path plugins itself, so no separate source repo is needed.

## Codex

Same manifest, different CLI:

```bash
codex plugin marketplace add ukumi-ai/montage-plugin
codex plugin add montage@montage
```

Codex reads `.claude-plugin/marketplace.json` as a first-class source. Do **not** add a second `.agents/plugins/marketplace.json` — an empty or duplicate one makes Codex report the marketplace with zero plugins and no error.

## ChatGPT — two steps

### Step 1: local and workspace distribution (works today)

The Codex install above plus install from the ChatGPT desktop app covers local testing and workspace sharing. Workspace sharing lives in the desktop app: **Plugins → Created by you → Share**. Shared plugins stay inside the workspace and are not listed publicly.

### Step 2: the public directory

Public listing goes through the OpenAI submission portal and needs one artifact this repo cannot generate on its own: **`.app.json`**, which maps the plugin to a *registered* MCP server connection.

The ID only exists after someone registers `https://mcp.montage.app/mcp` as a connection under **Settings → Security and login → Developer mode**. The developer-mode URL shows `plugin_asdk_app_…`; the file wants `asdk_app_…`, so strip the prefix. Ship it as `.app.json.example` until then — a wrong ID fails at install time with no useful error.

### The OAuth redirect allowlist does *not* block registration

Earlier notes in this project said `https://api.ukumi.ai` must allowlist `https://chatgpt.com/connector/oauth/{callback_id}` before a ChatGPT connection can sign in. That was wrong, and it was checked:

```console
$ curl -X POST https://api.ukumi.ai/oauth/register \
    -d '{"client_name":"probe","redirect_uris":["https://chatgpt.com/connector/oauth/probe-test"],
         "token_endpoint_auth_method":"none", ...}'
201
{"client_id":"mcp-…","redirect_uris":["https://chatgpt.com/connector/oauth/probe-test"], …}
```

`api.ukumi.ai` supports open Dynamic Client Registration. The redirect URI arrives *with* the registration, so there is no static allowlist to be absent from and no chicken-and-egg. Register in developer mode and sign in; no backend change is needed first.

Two consequences worth knowing:

- **Open DCR means anyone can register a client** with any `redirect_uris`, including one they control. Normal for MCP, and the consent screen is the intended mitigation — so the consent screen must clearly show the client name and requested scopes before publishing.
- Not confirmed either way: whether developer-mode sign-in uses the same `chatgpt.com/connector/oauth/` callback path as a published connector. With open DCR it does not matter.

### Still missing for submission

Public review checks store metadata and legal links. Currently absent:

- `interface.privacyPolicyURL` — needs a real Montage privacy policy URL
- `interface.termsOfServiceURL` — needs a real terms URL
- `interface.composerIcon`, `interface.logo`, `interface.screenshots` — need assets under `./assets/`
- `.app.json` — needs the registered connector ID above

Reference: [Submit plugins](https://developers.openai.com/plugins/deploy/submission), [Plugin guidelines](https://developers.openai.com/plugins/app-guidelines), [Submission errors](https://developers.openai.com/plugins/deploy/submission-errors).

## Known backend nits

Neither affects Claude Code today; both are what a stricter host trips over.

- Protected-resource metadata advertises `"https://api.ukumi.ai/"` while the AS document says `"issuer":"https://api.ukumi.ai"` — no trailing slash. RFC 8414 §3.3 issuer validation can reject that.
- `https://api.ukumi.ai/.well-known/oauth-protected-resource` returns `Not Found`. Only the `/mcp`-suffixed path resolves. Spec-legal, since the path is advertised in the `WWW-Authenticate` header, but OpenAI's docs show clients probing the bare path.

## Release checklist

1. `version` matching across every manifest.
2. `claude plugin validate .claude-plugin/marketplace.json` clean — note that `validate .` resolves to the marketplace manifest, not the plugin manifest, so validate the plugin manifest by path too.
3. Fresh install from the marketplace, skills and agents present, MCP server connects and authenticates.
4. Merge to `main`, tag `v<version>`.
5. ChatGPT only: re-submit through the portal if store metadata or the tool surface changed.

## Gotchas that cost real time

- **`{"source":"github","repo":"owner/repo"}` in a marketplace manifest is a trap.** Claude Code clones plugin sources over SSH with no HTTPS fallback, so a public repo still fails for anyone without an SSH key: `git@github.com: Permission denied (publickey)`. Use `{"source":"url","url":"https://….git"}`. Note the asymmetry — `marketplace add owner/repo` *does* fall back to HTTPS; the plugin-source clone does not. `{"source":"git",…}` is rejected outright.
- **The marketplace `$schema` URL is `claude-code-marketplace.json`**, not `claude-code-plugin-marketplace.json`. The latter 404s.
- **`claude plugin marketplace add .` fails** with `Invalid marketplace source format`. It needs `./`.
