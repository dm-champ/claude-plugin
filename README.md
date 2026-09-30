# DM Champ plugin for Claude Code

Let Claude work inside your own DM Champ account, and know what it is doing while it is there.

The plugin bundles three things:

1. **The connection.** The MCP server for your account, authenticated with your own API key.
2. **A skill.** The playbook Claude reads first: how DM Champ is put together, the common workflows step by step, and what it must ask you before doing.
3. **Live help docs.** The server gives Claude `docs_search`, `docs_read` and `docs_list`, which read the help site as it is today. When we ship a feature and document it, Claude can look it up the same day.

It covers AI agents, the knowledge base, channels, entry points, contacts, lists, broadcasts, WhatsApp templates, follow-ups, appointments, deals, tasks and agency client accounts.

## Install

You need [Claude Code](https://claude.com/claude-code) and your DM Champ API key (in the app: Settings > Integrations > API Key).

```bash
claude plugin marketplace add dm-champ/claude-plugin
claude plugin install dm-champ@dm-champ
```

Then give the plugin your key. Start `claude` and run:

```
/plugin configure dm-champ@dm-champ
```

Paste the key when it asks. It is stored in your system keychain, not in a file. Restart Claude Code afterwards so the connection picks it up.

Check that it works: start `claude`, run `/mcp` and look for the server marked connected, then ask something like "which of my agents are live, and which channel does each one answer?".

## What to ask

- "Build an agent for my dental practice from our website, then walk me through testing it before it goes live."
- "Why did the bot not answer the last message from +31 6 1234 5678?"
- "Add these five FAQs to my support agent."
- "Draft a WhatsApp broadcast for my VIP list, show me the cost, and wait for my go."
- "What changed in automations this month?"
- "Which of my client accounts used the most credits this week?"

## What Claude will and will not do on its own

The skill tells Claude to read before it writes, to show you what will change, and to get your explicit go before anything that messages real people (a broadcast, a template, a direct message), anything that cannot be undone (deleting contacts, releasing a number), and anything that spends credits.

Your API key is the access control: Claude can do what the key can do. To let Claude look without being able to change anything, give the plugin a read-only scoped key instead of your main key. See [API Keys](https://help.dmchamp.com/api/api-keys/) in the help docs.

## Good to know

- **Claude Code only for now.** The plugin installs in the Claude Code terminal app. The Claude desktop and web apps are not supported yet.
- **Updates.** `claude plugin marketplace update dm-champ`, then `claude plugin update dm-champ@dm-champ`. The help docs need no update, Claude reads them live.
- **Remove.** `claude plugin uninstall dm-champ@dm-champ`.

## Help

- Help docs: https://help.dmchamp.com
- Questions: hi@dmchamp.com

## For maintainers

```bash
python3 scripts/check.py --live                     # wording rules, every tool name the skill mentions exists on the live server
claude plugin validate . --strict                   # the marketplace
claude plugin validate ./plugins/dm-champ --strict  # the plugin
```

Every statement in the skill must be backed by the help docs, and anything that can change (limits, prices, what is in beta) belongs in the docs, not in the skill. Bump `version` in `plugins/dm-champ/.claude-plugin/plugin.json` when the skill or the MCP config changes, otherwise installed copies do not update.
