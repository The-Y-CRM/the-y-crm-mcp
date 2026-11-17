<p align="center"><img src="assets/the-y-crm-icon-512.png" width="96" height="96" alt="The-Y CRM"></p>

# The-Y CRM MCP server – documentation

> **This repository contains documentation only. The-Y CRM is proprietary software (hosted service, self-hostable Docker images); it is not open source.**
> There is no server code here. The MCP server runs as part of The-Y CRM at `https://crm.the-y.at/api/mcp`.

The-Y CRM is a shared inbox for phone calls, WhatsApp, email and web chat, with a phone system in the browser, contacts, deals, tasks and invoices. Its **MCP server** lets an AI program such as Claude or ChatGPT set up and run an account for you: mailbox, web chat, phone extensions, ring groups and phone hours, contacts, deals, tasks and reply drafts. You decide what the AI may do, and every call is in the account's log.

- **Server URL:** `https://crm.the-y.at/api/mcp` (self-hosted: `https://<your-address>/api/mcp`)
- **Transport:** Streamable HTTP. Specification 2026-07-28, older clients from 2025-03-26 work too.
- **Sign-in:** OAuth 2.1 by click (PKCE S256, Client ID Metadata Documents and dynamic client registration), or an API key in a header
- **Listed in:** the official MCP Registry as `at.the-y/crm`, and the Claude directory as a community connector: https://claude.ai/directory/the-y-crm
- **Website and full guide:** https://the-y.at/en/crm/mcp/?src=github-1117-eintrag
- **As of:** The-Y CRM 1.1.68.4 (16 November 2026)
- **Status:** alpha. Features and limits still change; known issues are public: https://the-y.at/en/known-issues/?src=github-1117-eintrag

