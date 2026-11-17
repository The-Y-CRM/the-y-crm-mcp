# Raw requests with curl

## 1. Discovery (no credentials)

```bash
curl -s -D - -o /dev/null -X POST https://crm.the-y.at/api/mcp -H 'content-type: application/json' -d '{}' | grep -i www-authenticate
# WWW-Authenticate: Bearer realm="the-y-crm", resource_metadata="https://crm.the-y.at/.well-known/oauth-protected-resource/api/mcp"

curl -s https://crm.the-y.at/.well-known/oauth-protected-resource/api/mcp
curl -s https://crm.the-y.at/.well-known/oauth-authorization-server
```

## 2. With an API key

```bash
# read the key without echoing it
read -rs THE_Y_KEY

curl -s https://crm.the-y.at/api/mcp \
  -H "Authorization: Bearer $THE_Y_KEY" \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-06-18","capabilities":{},"clientInfo":{"name":"curl","version":"1.0"}}}'

curl -s https://crm.the-y.at/api/mcp \
  -H "Authorization: Bearer $THE_Y_KEY" \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -H 'MCP-Protocol-Version: 2025-06-18' \
  -d '{"jsonrpc":"2.0","id":2,"method":"tools/list"}'
```

The server is stateless: no session ID is needed between requests. Older protocol versions receive the single response as a short SSE stream (`event: message` / `data: {...}`).

## 3. Call a tool

```bash
curl -s https://crm.the-y.at/api/mcp \
  -H "Authorization: Bearer $THE_Y_KEY" \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -H 'MCP-Protocol-Version: 2025-06-18' \
  -d '{"jsonrpc":"2.0","id":3,"method":"tools/call","params":{"name":"get_account_overview","arguments":{}}}'
```

Every call appears in the account's change log under the key's name.
