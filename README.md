# @nmbrai/claude-code

> **Beta.** Verified live inside Claude Code: phone round-trip and the permission-card relay (approve and reject, fail-closed expiry) proven end-to-end. If something breaks, please tell us at support@nmbr.ai (include the version and the error) — fixes ship fast.


**Text your Claude Code session from your phone — and approve its permission prompts there.** One package, two roles:

- **Claude Code channel** (`--channels`): messages you send to your nmbr agent are pushed into your running session; Claude replies through the `reply` tool; when Claude needs a permission (Bash, Write, …) you get an **Approve / Reject card in nmbr** — tap on the phone, the session continues.
- **Plain MCP server** for any MCP host (Claude Desktop, Cursor, …): tools `send_message`, `propose_action`, `list_conversations`, `get_messages`, `whoami`. Pull only — nothing wakes a vanilla MCP host on an inbound message; only Claude Code consumes channel events.

Runs on Node ≥ 18 (built JavaScript; no Bun needed). Built on [`@nmbrai/sdk`](https://www.npmjs.com/package/@nmbrai/sdk).

## Setup (Claude Code)

1. In the nmbr app: **Agents → Yours → Create an agent**. Copy the token (shown once).
2. Add **nmbr** from Claude's plugin directory: on claude.ai under **Customize → Plugins → Discover**, or in Claude Code with `/plugin directory`. A plugin you add on claude.ai shows up in Claude Code at the next session start. Then configure it in Claude Code:
   ```
   /nmbr:configure agent:…                       # saves ~/.claude/channels/nmbr/.env
   /nmbr:access allow 123-456-789               # your own nmbr
   ```
3. Restart with the channel enabled. Use the plugin's name exactly as `/plugin` lists it under **Installed** (a plugin added from the directory is listed as `nmbr@synced`). Claude Code channels are a research preview: individual accounts load them with the development flag; Team and Enterprise admins can allow the plugin through managed settings, after which the plain `--channels` form works.
   ```bash
   claude --dangerously-load-development-channels plugin:nmbr@synced
   # Team / Enterprise, once an admin has allowed the plugin in managed settings:
   claude --channels plugin:nmbr@synced
   ```
   If the channel does not start this way, use the bare MCP server below; it is the same server.
4. Text your agent from the phone. Ask it to run the tests; approve the Bash permission on the card that appears.

### Without the directory (bare MCP server)

Add the server to `.mcp.json` and load it by server name:

```json
{ "mcpServers": { "nmbr": { "command": "npx", "args": ["-y", "@nmbrai/claude-code@0.1.0-beta.3"] } } }
```
```bash
NMBR_AGENT_TOKEN=agent:… claude --dangerously-load-development-channels server:nmbr
```

In this mode the plugin's skills (`/nmbr:configure`, `/nmbr:access`) are NOT loaded —
they ship with the plugin packaging, not the bare server. Configure by hand instead:

```bash
mkdir -p ~/.claude/channels/nmbr && chmod 700 ~/.claude/channels/nmbr
printf 'NMBR_AGENT_TOKEN=%s\n' 'agent:…' > ~/.claude/channels/nmbr/.env
cat > ~/.claude/channels/nmbr/access.json <<'JSON'
{ "dmPolicy": "allowlist", "allowFrom": ["123-456-789"], "pending": {} }
JSON
```

(`access.json` is re-read on every message — no restart needed after edits.)

## Who can reach the session

Only people who added your agent as a contact in nmbr can message it at all; on top of that this channel keeps an allowlist of **nmbrs** in `~/.claude/channels/nmbr/access.json`, managed with `/nmbr:access` (`allow`, `remove`, `list`, `policy allowlist|open`). Someone not on the list gets a one-time note telling them who to ask, and shows up as *pending* in `/nmbr:access list`. Allowed senders can also approve permissions, so allow only people you'd trust at your keyboard.

Because nmbr authenticates every sender, there are no pairing codes: an nmbr **is** the identity.

## Permission relay

When Claude Code opens a permission prompt, the channel proposes an nmbr action (`kind: claude_code.permission`, 10-minute expiry) to every allowed sender. **Approve** → `allow`; **Reject** or expiry → `deny`; first decision wins. The terminal dialog stays open too — whoever answers first wins. Typing `yes abcde` / `no abcde` in the chat also works.

## What this plugin runs, stores and sends

- **Runs**: one local MCP server, the npm package [`@nmbrai/claude-code`](https://www.npmjs.com/package/@nmbrai/claude-code) at the exact version pinned in `.mcp.json`, started with `npx`. Nothing else is executed, and there are no hooks.
- **Sends**: it talks to one service, the nmbr API at `https://nmbr.ai/api` (or the base you set in `NMBR_BASE_URL`), authenticated with your agent token. To nmbr it sends the replies Claude writes with the `reply` tool, the permission requests Claude Code raises (the tool's name, the request's description and a preview of its input, shown to you as an approval card), and the calls behind the tools listed above. From nmbr it receives the messages people send to your agent.
- **Stores**: on your machine only, under `~/.claude/channels/nmbr/`: your agent token (`.env`) and the allowlist of senders (`access.json`).
- **Does not**: read your files or your shell environment beyond the three variables below, change Claude Code's permission settings, or contact any other host.

## Environment

| Variable | Meaning |
|---|---|
| `NMBR_AGENT_TOKEN` | The agent token (or in `<state-dir>/.env`) |
| `NMBR_BASE_URL` | API base, default `https://nmbr.ai/api` |
| `NMBR_STATE_DIR` | Defaults to `${CLAUDE_CONFIG_DIR:-~/.claude}/channels/nmbr` |

Docs: https://nmbr.ai/developers/docs/ · Issues: support@nmbr.ai

---

## About this repository

This is the **distribution mirror** for the nmbr Claude Code plugin, submitted to
Anthropic's plugin directory. The source of truth lives in the nmbr monorepo; the
MCP server itself ships as [`@nmbrai/claude-code`](https://www.npmjs.com/package/@nmbrai/claude-code)
on npm. This repo's `.mcp.json` launches it with `npx -y @nmbrai/claude-code@0.1.0-beta.3`:
an exact version, raised here with every release. Issues: support@nmbr.ai.
