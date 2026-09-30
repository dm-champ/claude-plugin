---
name: dm-champ
description: How DM Champ works and how to run a DM Champ account through its MCP tools. Use when the user wants to set up, change, check or debug anything in DM Champ (AI agents, knowledge base and FAQs, channels, entry points, contacts, lists, tags, broadcasts, WhatsApp templates, follow-ups, appointments, deals, tasks, automations, agency client accounts), asks why their bot did or did not reply, or asks how a DM Champ feature works or what is new.
---

# DM Champ

DM Champ is an AI messaging platform. AI agents answer a business's customers on WhatsApp, Instagram, Facebook Messenger, SMS, Telegram, email, a website chat widget and more, and every conversation lands in one inbox. The `dm-champ` MCP server that ships with this plugin exposes the user's own account as tools, authenticated with their API key.

This file is the map. The help docs are the territory, and you can read them live.

## 1. Read the docs before you act

The server has three docs tools that read the live help site, so they know features that are newer than this file and newer than your training data:

- `docs_search`: find the section that answers a question. Use it before you explain a feature, before you pick between two ways of doing something, and before the first time you use a kind of tool in a conversation.
- `docs_read`: read the whole page (pass the URL from a search result, and `section` for one heading of a long page).
- `docs_list`: the table of contents.

Habits that keep you right:

- The pages under `/api/` describe each endpoint's request body and its rules. The tools are those endpoints, so the API page for an area is the reference for that area's tools.
- For "what is new" or "did X ship", search with words like "latest" or "changelog".
- When the docs and this file disagree, the docs win. When the docs do not cover something, say so instead of guessing.
- Tools prefixed `linkedin_` are not DM Champ tools. They belong to SDRPilot, a separate LinkedIn outreach product with its own plugin. Leave them alone unless the user asks for LinkedIn work by name.

## 2. The model in one page

**Agent.** The brain of the bot: its personality, instructions, knowledge, language and abilities. An agent has no contact list, no channel and no schedule of its own.

- Edits to an agent are saved to a **draft**. Live conversations keep running on the previous setup until the draft is published. A draft can be discarded. A few settings save live with no publish step, the agent docs list which.
- Every publish saves a version that can be restored. A restore comes back as a draft, so nothing goes live until it is published again.
- Pausing an agent stops it everywhere. Test mode makes it answer only the contacts on its test list, which is the safe way to try an agent on a real channel.

**Knowledge base.** Sources (website pages and documents), the FAQ library and knowledge groups. An FAQ is only used by an agent when it is linked to that agent. "Global" means reusable, not "applied everywhere".

**Channel.** A connected WhatsApp number, Instagram account, Telegram account, email inbox, chat widget and so on. Connecting a channel gets messages into the account. It does not decide who answers them.

**Entry Point.** The routing rule: it hands NEW conversations on a channel to one agent. It never moves a conversation that is already under way. Keyword and comment rules are checked first, then the channel's default. A channel with no matching Entry Point and no default gets no automatic reply.

**Contacts, lists, tags, custom fields.** Lists are static (filled by hand or by import) or smart (filled by rules). Agents apply tags after a reply. The AI does not see a contact's tags while it chats, but it does see custom field values.

**Broadcast.** The outbound send: one channel, an opening message, a list. Replies go to an agent or to the team inbox. On the WhatsApp Business API the opening message must be an approved template. Other channels send plain text.

**The 24 hour window.** On WhatsApp Business, Instagram DMs and Messenger you can reply freely for 24 hours after the contact's last message. After that, WhatsApp only allows an approved template.

**Follow-ups.** Set per agent: the nudges sent when a conversation goes quiet.

**Appointments.** Event types on the built-in calendar, with optional Google Calendar sync. An agent books when bookings are switched on for it.

**Deals and tasks.** Pipelines with stages, and a task board that also holds the alerts agents raise for a human.

