# @nmbrai/claude-code

> **Beta.** This plugin is new and has been tested against the framework's SDK and a real nmbr server, but not yet by many people inside the framework itself. If something breaks, please tell us at support@nmbr.ai (include the version and the error) — fixes ship fast.


**Text your Claude Code session from your phone — and approve its permission prompts there.** One package, two roles:

- **Claude Code channel** (`--channels`): messages you send to your nmbr agent are pushed into your running session; Claude replies through the `reply` tool; when Claude needs a permission (Bash, Write, …) you get an **Approve / Reject card in nmbr** — tap on the phone, the session continues.
- **Plain MCP server** for any MCP host (Claude Desktop, Cursor, …): tools `send_message`, `propose_action`, `list_conversations`, `get_messages`, `whoami`. Pull only — nothing wakes a vanilla MCP host on an inbound message; only Claude Code consumes channel events.

Runs on Node ≥ 18 (built JavaScript; no Bun needed). Built on [`@nmbrai/sdk`](https://www.npmjs.com/package/@nmbrai/sdk).

## Setup (Claude Code)

1. In the nmbr app: **Agents → Yours → Create an agent**. Copy the token (shown once).
2. Install the plugin and configure it:
   ```
   /plugin marketplace add <marketplace>        # where this plugin is listed
   /plugin install nmbr@<marketplace>
   /nmbr:configure agent:…                       # saves ~/.claude/channels/nmbr/.env
   /nmbr:access allow 123-456-789               # your own nmbr
   ```
3. Restart with the channel enabled:
   ```bash
   claude --channels plugin:nmbr@<marketplace>
   # while the plugin is not on Anthropic's approved list (research preview):
   claude --dangerously-load-development-channels plugin:nmbr@<marketplace>
   ```
4. Text your agent from the phone. Ask it to run the tests; approve the Bash permission on the card that appears.

### Without a marketplace (bare MCP server)

Add the server to `.mcp.json` and load it by server name:

```json
{ "mcpServers": { "nmbr": { "command": "npx", "args": ["-y", "@nmbrai/claude-code"] } } }
```
```bash
NMBR_AGENT_TOKEN=agent:… claude --dangerously-load-development-channels server:nmbr
```

## Who can reach the session

Only people who added your agent as a contact in nmbr can message it at all; on top of that this channel keeps an allowlist of **nmbrs** in `~/.claude/channels/nmbr/access.json`, managed with `/nmbr:access` (`allow`, `remove`, `list`, `policy allowlist|open`). Someone not on the list gets a one-time note telling them who to ask, and shows up as *pending* in `/nmbr:access list`. Allowed senders can also approve permissions, so allow only people you'd trust at your keyboard.

Because nmbr authenticates every sender, there are no pairing codes: an nmbr **is** the identity.

## Permission relay

When Claude Code opens a permission prompt, the channel proposes an nmbr action (`kind: claude_code.permission`, 10-minute expiry) to every allowed sender. **Approve** → `allow`; **Reject** or expiry → `deny`; first decision wins. The terminal dialog stays open too — whoever answers first wins. Typing `yes abcde` / `no abcde` in the chat also works.

## Environment

| Variable | Meaning |
|---|---|
| `NMBR_AGENT_TOKEN` | The agent token (or in `<state-dir>/.env`) |
| `NMBR_BASE_URL` | API base, default `https://nmbr.ai/api` |
| `NMBR_STATE_DIR` | Defaults to `${CLAUDE_CONFIG_DIR:-~/.claude}/channels/nmbr` |

Docs: https://nmbr.ai/developers/docs/ · Issues: support@nmbr.ai

---

## About this repository

This is the **distribution mirror** for the nmbr Claude Code plugin, submitted to the
`claude-community` marketplace. The source of truth lives in the nmbr monorepo; the
MCP server itself ships as [`@nmbrai/claude-code`](https://www.npmjs.com/package/@nmbrai/claude-code)
on npm — this repo's `.mcp.json` launches it with `npx -y @nmbrai/claude-code`, so
installs always resolve the current published server with its dependencies. Issues:
support@nmbr.ai.
