Title: Send Your First Message to Slack via AI Agent
Date: 2026-05-12
Category: AI Agents
Tags: slack, nemoclaw, ai-agents, mcp, automation, sandbox, messaging
Slug: send-your-first-message-to-slack-via-ai-agent
---

# Send Your First Message to Slack via AI Agent

Your sandbox is running. Your agent is alive inside it. Now — can it actually *do* anything?

Let's find out by sending a real message to Slack.

---

## Why Slack?

Slack is the perfect first test. It's:
- External (proves your agent can reach outside itself)
- Verifiable (you can see the message land in real time)
- Useful (this is the foundation of real automation workflows)

If your agent can send a Slack message, it can do a lot more.

---

## What You Need Before Starting

- A running NemoClaw sandbox (see Blog 1)
- A Slack workspace you control
- A Slack Bot Token (`xoxb-...`) with `chat:write` permission
- The channel ID you want to post to

---

## Step 1: Get Your Slack Bot Token

Go to [api.slack.com/apps](https://api.slack.com/apps) and create a new app.

Under **OAuth & Permissions**, add the scope:
```
chat:write
```

Install the app to your workspace. Copy the **Bot User OAuth Token** — it starts with `xoxb-`.

---

## Step 2: Find Your Channel ID

In Slack, right-click the channel name → **Copy link**. The channel ID is the last segment:

```
https://app.slack.com/client/T0123ABCD/C0456EFGH
                                        ^^^^^^^^^^
                                        This is your channel ID
```

---

## Step 3: Configure the MCP Slack Plugin

Inside your sandbox, check what plugins are available:

```bash
openclaw plugins list
```

Look for a Slack or MCP messaging plugin. If it's there but disabled, the config lives on your **host**, not inside the sandbox.

On your host machine, open or create:

```
~/.config/openclaw/config.json
```

Add your Slack credentials:

```json
{
  "plugins": {
    "slack": {
      "enabled": true,
      "token": "xoxb-your-token-here",
      "defaultChannel": "C0456EFGH"
    }
  }
}
```

Restart your sandbox after saving:

```bash
# Exit sandbox first
exit

# Restart
nemoclaw restart kaav-assistant
nemoclaw shell kaav-assistant
```

---

## Step 4: Send the Message

From inside your sandbox, ask your agent to send a message:

```
Send a message to Slack: "Hello from NemoClaw — agent is live."
```

Or if you're testing via the MCP interface directly:

```bash
/mcp slack send --channel C0456EFGH --text "Hello from NemoClaw"
```

---

## Step 5: Verify It Landed

Switch to Slack. You should see the message in your target channel, posted by your bot.

If you see it — congratulations. You just closed the loop between an isolated AI sandbox and the real world.

---

## What Actually Happened Under the Hood

This is worth understanding because it shapes how you build more complex workflows.

```
You (in sandbox)
    └──► Agent processes your request
             └──► MCP Slack plugin called
                      └──► Plugin sends HTTP request to Slack API
                                └──► Message appears in Slack
```

Your agent never directly hit the Slack API. It went through the **MCP (Model Context Protocol) plugin layer** — a controlled interface that handles external calls on the agent's behalf.

This is important: the sandbox doesn't have open internet access. The MCP plugin is a curated, permission-gated channel for specific external services. Slack works because there's an approved plugin for it. Random HTTP calls to arbitrary URLs? Still blocked.

---

## Troubleshooting

**Message didn't send — `channel_not_found`**
Make sure your bot has been invited to the channel:
```
/invite @your-bot-name
```

**`missing_scope` error**
Your token doesn't have `chat:write`. Go back to your Slack app settings and add it, then reinstall.

**Plugin not found in `openclaw plugins list`**
The plugin may need to be installed on the host gateway. See Blog 3 for how ACP agents and host-side config work.

---

## What You've Proved

By sending that one Slack message, you've demonstrated:

- Your sandbox is running and your agent is responsive
- The MCP plugin layer is wired up correctly
- External communication from an isolated agent is possible — through controlled channels
- You have the foundation for real automation: Slack alerts, status updates, notifications, auto-responses

From here, the patterns scale. Monitor a process and Slack on failure. Summarise a doc and post it. Watch a channel and respond to mentions.

It all starts with that first message.

---