**Automations.** One trigger followed by steps. Through MCP they can be listed and read, not created or edited. Those are built in the app.

**Custom functions and MCP servers.** The extra tools an agent can call during a conversation, switched on per agent.

**Agency.** An agency account manages client accounts (sub-accounts). Snapshots copy an agent setup between accounts and never copy contacts, chats or channels. White-label puts the agency's own brand and domain on the app.

**Credits.** Credits are spent when the AI does work (a reply, a tool call, a tag, a summary, a follow-up), not per message sent or received. At zero credits the AI stops replying and broadcasts cannot launch.

**Campaigns.** Classic campaigns were replaced by three things: the Broadcast (the send), the Agent (the brain) and the Entry Point (the routing). For new work use those. Tools with `campaign` in the name act on classic campaigns.

## 3. Picking a tool out of several hundred

- Tool names read `verb_object`: `list_agents`, `get_agent`, `create_agent`, `update_agent`, `delete_agent`. Search the tool list by the object ("broadcast", "entry point", "template", "knowledge base").
- Path and query parameters are top-level arguments. A JSON request body goes in the single `body` argument.
- Every result starts with the HTTP status. 401 means the key is wrong or revoked. 403 means the key is not allowed to do this (a scoped key can be read-only or limited to some areas). 429 means slow down.
- Agencies: pass `sub_account_id` on tools that accept it to act on a client account. Check the tool's schema, not every tool takes it.
- Prefer the narrow read first. `count_contacts`, `get_contact_stats` and `get_analytics_summary` answer a question without pulling whole lists.

## 4. Core workflows

Each list names the tools. Read the matching docs page for the fields before the first call.

**Build an agent and put it live**
1. `create_agent`, then `update_agent` for instructions and behaviour. The changes sit in the draft.
2. Give it knowledge: see the next workflow.
3. Check it: `get_agent`, `agent_readiness_score`. Ask the user to test it in the app's Try Out step, or switch on test mode for a real channel.
4. With the user's OK, `publish_agent_draft`. `discard_agent_draft` backs out.
5. Route a channel to it: `list_channel_connections`, then `create_agent_entry_point` (or `set_channel_default_agent` for the plain default). Confirm with `list_agent_entry_points` and `get_agent_routing_status`.

**Add knowledge**
- A single answer: `create_faq` with the agent's id in the body, which adds it straight to that agent's knowledge base. A whole group of FAQs: `apply_knowledge_base_group_to_agent`.
- A website: `discover_website_pages`, let the user pick, `estimate_knowledge_base_import_cost`, then `bulk_import_website_pages` or `add_website_page_to_knowledge_base`.
- A document: `add_a_file_uploaded_or_pdf_link_to_knowledge_base`.
- Pages and documents are knowledge sources. Give an agent access with `attach_knowledge_sources_to_agent`.
- Imports run in the background: poll `get_knowledge_base_source_status`.

**Send a broadcast on the WhatsApp Business API**
1. The audience: `get_user_lists`, `create_list`, `preview_smart_list_rules`.
2. `create_broadcast` with the channel, list and reply handling.
3. The template: pick an approved one (`get_whatsapp_templates`, `select_broadcast_template`) or write one and send it for approval (`submit_broadcast_template`). Approval can take minutes to days, so check `get_whatsapp_template_approval_status` instead of waiting in a loop.
4. `estimate_broadcast_cost`, show the user the audience size and the cost.
5. Only after an explicit yes: `launch_broadcast`. `pause_broadcast` stops it.

**Import contacts and build lists**
1. `start_csv_contact_import` or `import_contacts`, then poll `get_contact_import_job_status`.
2. `create_list` for a static list, or a smart list from rules. Test the rules with `preview_smart_list_rules` first.
3. Adding people to a list that a running broadcast watches will message them. Warn the user before you do.