[Deutsch weiter unten](#deutsch)

---

## Contents

- [Connect your AI program](#connect-your-ai-program)
- [Permissions](#permissions)
- [The 23 tools](#the-23-tools)
- [What the AI can't do](#what-the-ai-cant-do)
- [Limits](#limits)
- [Privacy and data](#privacy-and-data)
- [Self-hosting](#self-hosting)
- [Pricing](#pricing)
- [Where it is listed](#where-it-is-listed)
- [Changes since 1.1.66](#changes-since-1166)
- [Support](#support)
- [License of this documentation](#license-of-this-documentation)
- More: [docs/tools.md](docs/tools.md) (all parameters) · [docs/authentication.md](docs/authentication.md) (OAuth details for client developers) · [examples/](examples/)

---

## Connect your AI program

You need a The-Y CRM account (the free plan works, up to 3 users): https://crm.the-y.at/register?src=github-1117-eintrag
Only people who may manage integrations in the account (for example admins) can connect an AI program.

### Claude (claude.ai and Claude Desktop)

**Easiest: from the Claude directory.** Open [The-Y CRM in the Claude directory](https://claude.ai/directory/the-y-crm), or open the directory in Claude and search for "The-Y CRM", and connect it. Our sign-in page opens: sign in to The-Y CRM, choose what the AI may do, click **Allow**.

**Or add it yourself as a custom connector.** You need this for a self-hosted installation with your own address, or if you prefer to add the connector yourself:

1. In Claude, go to **Customize → Connectors**, click **+ Add**, then **Add custom connector**.
2. Name: `The-Y CRM`. Remote MCP server URL: `https://crm.the-y.at/api/mcp` (self-hosted: your own address). Click **Continue**, check the sign-in settings Claude detected and click **Continue** again.
3. Authentication: **Sign in now**. OAuth client: **Use Claude's published identity (Recommended)**. Click **Add**.
4. Our sign-in page opens: sign in to The-Y CRM, choose what the AI may do, click **Allow**.

Then, in a chat, click **+** at the bottom left, open **Connectors**, switch on The-Y CRM and ask, for example: *"Show me what is still missing in my CRM."*

On Claude Team and Enterprise, the people who manage your Claude organization add the connector first (**Organization settings → Connectors → Add → Custom → Web**). After that, everyone connects it under **Customize → Connectors** with their own sign-in. According to Anthropic, custom connectors work on the Free, Pro, Max, Ultra, Team and Enterprise plans; on Free you can add one.

### Claude Code

```bash
claude mcp add --transport http the-y-crm https://crm.the-y.at/api/mcp
```
Then type `/mcp`, choose `the-y-crm` and sign in in the browser. Our sign-in page tells you that the program runs on your computer (localhost); that is expected for Claude Code. More: [examples/claude-code.md](examples/claude-code.md)

### ChatGPT

1. In ChatGPT on the web, open **Plugins**, click **+**, then **Add custom MCP server**.
2. Name: `The-Y CRM`. Under **Connection** enter `https://crm.the-y.at/api/mcp`. Authentication: **OAuth**.
3. Tick **I understand and want to continue**, click **Create as a plugin**.
4. Our sign-in page opens: sign in to The-Y CRM, choose what the AI may do, click **Allow**. ChatGPT then shows the tools it found.
5. Start a new chat, type `@` and choose The-Y CRM.

According to OpenAI, custom MCP servers with full access are available in ChatGPT Business, Enterprise and Edu, on the web only. In Business workspaces only admins can add them; other plans are limited. Menu names and plans of AI programs change; please check with the provider.

### Other programs (VS Code, Cursor, MCP Inspector, your own client)

- Programs that support OAuth for remote MCP servers sign in by click. Configuration examples: [examples/vscode-mcp.json](examples/vscode-mcp.json), [examples/cursor-mcp.json](examples/cursor-mcp.json).
- Otherwise send an API key with every request: `Authorization: Bearer <key>` (or `X-API-Key: <key>`). Create the key in The-Y CRM under **Settings → Integrations → Connect AI (MCP) → Create key**, choose the level, then **Create key for the AI**. It is shown only once and expires after 90 days unless you switch that off.
- Raw requests with `curl`: [examples/curl.md](examples/curl.md). MCP Inspector: [examples/mcp-inspector.md](examples/mcp-inspector.md).

---

## Permissions

You choose when you connect:

| Level | What the AI may do | Tools |
|---|---|---|
| **Read and write drafts** (default, recommended) | sees the setup, contacts, deals and the inbox; saves reply drafts as internal notes. Changes nothing, sends nothing | 9 |
| **Set up** | additionally connects a mailbox and the web chat, sets up the phone system, creates templates, contacts, deals and tasks | 22 |
| **May send messages to customers** | separate tick box, never preselected | +1 (`send_reply`) |

- A key or connection can never do more than the person who created it may do in the app. If that person is deactivated, access stops.
- Connecting a channel and sending are permissions of their own. With an API key you can switch each permission on or off individually.
- A connection by click is valid for 90 days. Disconnect any time: **Settings → Integrations → Connected AI programs → Disconnect**.

---

## The 23 tools

The AI only sees the tools its connection may use. 8 tools only read, 15 change something. The hints are the MCP tool annotations that AI programs read:

- **read-only:** `readOnlyHint: true`. Changes nothing.
- **write:** changes data in the account. Unless more is listed: `destructiveHint: false`, `idempotentHint: false`, `openWorldHint: false`.
- **destructive:** `destructiveHint: true`. It overwrites or replaces something, or reaches a customer. Claude asks before each call, even if the connection may do it; other AI programs decide for themselves.
- **repeatable:** `idempotentHint: true`. Calling it again with the same input changes nothing more.
- **external:** `openWorldHint: true`. It has an effect outside the account.

Full parameter list and exact values: [docs/tools.md](docs/tools.md).

| Tool | What it does | Permission | Hint |
|---|---|---|---|
| `get_account_overview` | Channels and their state, phone line, what is still missing (with a link into the app), what this connection may do. A channel is `live` when it is connected and working; the web chat only once a real visitor message has arrived, before that it is `ready` | Read and write drafts | read-only |
| `setup_mailbox` | Creates an email mailbox and enters the IMAP/SMTP servers (Gmail filled in automatically). If a mailbox with this address exists, its server details are overwritten. **No password:** you enter it via a link | Set up (connect channels) | write, destructive, repeatable, external |
| `get_webchat_embed_code` | Embed code for your website and the link to the chat page | Read and write drafts | read-only |
| `set_webchat_enabled` | Switches the web chat on or off and returns the embed code. "Off" takes the chat off your website | Set up (connect channels) | write, destructive, repeatable, external |
| `list_reply_templates` | Lists text snippets for replies, 50 per call (at most 100); each text shortened to 500 characters | Read and write drafts | read-only |
| `create_reply_template` | Creates a text snippet (title, text, shortcut, language, brand) | Set up | write |
| `get_phone_setup` | Extensions, ring groups, phone hours, numbers with destination, lines (without passwords), queues, menus, announcements | Read and write drafts | read-only |
| `create_extension` | Extension with 2–6 digits, optionally for a team member and with voicemail; the device password is generated and only shown in the app | Set up | write |
| `create_ring_group` | Ring group: several extensions ring at once | Set up | write |
| `set_ring_group_members` | Replaces the members of a ring group | Set up | write, destructive, repeatable |
| `set_phone_hours` | When the phone is "open" and where calls go during and outside those times. Without `phone_hours_id` every call creates new phone hours | Set up | write, destructive |
| `add_phone_number` | Adds a number from your phone provider (`+` and 8–15 digits) | Set up | write |
| `set_number_destination` | Where calls to a number go: extension, ring group, queue, voicemail, menu, announcement, phone hours or hang up | Set up | write, destructive, repeatable |
| `create_phone_line` | Creates the SIP line to your provider, **without password**; active once you enter it in the app. Every call creates a new line | Set up (connect channels) | write, external |
| `search_contacts` | Searches contacts by name, email or phone | Read and write drafts | read-only |
| `import_contacts` | Creates up to 500 contacts at once; same email = the existing contact is completed. Preview with `dry_run`: saves nothing and returns no IDs, only the result per row. Never sets marketing consent | Set up | write |
| `list_pipelines` | Pipelines and their stages | Read and write drafts | read-only |
| `create_deal` | Deal with title, value, stage, contact, company, expected close | Set up | write |
| `create_task` | Task with due date, assignee, link to contact, company, deal or conversation | Set up | write |
| `list_conversations` | Inbox, newest first, filter by status and channel | Read and write drafts | read-only |
| `get_conversation` | One conversation with its latest messages; internal notes are marked. At most 4,000 characters per message and 60,000 in total; in very long conversations the oldest are left out | Read and write drafts | read-only |
| `create_reply_draft` | Saves a reply as an internal note headed "NOT sent yet"; never reaches the customer. If `send_reply` later sends exactly this text, the draft is marked as done | Read and write drafts | write (internal note only) |
| `send_reply` | Sends a reply over the conversation's channel, same rules as "Send" in the inbox (24-hour window, consent, monthly limit of the Free plan). In a test access nothing is delivered, and the answer says so | May send messages (separate tick box) | write, destructive, external |

Tool titles and descriptions are returned in the language of the person who connected (English, German or Turkish).

---

## What the AI can't do

- **Delete:** there is no tool for deleting anything.
- **Make calls:** it sets up the phone system but never calls anyone.
- **Forward calls to outside numbers:** only in the app.
- **Connect WhatsApp:** that needs your own sign-in with Meta, in the app.
- **See or accept passwords:** for a mailbox or phone line the AI gives you a link, and you enter the password yourself.
- **Set consent:** imported contacts never get marketing consent from the AI.

---

## Limits

Per key or connection, on top of the account's API limits:

| Kind of tool | Per minute | Per hour |
|---|---|---|
| Read | 60 | 1,200 |
| Change | 30 | 600 |
| Connect a channel | 10 | 30 |
| Send | 10 | 60 |

- **Account limits** (shared with the REST API): Free 60 requests a minute and 1,000 a day, Cloud Pro 300 a minute; self-hosted none unless you set them. Messages the AI sends count toward the Free plan's 1,000 outgoing messages a month; drafts don't.
- **Per IP address:** only failed sign-ins count, at most 120 a minute. Signed-in calls are limited per key or connection and per account. Claude's address range (`160.79.104.0/21`) gets ten times the limit, because many users share these addresses.
- **Answer sizes:** `list_reply_templates` returns 50 templates per call (at most 100), each text at most 500 characters. `get_conversation` returns at most 4,000 characters per message and 60,000 in total; if that is not enough, the oldest messages are left out and the answer says how many.

---

## Privacy and data

- Your CRM data stays on our server in Vienna, Austria (EU), or on your own server if you self-host.
- What the AI reads through the MCP server (depending on the permissions you choose) goes to the provider of your AI program, such as Anthropic or OpenAI, under that provider's terms.
- Every call is in the account's change log: time, key or connection, tool, result, record IDs and the IP address of the calling program. No message text, no names, addresses or phone numbers.
- No tool accepts a password, and responses never contain passwords, keys or tokens.
- Privacy policy, section "Connecting AI programs (MCP)": https://the-y.at/en/datenschutz/#mcp
- Security: https://the-y.at/en/crm/security/?src=github-1117-eintrag · Vulnerability reports: https://the-y.at/.well-known/security.txt

---

## Self-hosting

The-Y CRM also runs on your own server with Docker images (proprietary licence). The MCP server is built in: use `https://<your-address>/api/mcp`, nothing extra to set up. Sign-in by click works against your own installation. For Claude and ChatGPT, your server must be reachable from the internet over HTTPS, and for Claude it must be able to reach claude.ai itself.
Two optional settings in the `.env` (since 1.1.67): `MCP_TRUSTED_EGRESS` (address ranges of large AI providers that get ten times the per-address limit; default `160.79.104.0/21`, `off` = none) and `DOMAIN_PROOFS` (proof files under `/.well-known/<name>` if an AI directory asks for one).
Guide: https://the-y.at/en/docs/self-host/?src=github-1117-eintrag

---

## Pricing

The MCP server is included in every plan at no extra cost.

- **Free:** up to 3 users.
- **Cloud Pro:** €20/month per company, all users and brands (alpha and beta price).
- Self-hosted licences: https://the-y.at/en/crm/costs/?src=github-1117-eintrag

---

## Where it is listed

- **Official MCP Registry:** `at.the-y/crm`, version 1.1.68.4. The [server.json](server.json) in this repository is the published entry. Look it up: https://registry.modelcontextprotocol.io/v0.1/servers?search=at.the-y/crm
- **Claude directory:** community connector, https://claude.ai/directory/the-y-crm

---

## Changes since 1.1.66

- **1.1.68.4** (16 Nov 2026): in a test access, `send_reply` and `get_conversation` say that nothing is delivered (`delivery_reason: "test_access_simulated"` and a sentence in `delivery_note`); real accounts are unchanged. A draft is marked as done once `send_reply` sends exactly its text. In `get_account_overview` the web chat counts as `live` only from the first real visitor message (before that `ready`), in the list and in the summary alike. `import_contacts` with `dry_run` returns no IDs.
- **1.1.68.3** (16 Nov 2026): sign-in from claude.ai and Claude Desktop works. The server now reads Client ID Metadata Documents in full (up to 5 KB); before, it read only the start, and Claude's sign-in failed.
- **1.1.67** (14 Nov 2026): tool hints, so Claude asks before sending, setting up a mailbox and switching the web chat; renewing the sign-in twice at the same moment no longer ends the connection (60-second grace period); the per-address limit counts failed sign-ins only; answer sizes (reply templates in pages, long messages shortened); the server introduces itself with name, website and logo.

Full release notes: https://the-y.at/en/release-notes/?src=github-1117-eintrag

---

## Support

- **Questions** about connecting or using the MCP server: please ask in [Discussions](https://github.com/The-Y-CRM/the-y-crm-mcp/discussions), category **Q&A**.
- **Errors in this documentation:** please open an [issue](https://github.com/The-Y-CRM/the-y-crm-mcp/issues).
- **Your account:** please email office@the-y.at. Never post keys, passwords or customer data in Discussions or Issues.
- Website: https://the-y.at/en/crm/?src=github-1117-eintrag

---

## License of this documentation

The text and examples in this repository are licensed under **Creative Commons Attribution 4.0 International (CC BY 4.0)**, see [LICENSE](LICENSE). The logo in `assets/` is not covered.
This licence covers the documentation only. It grants no rights to The-Y CRM software, which is proprietary. "The-Y" and the logo are marks of The-Y e.U., Vienna. Claude is a trademark of Anthropic, ChatGPT of OpenAI.

---

## Deutsch

> **Dieses Repository enthält nur Dokumentation. The-Y CRM ist proprietäre Software (gehosteter Dienst, selbst hostbare Docker-Images) und nicht Open Source.**

The-Y CRM ist ein gemeinsamer Posteingang für Anrufe, WhatsApp, E-Mail und Web-Chat, mit Telefonanlage im Browser, Kontakten, Verkaufschancen, Aufgaben und Rechnungen. Über den **MCP-Server** richtet eine KI wie Claude oder ChatGPT dein Konto ein und arbeitet mit: Postfach, Web-Chat, Durchwahlen, Gruppenruf, Telefon-Zeiten, Kontakte, Verkaufschancen, Aufgaben und Antwort-Entwürfe. Du legst fest, was die KI darf. Jeder Aufruf steht im Änderungsprotokoll.

- **Adresse:** `https://crm.the-y.at/api/mcp` (selbst gehostet: deine Adresse + `/api/mcp`)
- **Claude:** am einfachsten über das Claude-Verzeichnis (Community-Connector): https://claude.ai/directory/the-y-crm → „Verbinden“ → im The-Y CRM anmelden, wählen, was die KI darf, „Erlauben“. Oder selbst als eigenen Connector mit der Adresse oben.
- **Offizielle MCP Registry:** `at.the-y/crm`
- **Anmeldung:** per Klick (OAuth) oder mit API-Schlüssel (`Authorization: Bearer <Schlüssel>`)
- **Rechte:** „Lesen und Entwürfe schreiben“ (Standard), „Einrichten“, und „Darf Nachrichten an Kunden senden“ als eigener Haken
- **Kann nicht:** löschen, anrufen, nach außen weiterleiten, WhatsApp verbinden, Passwörter sehen, Einwilligungen setzen
- **Daten:** auf unserem Server in Wien (EU) oder auf deinem eigenen. Was die KI liest, geht an den Anbieter des KI-Programms. Datenschutz: https://the-y.at/datenschutz/#mcp
- **Preis:** In jedem Tarif enthalten. Free bis 3 Nutzer, Cloud Pro 20 €/Monat pro Unternehmen, alle Nutzer und Marken (Alpha- und Beta-Preis).
- **Anleitung auf Deutsch:** https://the-y.at/crm/mcp/?src=github-1117-eintrag
- **Stand:** The-Y CRM 1.1.68.4, Alpha
- **Fragen:** gern in den [Discussions](https://github.com/The-Y-CRM/the-y-crm-mcp/discussions) (Kategorie Q&A). Fehler in dieser Doku bitte als [Issue](https://github.com/The-Y-CRM/the-y-crm-mcp/issues). Zu deinem Konto: office@the-y.at
