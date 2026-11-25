---
name: heelo-setup
description: Manage local Heelo CLI onboarding, authentication, and MCP connections for Codex, Claude Code, Cursor, and Claude Desktop. Use when a user asks to install or set up Heelo, sign in or out, check authentication, connect a specific coding agent, inspect MCP configuration, or verify the connection. Do not use for routine Heelo business/tool requests.
---

# Heelo: local setup and MCP management

Translate the user's natural-language setup request into the smallest safe sequence of **existing Heelo CLI commands**. This skill manages local authentication and MCP registration; it does **not** implement Heelo business workflows or authorize posting, sending messages, changing campaigns, or connecting social accounts.

The quickstart in this repository is currently **Codex-focused**. When invoked from local Codex, target **Codex by default** for requests such as "connect Heelo here." Only configure Claude Code, Cursor, or Claude Desktop when the user explicitly names that client; never silently expand a Codex request into installation for every client. If no target is inferable, ask which client they use. A hosted agent cannot configure the user's local machine unless it actually has access to that machine.

## 1. Supported CLI (v0.2.2)

Use `heelo --help` as the source of truth for the CLI actually installed. The following commands are supported in v0.2.2:

| Command | Effect |
| --- | --- |
| `heelo setup [--no-browser]` | Sign in if needed, then register Heelo in **all detected** supported clients. Broad, potentially multi-client change. |
| `heelo auth login [--no-browser]` | Browser/device-code sign-in; saves a local API key. |
| `heelo auth status --json` | Check the effective key and API reachability; does not change state. |
| `heelo auth logout --json` | Delete the saved local key; **does not revoke** it at the server. |
| `heelo mcp install codex --json` | Register Heelo for Codex only. |
| `heelo mcp install claude --json` | Register Heelo for Claude Code only. |
| `heelo mcp install cursor --json` | Register Heelo for Cursor only. |
| `heelo mcp install claude-desktop --json` | Register Heelo for Claude Desktop only. |
| `heelo mcp install --json` | Register Heelo for **all supported detected** clients. |
| `heelo mcp install --print` | Print example MCP configuration **without modifying clients**. |
| `heelo --help`, `heelo --version` | Inspect the installed CLI. |

Use `--json` when processing results programmatically, but remember browser-login instructions and errors may appear on stderr. `--no-browser` applies to `setup` and `auth login`, not to MCP registration. Do not invent `heelo` subcommands such as `disconnect`, `uninstall`, or `auth refresh`.

## 2. Route the user's intent

Choose **one** workflow based on what the user requested:

| User intent / examples | Required behavior |
| --- | --- |
| "What can the Heelo CLI do?" / "Show commands" | `heelo --help`; explain only, no changes. |
| "Am I signed in?" / "Check my Heelo login" | `heelo auth status --json`; report status without signing in. |
| "Sign in to Heelo" / "Log in again" | Check status; reuse an already working session unless the user explicitly requests a fresh sign-in. Otherwise follow §4. |
| "Set up Heelo in Codex" / "Connect Heelo here" in local Codex | Follow §§3–6 for **Codex only**; do **not** run `heelo setup`. |
| "Connect Heelo to Claude Code / Cursor / Claude Desktop" | Follow §§3–6 for the specifically named client only. |
| "Set up Heelo for all my coding agents" | Explain that multiple detected clients may be modified; obtain explicit confirmation, then follow §7. |
| "Show me the MCP config" | `heelo mcp install --print`; do not register anything. |
| "Is Heelo actually working in Codex?" | Read-only checks and §6; do not reinstall unless requested. |
| "Log out of Heelo" | Explain the local-only effect, confirm the action, then follow §8. |
| "Disconnect / uninstall Heelo from an agent" | This CLI has no removal command. Explain the distinction between removing an MCP entry and deleting credentials. Do **not** log out or remove an MCP entry as a substitute without explicit user approval and a verified client-specific procedure. |

For a request that is only a status check, never initiate login, installation, logout, or an MCP tool action. If the intent is ambiguous in a way that could affect multiple clients or accounts, ask a concise clarifying question instead of guessing.

