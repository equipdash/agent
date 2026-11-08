# EquipDash for AI agents

[EquipDash](https://equipdash.com) is booking and rental software for businesses that rent out
equipment and run activities, lessons and classes. This repository connects an AI agent to an
EquipDash account through the EquipDash MCP server, and adds a skill that shows the agent how to run
the day: who is picking up and bringing items back, what is booked, what is still free, and who has
paid.

The same files install in Claude Code, Cursor, Codex, Hermes Agent and any client that reads Agent
Skills.

## The server

| | |
|---|---|
| URL | `https://app.equipdash.com/mcp/v1` |
| Transport | Streamable HTTP (JSON-RPC over `POST`) |
| Sign-in | OAuth 2.0 with dynamic client registration, or an EquipDash API key |
| Tools | Bookings, availability, customers, items, payments, waivers, staff, reports and settings. The list a person sees follows their permissions and the business's plan. |

There is no local code in this repository. Every tool call goes to `app.equipdash.com`.

## Install

### Claude Code

```
/plugin marketplace add equipdash/agent
/plugin install equipdash@equipdash
```

Then run `/mcp`, pick `equipdash` and sign in. To add only the server:

```
claude mcp add --transport http equipdash https://app.equipdash.com/mcp/v1
```

### Cursor

Add the plugin from the Cursor Directory, or put this in `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "equipdash": {
      "url": "https://app.equipdash.com/mcp/v1"
    }
  }
}
```

### Codex

```
codex mcp add equipdash --url https://app.equipdash.com/mcp/v1
codex mcp login equipdash
```

### Hermes Agent

```
hermes plugins install equipdash/agent
hermes plugins enable equipdash
```

The plugin brings the skill and the server. To sign in with OAuth, add the server to
`~/.hermes/config.yaml` under the same name (it takes the place of the plugin's entry), then run
`hermes mcp login equipdash`:

```yaml
mcp_servers:
  equipdash:
    url: https://app.equipdash.com/mcp/v1
    auth: oauth
    trust: untrusted
```

With `trust: untrusted`, Hermes asks before any tool that changes data runs, and refuses those
tools in unattended runs. The plugin asks for the same setting.

### The skill on its own

```
npx skills add equipdash/agent
```

This installs `skills/equipdash/SKILL.md` into any agent the [skills CLI](https://skills.sh)
supports. The agent still needs the server above.

### Claude, ChatGPT and other apps

Add a custom connector (or remote MCP server) with the address `https://app.equipdash.com/mcp/v1`
and sign in when the app asks.

## Signing in

| | OAuth sign-in | API key |
|---|---|---|
| Use it for | An AI app a person uses | Your own scripts and agents, or an app that cannot sign in |
| EquipDash plan | Every plan | Pro |
| What the agent can do | What the person who signed in can do in EquipDash | What the key's scopes allow |

**OAuth.** When the client asks, sign in to EquipDash, pick the business and click **Allow**. The
agent then acts as you, so a staff login cannot do more through it than in EquipDash. Every
connected app is listed under **Settings > API & Integrations > AI apps**, where **Disconnect**
stops it at once.

**API key.** Create one under **Settings > API & Integrations** with the **MCP Access** scope
(`mcp:access`) plus the scopes the agent needs. Read scopes are enough to look things up; add write
scopes only if the agent should make bookings or take payments. Send it as a header:

```
Authorization: Bearer YOUR_API_KEY
```

Keep the key in your client's secret store or an environment variable, never in a file you commit.
In Cursor, for example:

```json
{
  "mcpServers": {
    "equipdash": {
      "url": "https://app.equipdash.com/mcp/v1",
      "headers": { "Authorization": "Bearer ${env:EQUIPDASH_API_KEY}" }
    }
  }
}
```

Scopes, error codes and the full tool reference are in the
[MCP documentation](https://api-docs.equipdash.com/mcp/mcp-authentication). The REST API, with the
same keys, is documented at [api-docs.equipdash.com](https://api-docs.equipdash.com).

## What an agent can do with it

- Read the day: sessions and check-ins, rentals going out and due back, late pickups and returns,
  unsigned waivers, unpaid bookings.
- Find bookings by customer, date, item or status, and open one in full.
- Check what is free on a date and quote a price with the shop's own pricing.
- Make, move, change and cancel bookings, and chase unsigned waivers.
- Send a payment link, record a payment taken in person, and refund a card payment.
- Look up customers, items, experiences and the business's sales.

Tools that change data return a preview first and act only when called again with confirmation.
The skill tells the agent to show that preview and wait for a yes. Read-only tools are marked
`readOnlyHint: true`, so clients that gate writes (Hermes with `trust: untrusted`, for example) ask
before everything else.

## Data and privacy

Tool calls send the arguments the agent chooses (dates, ids, names, amounts) to EquipDash, and
return the business's own records to the agent and the model it uses. This repository has no
install scripts, no background processes, no bundled credentials and no telemetry. See the
[EquipDash privacy policy](https://equipdash.com/privacy-policy).

## Files

| Path | Read by |
|---|---|
| `skills/equipdash/SKILL.md` | Every client that reads Agent Skills |
| `.mcp.json` | Claude Code, Cursor |
| `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json` | Claude Code |
| `.cursor-plugin/plugin.json` | Cursor, Cursor Directory |
| `plugin.json`, `mcp.json` | Hermes Agent and other Agent Plugins v1 clients |

## Support

Email support@equipdash.com or use the [contact page](https://equipdash.com/contact). Start a
[free trial](https://app.equipdash.com/free-trial) to try it on your own business.

## License

[MIT](LICENSE)
