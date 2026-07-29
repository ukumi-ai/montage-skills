# Publishing

Two hosts, two very different bars. Claude Code needs nothing but a public repo. ChatGPT/Codex needs a registered connector and a review.

## Claude Code — already live

A Claude Code marketplace is just a git repo containing `.claude-plugin/marketplace.json`. Once the packaging is on `main`:

```bash
claude plugin marketplace add ukumi-ai/montage-skills
claude plugin install montage@montage
```

There is no registry, no submission, no review. Users refresh with `claude plugin marketplace update montage`.

Pin a release if you want installs to stop tracking `main`:

```bash
claude plugin marketplace add ukumi-ai/montage-skills@v0.2.0
```

Nothing else to do here. To distribute across an org on a Team or Enterprise plan, the marketplace repo must be **private or internal** and gets wired up under Organization settings → Plugins; org sync packages relative-path plugins itself, so no separate source repo is needed.

## ChatGPT and Codex — two steps

### Step 1: local and workspace distribution (works today)

`codex plugin marketplace add ukumi-ai/montage-skills` plus install from the ChatGPT desktop app covers local testing and workspace sharing. Workspace sharing lives in the desktop app: **Plugins → Created by you → Share**. Shared plugins stay inside the workspace and are not listed publicly.

### Step 2: the public directory

Public listing goes through the OpenAI submission portal, and it needs one artifact this repo cannot generate on its own: **`.app.json`**, which maps the plugin to a *registered* MCP server connection.

See [`.app.json.example`](../.app.json.example). It is intentionally not wired up — a wrong ID fails at install time with no useful error. `CONTRIBUTING.md` has the paste-in recipe; the ID only exists after someone registers `https://mcp.montage.app/mcp` as a connection under **Settings → Security and login → Developer mode**. The developer-mode URL shows `plugin_asdk_app_…`; the file wants `asdk_app_…`, so strip the prefix.

### The OAuth redirect allowlist blocks registration, not just review

Montage's authorization server (`https://api.ukumi.ai`) must allowlist `https://chatgpt.com/connector/oauth/{callback_id}` before a ChatGPT connection can sign in. As of this writing it does not.

This is chicken-and-egg: `{callback_id}` is per-connection and only appears once the connection exists. Expected sequence:

1. Register the connection in developer mode.
2. Start sign-in. If the redirect URI is not allowlisted, the AS rejects it — read the `redirect_uri` out of the failing authorize request or the error page.
3. Allowlist that exact URI on `api.ukumi.ai`.
4. Retry sign-in.

Unverified: whether developer-mode sign-in uses the same `chatgpt.com/connector/oauth/` callback path as a published connector. The docs describe one callback shape and do not carve out developer mode, so plan for step 2 failing rather than being surprised by it. Nothing about this affects Claude Code, which registers itself dynamically via DCR.

### Still missing for submission

Public review checks store metadata and legal links. Currently absent:

- `interface.privacyPolicyURL` — needs a real Montage privacy policy URL
- `interface.termsOfServiceURL` — needs a real terms URL
- `interface.composerIcon`, `interface.logo`, `interface.screenshots` — need assets under `./assets/`
- `.app.json` — needs the registered connector ID above
- `https://chatgpt.com/connector/oauth/{callback_id}` on the `api.ukumi.ai` redirect allowlist

Reference: [Submit plugins](https://developers.openai.com/plugins/deploy/submission), [Plugin guidelines](https://developers.openai.com/plugins/app-guidelines), [Submission errors](https://developers.openai.com/plugins/deploy/submission-errors).

## Release checklist

1. `version` matching across all three manifests.
2. `claude plugin validate .claude-plugin/plugin.json` and `claude plugin validate .claude-plugin/marketplace.json` both clean.
3. Fresh install from the local marketplace, skills and agents present, MCP server connects and authenticates.
4. Merge to `main`, tag `v<version>`.
5. Codex side only: re-submit through the portal if store metadata or the tool surface changed.