## 3. Check the local environment and installed version

1. Establish that commands run **on the machine where the requested agent is installed**. For Codex, check `codex --version` if available. For other clients, use their supported presence check; do not assume a client exists just because its name is mentioned.
2. Run `heelo --version` and `heelo --help`. Confirm the intended subcommands and target client appear. A shell's `heelo` may differ from a source checkout's `node dist/heelo.js`; do not silently substitute one for the other. For an internal test, use the exact CLI executable supplied by the tester.
3. If Heelo is missing, report the prerequisite. **Do not run an unverified remote install script, use `sudo`, edit shell startup files, or download executables without authorization.** For an explicitly authorized **staging preview test**, Node.js 20+ and the documented npm package `@heelo-ai/mcp-local@next` can be used after verifying its published version and install source. Do not assume staging is a production release. Otherwise ask the user to use the current official installation instructions and verify they are live before executing them.
4. Preserve the user's chosen environment. `HEELO_WEB_URL` selects the sign-in website, `HEELO_API_URL` selects the API, and `HEELO_API_KEY` overrides the saved key. Never switch between production and staging or print an environment-provided API key without permission. Do not read, paste, or expose credential files.
5. On a local-source test, note that a successful `node dist/heelo.js` CLI run does **not**, by itself, prove the published MCP server works. In v0.2.2 the installer pins the server to the CLI package version; check what is actually registered when testing a build.

## 4. Check authentication; sign in only when needed

Run:

```sh
heelo auth status --json
```

Parse its JSON **even if the command exits nonzero**. Interpret it carefully:

- `status: "ok"` with HTTP 200: the effective key works. Reuse it, unless a fresh sign-in was explicitly requested. Report whether `source` is a saved-credentials path or `HEELO_API_KEY`. An environment key means browser login is **not** verified.
- `status: "signed_out"`: there is no effective key. If the user's request requires authentication or setup, run `heelo auth login` (or `heelo auth login --no-browser` if needed).
- `status: "key_not_working"`, `http_status: null`: the API could not be reached. **Do not** treat this as signed out or automatically start login. Explain the connection problem.
- `status: "key_not_working"`, HTTP 401/403: the effective key appears rejected. If `source` is `HEELO_API_KEY`, tell the user that it overrides saved login, and do not display or replace its value. If using a saved key, offer a new browser sign-in when the user has requested setup or reauthentication.
- Other non-200 HTTP responses, malformed JSON, or an unexpected status: report the actual error; do not guess that reauthentication will fix it.

### Browser handoff

For a required login, run `heelo auth login` and keep its process alive while waiting for approval. Show the **verification URL and short device code** from the CLI and ask the user to approve in their own browser. If local browser launching is unavailable (SSH, no display, sandbox), use `heelo auth login --no-browser`, and let the user open the printed URL on another device. Never ask for the account password or API key.

If the agent's sandbox blocks network access, ask for the normal permission needed to retry; do not bypass restrictions silently. If authorization is denied, expires, or the device endpoint fails (including `device_start_failed_404`), stop and describe the problem. Do not automatically change backends or substitute manual key entry.

After login reports success, rerun `heelo auth status --json` to verify the **effective** key and chosen API. If `HEELO_API_KEY` is set, browser login may save a new key while status still uses that environment key; state that limitation explicitly. Do not proceed to claim authenticated setup when the effective key is failing.

## 5. Register only the requested MCP client

For a specific client, run exactly one of:

```sh
heelo mcp install codex --json
heelo mcp install claude --json
heelo mcp install cursor --json
heelo mcp install claude-desktop --json
```

Choose **only** the command matching the user's target. Check `results[]` for that client:

- `status: "added"`: registration was accepted; the `detail` may indicate newly added, already present, updated, or **kept because its entry has custom environment settings**. A kept custom entry may use a different key or backend; never claim it uses the saved Heelo key without verification.
- `status: "not_found"`: the client was not detected. Explain this; do not claim setup succeeded.
- `status: "failed"`: display the sanitized reason and stop for that target. In particular, if another MCP server is already named `heelo`, **do not remove or overwrite it**; it may be the user's hosted or custom server. Ask the user how to proceed.

