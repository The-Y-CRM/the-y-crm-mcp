# Tool reference

All 23 tools of the The-Y CRM MCP server, as of version 1.1.68.4. The AI only sees the tools its connection may use (see [Permissions](../README.md#permissions)).
IDs are UUIDs from the account. Titles, descriptions and the sentences in answers come in the language of the person who connected (English, German or Turkish).

Every response is JSON (`structuredContent`, plus the same JSON as text). Errors come with `isError: true`, a machine-readable `error` code (for example `invalid_field`, `invalid_reference`, `not_found`, `insufficient_scope`, `forbidden`, `rate_limited`, `terms_required`) and a sentence that says what to fix. Many answers also contain a `link` into the app and a `human_step` sentence that says what the person does next.

Destination object (used by phone tools): `{ "type": <type>, "ref": <string ≤ 200> }`
- `type`: `hangup`, `extension`, `ring_group`, `queue`, `voicemail`, `ivr` (menu), `announcement`, `time_condition` (phone hours)
- `ref`: the extension number for `extension` and `voicemail` (voicemail must be on for that extension), otherwise the ID (from `get_phone_setup`); empty for `hangup`
- The destination must exist in this account. Forwarding to outside numbers is not available through MCP.

---

## Server info

`initialize` returns `serverInfo` with the name `the-y-crm`, the title `The-Y CRM`, the version of the installation (for example `1.1.68.4`), the website `https://the-y.at/en/crm/mcp/` and the logo. It also returns short `instructions` for the AI: start with `get_account_overview`, never ask for passwords, save replies as drafts, and only send when the person wants it and the connection allows it.

## Hints (tool annotations)

| Shown here as | `readOnlyHint` | `destructiveHint` | `idempotentHint` | `openWorldHint` | Tools |
|---|---|---|---|---|---|
| read-only | `true` | – | – | `false` | `get_account_overview`, `get_webchat_embed_code`, `list_reply_templates`, `get_phone_setup`, `search_contacts`, `list_pipelines`, `list_conversations`, `get_conversation` |
| write | `false` | `false` | `false` | `false` | `create_reply_template`, `create_extension`, `create_ring_group`, `add_phone_number`, `import_contacts`, `create_deal`, `create_task`, `create_reply_draft` |
| write, destructive, repeatable | `false` | `true` | `true` | `false` | `set_ring_group_members`, `set_number_destination` |
| write, destructive | `false` | `true` | `false` | `false` | `set_phone_hours` |
| write, destructive, repeatable, external | `false` | `true` | `true` | `true` | `setup_mailbox`, `set_webchat_enabled` |
| write, external | `false` | `false` | `false` | `true` | `create_phone_line` |
| write, destructive, external | `false` | `true` | `false` | `true` | `send_reply` |

Each tool's annotations also carry its `title`. Claude asks before every call of a destructive tool, even if the connection may do it. Other AI programs decide for themselves.

---

## Overview

### `get_account_overview` · Account overview · read-only
Shows what is set up in the account: channels (mailboxes, WhatsApp, SMS, web chat) with their state, phone line, what is still missing (with a link into the app) and what this connection may do. Usually the first call.
Parameters: none.

Returns:
- `account`: `name`.
- `channels`: one row per channel with `id`, `type`, `address`, `label`, `brand_id`, `enabled`, `state` and `reason`.
  - `state` `live` = connected and working; `incomplete` = access details missing or not checked yet; `off` = switched off.
  - Web chat: `live` only once a real visitor message has arrived (sample conversations don't count). Before that `ready` with reason `no_real_message_yet`: switched on, embed code not in use yet. `off` when switched off.
- `webchat`: `enabled`, `live`, `received_real_message`.
- `phone`: `line_state`, `enabled_numbers`, `last_incoming_call_at` (or `null`).
- `any_channel_live`: `true` if at least one channel is `live` or the phone line works.
- `areas` (`channels`, `phone`, `hours`, `team`, `company`, `branding`): `state` `running` · `missing` · `off` · `done`, a `missing` sentence and a `link`.
- `open_items`: what to do next, each with a link.
- `this_key`: name, scopes and the tools of this connection.
- `explain`: a sentence that explains the states.

---

## Channels: email and web chat

### `setup_mailbox` · Set up mailbox · write, destructive, repeatable, external
Creates an email mailbox, or completes an existing one with the same address, and enters the IMAP/SMTP servers. For an existing mailbox, the server details you send overwrite the saved ones. Gmail servers are set automatically. No password: the person enters it via the returned link; for Microsoft they click "Sign in with Microsoft".

| Parameter | Type | Required | Description |
|---|---|---|---|
| `address` | string (≤ 254) | yes | Email address of the mailbox |
| `label` | string (≤ 100) | | Label |
| `brand_id` | string | | ID of the brand (from `get_account_overview` or `get_webchat_embed_code`); empty = default brand |
| `from_name` | string (≤ 120) | | Sender name in emails |
| `imap_host` | string (≤ 253) | | Server for incoming mail, e.g. `imap.example.com` |
| `imap_port` | integer 1–65535 | | Port |
| `imap_user` | string (≤ 254) | | User name at the mail server (usually the email address) |
| `imap_ssl` | boolean | | Encrypted (SSL/TLS) |
| `smtp_host` | string (≤ 253) | | Server for outgoing mail, e.g. `smtp.example.com` |
| `smtp_port` | integer 1–65535 | | Port |
| `smtp_user` | string (≤ 254) | | User name at the mail server |
| `smtp_ssl` | boolean | | Encrypted (SSL/TLS) |

Returns the mailbox (`id`, `address`, `label`, `created`), the detected `provider`, `server_saved`, the saved servers, `password_set`, a `human_step` sentence and a `link` into the app.

### `get_webchat_embed_code` · Web chat: embed code · read-only
Returns the embed code for the website and the link to the chat page.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `brand_id` | string | | With several brands: which brand |

Returns `enabled`, `embed_code`, `chat_page_url`, `brands` and a `human_step` sentence.

### `set_webchat_enabled` · Switch web chat on/off · write, destructive, repeatable, external
Switches the web chat on or off and then returns the embed code. "Off" takes the chat off the website.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `enabled` | boolean | yes | `true` = on, `false` = off |

---

## Phone system

### `get_phone_setup` · Phone overview · read-only
Extensions, ring groups, phone hours, numbers with destination, lines (`password_set` instead of passwords) and their state, queues, menus, announcements.
Parameters: none.

### `create_extension` · Create extension · write
Creates an extension, optionally for a team member and with voicemail. The device password is generated automatically and only shown in the app.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `number` | string | yes | Extension, 2–6 digits (no emergency numbers, not already in use) |
| `name` | string (≤ 60) | | Name shown on calls |
| `user_id` | string | | ID of a team member of this account |
| `voicemail` | boolean | | Switch on voicemail |

### `create_ring_group` · Create ring group · write
Several extensions ring at once.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `name` | string (≤ 60) | yes | Name of the ring group |
| `members` | array of strings (≤ 100) | | Existing extensions, e.g. `["100", "101"]` |

### `set_ring_group_members` · Ring group: set members · write, destructive, repeatable
Replaces the members of a ring group with the given extensions.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `ring_group_id` | string | yes | ID of the ring group (from `get_phone_setup`) |
| `members` | array of strings (≤ 100) | yes | Existing extensions |

### `set_phone_hours` · Set phone hours · write, destructive
Sets when the phone is "open" and where calls go during and outside those times. With `phone_hours_id` the existing phone hours are replaced. Without it, every call creates new phone hours.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `phone_hours_id` | string | | ID of the phone hours to change; empty = create new |
| `name` | string (≤ 60) | | Name, e.g. "Opening hours" |
| `ranges` | array (1–14) | yes | Times in which "open" applies, each `{ from_day, to_day?, from, to }`: days `mon`…`sun`, times `HH:MM`, end after start |
| `open_destination` | destination | yes | Where calls go during the times |
| `closed_destination` | destination | yes | Where calls go outside the times |

The hours only take effect once a phone number points to them (`set_number_destination` with type `time_condition`).

### `add_phone_number` · Add phone number · write
Adds a number from your phone provider. Its calls hang up until you set a destination with `set_number_destination`.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `number` | string (≤ 24) | yes | Number with country code: `+` and 8–15 digits, e.g. `+4312345678` (spaces and hyphens are removed) |
| `label` | string (≤ 60) | | Label |

### `set_number_destination` · Phone number: set destination · write, destructive, repeatable
Sets where calls to a number go.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `number` | string (≤ 40) | yes | Phone number (`+…`) or its ID |
| `destination` | destination | yes | Destination for calls |

### `create_phone_line` · Create phone line · write, external
Creates the SIP line to the phone provider, without password. It only becomes active once the person enters the SIP password in the app (link in the response). Every call creates a new line. Server checks are the same as in the app (public host names only).

| Parameter | Type | Required | Description |
|---|---|---|---|
| `name` | string (≤ 60) | yes | Name of the line, e.g. the provider's name |
| `server` | string (≤ 253) | yes | SIP server of the provider (host name) |
| `port` | integer 1–65535 | | Port (default 5060) |
| `username` | string (≤ 120) | yes | SIP user name from the provider |
| `provider` | string (≤ 60) | | Provider |
| `register` | boolean | | Register with the provider (default: yes) |
| `country_code` | string (1–4 digits) | | Country code without `+`, e.g. `43` |

---

## Contacts, deals and tasks

### `search_contacts` · Search contacts · read-only
Searches contacts by name, email or phone number; without a search term: the newest.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `query` | string (≤ 200) | | Search term |
| `limit` | integer 1–50 | | At most how many (default 20) |
| `offset` | integer ≥ 0 | | How many to skip |

### `import_contacts` · Create/import contacts · write
Creates up to 500 contacts at once. Same email address = the existing contact is completed (empty fields only). Marketing consent is never set; automations do not start.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `contacts` | array (1–500) of `{ first_name?, last_name?, email?, phone? }` | yes | At least one field per contact. `first_name` and `last_name` ≤ 100, `email` ≤ 254, `phone` ≤ 40 characters |
| `dry_run` | boolean | | `true` = preview only, save nothing |

Returns:
- **Import:** `dry_run: false`, `created`, `updated`, `skipped`, `created_ids` (IDs of the new contacts) and `problems` (skipped rows, at most 50).
- **Preview** (`dry_run: true`): `saved: false`, the counts `would_create`, `duplicates` and `errors`, and `rows` with `{ row, outcome, reason?, key? }` per row. `outcome` is `would_create`, `duplicate` (a contact with this email exists and would be completed) or `error` (would be skipped). Plus a `note` sentence. **No IDs**, because nothing is saved.

### `list_pipelines` · View pipelines · read-only
Lists the pipelines with their stages (for `create_deal`).
Parameters: none.

### `create_deal` · Create deal · write
Without a stage: first stage of the default pipeline.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `title` | string (≤ 300) | yes | Title of the deal |
| `value` | number ≥ 0 | | Value |
| `currency` | string (3 letters) | | Currency, e.g. `EUR` (default `EUR`) |
| `pipeline_id` | string | | From `list_pipelines` |
| `stage_id` | string | | From `list_pipelines` |
| `contact_id` | string | | ID of a contact |
| `company_id` | string | | ID of a company |
| `expected_close_date` | string | | `YYYY-MM-DD` |

### `create_task` · Create task · write

| Parameter | Type | Required | Description |
|---|---|---|---|
| `title` | string (≤ 300) | yes | Title of the task |
| `description` | string (≤ 10,000) | | Description |
| `due_at` | string (≤ 40) | | `YYYY-MM-DD` or date with time |
| `priority` | `low` · `normal` · `high` · `urgent` | | Priority |
| `kind` | `todo` · `callback` · `call` · `email` · `follow_up` | | Kind of task |
| `assignee_user_id` | string | | ID of a team member |
| `contact_id`, `company_id`, `deal_id`, `conversation_id` | string | | Link the task to a record |

---

## Inbox and reply templates

### `list_conversations` · View inbox · read-only
Conversations, newest first.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `status` | `open` · `snoozed` · `closed` | | Filter by status |
| `channel` | `email` · `whatsapp` · `sms` · `webchat` · `messenger` · `instagram` · `call` · `fb_comment` · `ig_comment` | | Filter by channel |
| `limit` | integer 1–50 | | At most how many (default 20) |
| `offset` | integer ≥ 0 | | How many to skip |

### `get_conversation` · Read conversation · read-only
A conversation with its latest messages; internal notes are marked. Each message text has at most 4,000 characters (longer ones are cut and marked `text_truncated`). All texts together have at most 60,000 characters; if that is not enough, the oldest messages are left out and `older_omitted` says how many.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `conversation_id` | string | yes | From `list_conversations` |
| `messages` | integer 1–100 | | How many of the latest messages (default 30) |

In a test access (an account that sends nothing outside, such as a test account or a reviewer access), the answer also has `delivery_reason: "test_access_simulated"` and a `delivery_note` sentence. Every sent reply there that was only saved carries `delivery_reason` too. Real accounts get neither field.

### `create_reply_draft` · Create reply draft · write
Saves a reply draft as an internal note in the conversation, headed "NOT sent yet". It is never sent to the customer; the team reviews and sends it.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `conversation_id` | string | yes | Conversation |
| `text` | string (≤ 8,000) | yes | Text of the draft |

If `send_reply` later sends exactly this text in the same conversation over the same connection, the draft is marked as done: its first line becomes "… done: used for the reply via send_reply". The note and its text stay. Notes written by people and drafts from other connections are never changed.

### `send_reply` · Send reply · write, destructive, external
Sends a reply in the conversation over the same channel, with the same rules as "Send" in the inbox: 24-hour window, consent, monthly limit of the Free plan. Needs the separate permission "May send messages to customers".

| Parameter | Type | Required | Description |
|---|---|---|---|
| `conversation_id` | string | yes | Conversation |
| `text` | string (≤ 8,000) | | Text of the reply |
| `template` | `{ name, language?, params? }` | | WhatsApp template, only outside the 24-hour window. `name` ≤ 512, `language` ≤ 15, `params` ≤ 20 items of ≤ 600 characters |

Returns `message_id`, `conversation_id`, `delivery` and, if there is one, `delivery_info`.
- **Test access:** nothing leaves the CRM. `delivery` is `simulated` (`noop` in the web chat), and the answer adds `delivery_reason: "test_access_simulated"` and a `delivery_note` sentence. Real accounts get neither field.
- **Drafts:** if an open draft from this connection in this conversation has exactly the sent text, it is marked as done, and `drafts_marked_done` lists its ID. This happens for text replies only (not templates), and not when the delivery failed.

### `list_reply_templates` · View reply templates · read-only
Lists the text snippets for replies in the inbox, in pages. Each text is shortened to 500 characters (`body_truncated: true`).

| Parameter | Type | Required | Description |
|---|---|---|---|
| `limit` | integer 1–100 | | At most how many (default 50) |
| `offset` | integer ≥ 0 | | How many to skip |

Returns `count`, `total`, `offset`, `has_more` and `templates`.

### `create_reply_template` · Create reply template · write

| Parameter | Type | Required | Description |
|---|---|---|---|
| `title` | string (≤ 200) | yes | Title |
| `body` | string (≤ 10,000) | yes | Text |
| `shortcut` | string (≤ 40) | | Shortcut of letters, digits, `_` or `-`, e.g. `thanks` |
| `locale` | `de` · `en` · `tr` | | Language |
| `brand_id` | string | | Brand |
