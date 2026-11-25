# Heelo Onboarding

Use natural language to set up Heelo in your coding agent—without copying an API key into chat or editing MCP configuration by hand.

This repository contains [`SKILL.md`](./SKILL.md), which helps an agent select the right **existing Heelo CLI commands** to sign in, check authentication, register MCP, and verify that the connection works.

> **Preview:** The local Codex flow has been tested end-to-end against Heelo staging with an installed CLI. A completely fresh public installation and the production rollout have not yet been verified. This skill does not silently install missing software.

## Requirements

- A **local** coding-agent session with access to your machine (Codex is the tested quickstart).
- **Node.js 20+** and a compatible `heelo` CLI on your `PATH`. Heelo CLI **v0.2.2** was used for this skill.
- Browser access to approve Heelo sign-in when required.

For an **authorized staging preview test**, install the preview CLI and confirm it is available:

```sh
npm install -g @heelo-ai/mcp-local@next
heelo --version
heelo --help
```

If `heelo` is unavailable, the skill will report the missing prerequisite rather than run an unverified installer. Use the environment and installation instructions provided by the Heelo team; do not mix staging and production credentials or endpoints.

## Quickstart: connect Heelo to Codex

1. Open a **fresh local Codex session**.
2. Copy this repository's GitHub URL (use the feature-branch URL while testing an unmerged PR).
3. Paste the request below, replacing `<REPO_URL>`:

> Read `SKILL.md` from `<REPO_URL>` and follow it to connect Heelo to **Codex only**. Check whether I'm signed in, guide me through browser approval if needed, register the MCP server, and verify the connection with a documented, free, read-only Heelo tool. Tell me which steps succeeded and which remain unverified.

Complete the browser sign-in yourself if prompted. A newly registered MCP server may require you to **restart Codex** before its tools appear.

## Other requests you can make

| Say this in your local coding agent | What the skill does |
| --- | --- |
| "What Heelo CLI commands are available?" | Shows CLI help; makes no changes. |
| "Am I signed in to Heelo? Don't log in." | Checks authentication only. |
| "Sign me in to Heelo." | Reuses valid credentials or guides browser sign-in. |
| "Show the Heelo MCP configuration without applying it." | Prints a configuration preview only. |
| "Connect Heelo to Claude Code / Cursor / Claude Desktop." | Targets only the named client. |
| "Set up Heelo for all my coding agents." | Explains the broader changes and asks for confirmation first. |
| "Log me out of Heelo." | Explains the impact and asks for confirmation before deleting the saved local key. |

**Scope:** The skill manages CLI onboarding, authentication, and MCP registration. It is **not** a guide to Heelo's business tools (for example, creating campaigns or publishing posts). Those tools are available separately through a connected Heelo MCP server.

## How verification works

The skill reports these stages separately:

1. **Authentication:** The Heelo API accepts the effective credential.
2. **Registration:** The requested client has a Heelo MCP entry.
3. **Discovery:** A fresh client session can see Heelo MCP tools.
4. **End-to-end verification:** An actual, documented, free, read-only Heelo tool call succeeds.

An HTTP 200 from `heelo auth status` or a successful MCP registration **alone** is not end-to-end verification. A successful read-only tool call is required. The skill does not publish, send, create, or modify Heelo data as part of verification.

## Troubleshooting

Check your CLI and authentication status:

```sh
heelo --version
heelo auth status --json
```

For Codex, inspect the registered MCP server:

```sh
codex mcp list
codex mcp get heelo --json
```

If the server is registered but its tools aren't visible, **start a new local Codex session** and retry verification. If your session runs in a sandbox, it may require permission to use the network or update local MCP settings.

**Security:** Never paste passwords or API keys into the coding agent. `heelo auth logout` deletes the saved local credential but **does not revoke** that credential on Heelo's server or remove existing MCP registrations.