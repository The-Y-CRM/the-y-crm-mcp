# MCP Inspector (command line)

Uses an API key (Settings → Integrations → Connect AI (MCP) → Create key). Keep the key out of shell history and repositories, e.g. read it into a variable first.

List the tools this key may use:

```bash
npx @modelcontextprotocol/inspector --cli https://crm.the-y.at/api/mcp --method tools/list --header "Authorization: Bearer $THE_Y_KEY"
```

Call a read-only tool:

```bash
npx @modelcontextprotocol/inspector --cli https://crm.the-y.at/api/mcp --method tools/call --tool-name get_account_overview --header "Authorization: Bearer $THE_Y_KEY"
```

Call a tool with arguments:

```bash
npx @modelcontextprotocol/inspector --cli https://crm.the-y.at/api/mcp --method tools/call --tool-name search_contacts --tool-arg query=anna --tool-arg limit=5 --header "Authorization: Bearer $THE_Y_KEY"
```

Note: the token endpoint does not send CORS headers, so the OAuth flow in a browser-based client may not work. Use an API key there, or a desktop client such as Claude Code.