**Follow-ups**
- Switch them on and configure them on the agent (`update_agent`). `generate_agent_follow_up_templates` writes the follow-up messages for you. On the WhatsApp Business API they need approval like any template.
- A one-off nudge for one contact: `schedule_manual_follow_up`, cancelled with `cancel_manual_follow_up`.

**Appointments**
1. `create_event_type`.
2. Switch bookings on for the agent (`update_agent`, the agents API page lists the fields).
3. To book on the user's behalf: `get_available_appointment_slots`, then `book_appointment`.

**Agency: a new client**
1. `create_subaccount`.
2. `list_snapshots`, `list_snapshot_apply_targets`, then `apply_snapshot_to_a_subaccount`. Applying the same snapshot twice creates duplicates, so check what the account already has.

**Why did the bot not reply?**
Work from the conversation outward: `get_contact_messages`, then the agent (`get_agent`: paused, test mode, active hours), the routing (`get_agent_routing_status`, `list_agent_entry_points`), the channel (`list_channel_connections`, `get_phone_number_status`), credits (`get_credit_usage`) and `list_error_log_entries`. Then `docs_search` the symptom.

## 5. Guardrails

Get the user's explicit OK in the conversation, stating exactly what will happen, before any of these:

- **Anything that messages real people**: `send_message_to_contact`, template sends, `launch_broadcast`, `bulk_send_whatsapp_template`, follow-ups, adding contacts to a list a live broadcast watches.
- **Anything that cannot be undone**: deleting contacts (their chat history goes with them), bulk deletes, deleting a list together with its contacts, deleting a tag or a pipeline with its deals, releasing a phone number, deleting a WhatsApp template, deleting a client account.
- **Anything that cuts access**: `rotate_api_key` and `revoke_api_key` take effect at once and cut off this connection too.
- **Anything that spends**: buying a phone number, large knowledge base imports, a broadcast. Show the estimate first.

Working rules:

- Read before you write. Fetch the object, tell the user what will change, then change it.
- Make agent changes in the draft, show what changed, publish on a yes.
- Outreach on WhatsApp is where accounts get banned, and a ban by the channel provider cannot be reversed by support. Identical openers to strangers, blasting an old list and sending at night are the risky patterns. Start small, and read the docs on safe outreach (`docs_search "scaling whatsapp outreach"`) before helping with volume.
- Never ask for, print or store the API key. It lives in the plugin's settings.
- One step at a time on bulk changes. Do one, check the result, then the rest.

## 6. Work that finishes later

These return before the work is done. Poll the status tool, do not repeat the call:

| Started with | Check with |
|---|---|
| knowledge base import, document upload | `get_knowledge_base_source_status` |
| website refresh | `get_website_refresh_status` |
| contact import | `get_contact_import_job_status` |
| contacts export | `get_contacts_export_job_status` |
| WhatsApp template approval | `get_whatsapp_template_approval_status` |
| auto-tagging with AI | `get_auto_tag_run_status` |
| calendar import | `get_calendar_import_job` |
| pairing WhatsApp Web | `get_the_connection_status` |
| connecting Telegram | `get_live_connection_status` |

## 7. Where things are in the docs

| Area | Docs path |
|---|---|
| First steps, contacts, lists, tags, custom fields | `/get-started/` |
| Channels | `/messaging-channels/` |
| WhatsApp templates | `/templates/` |
| Agents and entry points | `/ai-agents/` |
| Knowledge base, follow-ups, custom functions, MCP servers | `/ai-automation/` |
| Automations | `/automations/` |
| Broadcasts | `/broadcasts/` |
| Inbox | `/chats/` |
| Appointments, deals, tasks | `/appointments/`, `/deals/`, `/tasks/` |
| Integrations, webhooks, connecting AI clients | `/integrations/` |
| Settings, team, privacy | `/settings/`, `/privacy/` |
| Billing and credits | `/billing/` |
| Agency: client accounts, snapshots, white-label | `/agency/` |
| API reference | `/api/` |
| What changed | `/changelog/` |
