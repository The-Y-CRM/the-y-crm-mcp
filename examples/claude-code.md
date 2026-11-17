# Claude Code

## Sign in by click (recommended)

```bash
claude mcp add --transport http the-y-crm https://crm.the-y.at/api/mcp
```

Then, inside Claude Code:

```
/mcp
```

Choose `the-y-crm` → authenticate. Your browser opens The-Y CRM: sign in, choose what the AI may do (**Read and write drafts** or **Set up**, plus the separate tick box **May send messages to customers**) and click **Allow**. The sign-in page points out that the program runs on your computer (localhost); that is expected for Claude Code.

## With an API key

Create a key in The-Y CRM: Settings → Integrations → Connect AI (MCP) → Create key. It is shown only once.

```bash
claude mcp add --transport http the-y-crm https://crm.the-y.at/api/mcp --header "Authorization: Bearer <your key>"
```

Never commit the key to a repository.

## Self-hosted

Replace `https://crm.the-y.at` with your own address, e.g. `https://crm.example.com/api/mcp`.

## First prompt

```
Show me what is still missing in my CRM.
```

More prompts: [prompts.md](prompts.md)
