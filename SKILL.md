---
name: heelo-setup
description: Set up Heelo MCP in a local Codex client. Use when the user asks to install, connect, sign in to, or verify Heelo through the Heelo onboarding repo. Do not use for routine Heelo tool requests.
---

# Set up Heelo in Codex

This preview covers local Codex setup only. Use the Heelo CLI for browser sign-in and MCP registration. Do not configure Claude Code, Cursor, or other clients.

## 1. Check the local environment

Run `codex --version` and `heelo --help` on the machine running the user's Codex client. For an internal test, use the exact Heelo CLI executable supplied by the tester.

Confirm that the Heelo CLI supports `auth status`, `auth login`, and `mcp install`. If the CLI is missing or too old, report that this preview requires a compatible Heelo CLI. Do not substitute an unverified installer. If you are running in a hosted environment that cannot configure the user's local Codex client, tell the user to open the quickstart in local Codex.

## 2. Check sign-in and hand off to the browser if needed

Run `heelo auth status --json` and use the result to decide what to do:

- `ok` — reuse the working credentials and skip browser sign-in. If the source is `HEELO_API_KEY`, note that browser sign-in was not tested. For a fresh-sign-in test, use a test shell without that variable.
- `signed_out` — run `heelo auth login`. If the CLI explicitly reports that saved credentials were rejected, sign in again.
- Network error or unexpected status — report the problem before attempting another sign-in. A backend failure does not mean the user is signed out.

When login waits for approval, keep the CLI process running and ask the user to complete sign-in and approve in their own browser. If the browser does not open, relay the verification URL and short code printed by the CLI to the user who started the setup. Never ask for their password or API key.

Wait for login to finish successfully. If approval is denied, expires, or cannot start, report the reason and do not register the MCP server. If the CLI reports `device_start_failed_404`, say the sign-in endpoint was unavailable at the configured website. Do not switch environments or fall back to manual API-key setup.

## 3. Register Heelo in Codex only

Run `heelo mcp install codex --json`. Check the result for Codex. Treat a new entry or a matching entry that already exists as registered. If a different server is already named `heelo`, leave it unchanged and explain the conflict. Do not remove or overwrite it.

Use the CLI to manage credentials. Do not read or display the saved credentials file, paste keys into chat, or put them in the MCP config.

## 4. Verify that Codex can use Heelo

Check whether Codex can discover tools from the `heelo` server. If the current session has not loaded the new configuration, ask the user to open a fresh local Codex session and finish verification there.

Call one clearly documented, free, read-only Heelo tool if one is available. Do not guess a tool name or use a tool that posts, writes data, or connects another account. If no suitable tool is available, report that the tool call is unverified.

Report sign-in, MCP registration, and the tool call separately. Say **verified** only after a Heelo tool call succeeds through Codex. A working credential or an MCP config entry alone does not prove that the connection works.