An `added` result is **registration**, not proof that tools can be called. A client restart may be needed, especially for Claude Desktop (fully quit and reopen it). Do not hand-edit agent configuration or inject keys into MCP settings. The Heelo CLI manages registration and reads its key locally.

## 6. Verify without changing Heelo data

Verify in increasing order of confidence, and state which level was reached:

1. **Authentication:** `heelo auth status --json` returns `ok` for the intended backend.
2. **Registration:** the requested client's Heelo MCP entry is present. For Codex, `codex mcp list` or `codex mcp get heelo --json` can inspect registration. Do not display config secrets.
3. **Tool discovery:** a **fresh session of the target agent** can discover the `heelo` MCP server and available tools. Restart if the prior session did not reload MCP settings.
4. **End-to-end verification:** with the user's setup/verification request, invoke **one actually discovered, documented, free, read-only Heelo tool** through the target agent, using only the minimum safe arguments. Do not invent tool names. Do not invoke tools that create, post, send, charge, connect accounts, or mutate data. A read-only tool returning an empty list may still count as success if the call itself succeeded.

Only say **"end-to-end verified"** after the actual MCP tool call succeeds. If discovery succeeds but no appropriate tool can be safely called, say **"registered and discovered; tool call unverified"**. A CLI status check or MCP config entry alone is insufficient. Do not reveal returned private data unnecessarily.

## 7. Explicit all-agent setup

`heelo setup` and bare `heelo mcp install` affect **every supported detected client**, including Codex, Claude Code, Cursor, and Claude Desktop. **Never use them for a request naming just one client.**

If the user explicitly asks to configure *all* their agents, describe that scope and confirm before changing multiple clients. Check the environment and preflight authentication status as in §§3–4 first; if the effective API is unreachable or an environment-key override is failing, stop. If `signed_out`, let `heelo setup` perform the browser login instead of running a redundant standalone login. Then run:

```sh
heelo setup --json
```

The command handles sign-in if necessary and attempts multi-client registration. Interpret its top-level `status` (`ready`, `not_changed`, or `no_agent`) **together with every item in `results[]`**: some clients may be absent or fail while others are registered. Never turn partial success into "all agents verified". Verify each requested/installed agent separately, as access allows. Use `heelo setup --no-browser --json` only when browser auto-opening is unsuitable.

## 8. Logout and disconnection

Before `heelo auth logout --json`, make sure the user wants to remove the **saved local Heelo credential** and confirm the deletion. Explain:

- Logout removes the local saved key; it **does not revoke** that key on Heelo's server. Server-side revocation, if desired, is a separate user action in the Heelo dashboard.
- If `HEELO_API_KEY` is set, that key **still takes precedence** after logout, and `auth status` may still report `ok`.
- Existing MCP registrations remain configured; removing an MCP connection from an agent is **not** the same as logging out.

Run `heelo auth logout --json` only after confirmation. Interpret `signed_out` as a saved key deleted and `no_saved_key` as nothing to delete. Do not claim the user has been disconnected everywhere, and do not revoke a server-side key or remove client configuration without a separate explicit request and verified process.

## 9. Report the outcome precisely

Keep the final report concise and distinguish:

- **CLI:** found, missing, or incompatible (include version if known).
- **Authentication:** working, signed out, blocked, or environment-key override; identify production/staging only when known.
- **Registration:** which client(s) were added, already present, preserved, skipped, or failed.
- **Verification:** not attempted, registered only, tools discovered, or actual read-only tool call succeeded (include exact tool name and success/failure).
- **Next action:** the single user action needed, such as approving a device code, granting sandbox permission, restarting the agent, or resolving an existing-server conflict.

Do not report success based on an approval screen, a completed login, or a registration response alone. Never print full credentials, passwords, cookies, or private configuration contents.