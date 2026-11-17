# Authentication (for client developers)

As of The-Y CRM 1.1.68.4. Cloud base URL: `https://crm.the-y.at`. Self-hosted installations serve the same paths under their own address.
Most users don't need this page: Claude, Claude Code and ChatGPT handle the sign-in by themselves.

Two ways to authenticate, with the same checks behind them:

1. **OAuth 2.1, sign-in by click.** For programs that support MCP authorization.
2. **API key.** Send `Authorization: Bearer <key>` or `X-API-Key: <key>` with every request. Keys are created in the app under Settings → Integrations → Connect AI (MCP) → Create key.

---

## Discovery

A request without credentials gets `401`:

```http
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Bearer realm="the-y-crm", resource_metadata="https://crm.the-y.at/.well-known/oauth-protected-resource/api/mcp"
```

If a token or key was sent but is not valid, the header also contains `error="invalid_token"`.

**Protected resource metadata** (RFC 9728), at `/.well-known/oauth-protected-resource/api/mcp` and `/.well-known/oauth-protected-resource`:

```json
{
  "resource": "https://crm.the-y.at/api/mcp",
  "authorization_servers": ["https://crm.the-y.at"],
  "scopes_supported": ["setup:read", "contacts:read", "conversations:read", "deals:read", "users:read", "notes:write"],
  "bearer_methods_supported": ["header"],
  "resource_name": "The-Y CRM",
  "resource_documentation": "https://the-y.at/en/crm/mcp/",
  "resource_policy_uri": "https://the-y.at/en/datenschutz/#mcp",
  "resource_tos_uri": "https://the-y.at/en/nutzungsbedingungen/"
}
```

**Authorization server metadata** (RFC 8414), at `/.well-known/oauth-authorization-server`:

| Field | Value |
|---|---|
| `issuer` | `https://crm.the-y.at` |
| `authorization_endpoint` | `https://crm.the-y.at/oauth/authorize` |
| `token_endpoint` | `https://crm.the-y.at/api/oauth/token` |
| `registration_endpoint` | `https://crm.the-y.at/api/oauth/register` |
| `revocation_endpoint` | `https://crm.the-y.at/api/oauth/revoke` |
| `response_types_supported` | `["code"]` |
| `response_modes_supported` | `["query"]` |
| `grant_types_supported` | `["authorization_code", "refresh_token"]` |
| `code_challenge_methods_supported` | `["S256"]` |
| `token_endpoint_auth_methods_supported` | `["none"]` (public clients only) |
| `revocation_endpoint_auth_methods_supported` | `["none"]` |
| `client_id_metadata_document_supported` | `true` |
| `authorization_response_iss_parameter_supported` | `true` |
| `scopes_supported` | the 12 scopes listed under [Scopes](#scopes) |
| `service_documentation` | `https://the-y.at/en/crm/mcp/` |
| `op_policy_uri` | `https://the-y.at/en/datenschutz/#mcp` |
| `op_tos_uri` | `https://the-y.at/en/nutzungsbedingungen/` |

Both metadata documents are public (`Access-Control-Allow-Origin: *`) and may be cached for 5 minutes.

---

## Client identification

- **Client ID Metadata Document (preferred).** `client_id` is an `https` URL with a path, without query, fragment or credentials. We fetch it from public addresses only, without redirects, within 5 seconds. The answer must be status 200, JSON and at most 5 KB; the whole document is read (since 1.1.68.3; before, only the start was read, and sign-in from claude.ai failed). It must contain `client_id` (equal to the URL), `client_name` and `redirect_uris` (1–20), no secret, and allow `token_endpoint_auth_method` `none`. It is cached for one hour and fetched fresh at each authorization.
- **Dynamic Client Registration** (RFC 7591). Public clients only; no client secret is ever issued. 1–10 redirect URIs. At most 10 registrations per IP address and hour. Registering alone grants no access. Registered clients without a connection are deleted after 30 days without use.
- **Redirect URIs:** `https`, or loopback (`http://localhost/…`, `http://127.0.0.1/…`, `http://[::1]/…`) on any port for native clients such as Claude Code. The consent page shows the program's name and, in large letters, the redirect host. It warns about loopback redirects, and it warns when a self-registered program calls itself Claude or ChatGPT but redirects somewhere else.

---

## Authorization

- PKCE with `S256` is required (`plain` is rejected).
- `resource` (RFC 8707) must be `https://crm.the-y.at/api/mcp` (or its origin).
- The `scope` you request is only a preselection. The person decides on the consent page: **Read and write drafts** (default), **Set up**, and the separate tick box **May send messages to customers**. "Set up" is preselected only if the client asks for write or connect scopes; sending is never preselected. Only people with the right to manage integrations can approve.
- Every redirect carries `state` and `iss` (RFC 9207). Errors that can be sent back safely go to the checked redirect URI; an unknown client or a mismatching redirect only gets an error page (no open redirects). If the person declines, the redirect carries `error=access_denied`.

## Token endpoint

- Accepts `application/x-www-form-urlencoded` (and JSON), at most 16 KB. A client secret (in the body or as `Authorization: Basic`) is rejected with `invalid_client`.
- Authorization code: valid 2 minutes, single use, bound to client, redirect URI, PKCE verifier and resource. A second redemption revokes the connection.
- Access token: valid 1 hour.
- Refresh token: rotates on every use; the old one is invalid afterwards.
  - **Grace period (since 1.1.67):** if the same client sends the refresh token it has just used again within 60 seconds, for example because two requests renewed at the same moment, it gets exactly the same new token pair again. This only works while the new refresh token has not been used yet.
  - Any other reuse of an old refresh token revokes the whole connection.
- The connection (grant) is valid for 90 days; then the person connects again.
- Error codes follow RFC 6749 (`invalid_grant`, `invalid_request`, `invalid_target`, `unsupported_grant_type`, `invalid_client`). Too many requests: `slow_down` with status `429` and `Retry-After`.
- Responses carry `Cache-Control: no-store`. The token endpoint sends no CORS headers.
- Access tokens only work at `/api/mcp`; the REST API does not accept them.

## Revocation

`POST /api/oauth/revoke` (RFC 7009). Revoking a refresh token ends the whole connection; revoking an access token ends only that token. A valid request always gets `200`, also for an unknown token. People can also disconnect a program in the app: Settings → Integrations → Connected AI programs → Disconnect.

---

## Scopes

| Scope | Allows |
|---|---|
| `setup:read` | read the setup: overview, channels, phone, templates, embed code (never passwords or keys) |
| `setup:write` | change internal setup: extensions, ring groups, phone hours, phone numbers and their destinations, reply templates |
| `channels:connect` | connect channels: mailbox servers, web chat on/off, phone line |
| `contacts:read` / `contacts:write` | search / create and import contacts |
| `conversations:read` | read the inbox |
| `deals:read` / `deals:write` | read pipelines / create deals |
| `tasks:write` | create tasks |
| `users:read` | read team members (for assignments) |
| `notes:write` | save reply drafts as internal notes |
| `messages:send` | send messages to customers |

Each tool also requires the person behind the connection to have the matching right in the app today.

---

## Request rules

- `POST /api/mcp` only; `GET` and `DELETE` return `405`. Request bodies up to 1 MB.
- Responses are JSON; older protocol versions get the single response as a short SSE stream.
- Requests with an `Origin` header are accepted only from the server's own address, `claude.ai`, `claude.com`, `chatgpt.com`, `chat.openai.com` and `localhost`; other origins get `403`. Server-to-server requests without `Origin` are fine.

## Limits per IP address

| Endpoint | Limit per IP address |
|---|---|
| `/api/mcp` | 120 failed sign-ins (`401`) a minute; signed-in calls don't count here |
| `/oauth/authorize` | 60 a minute |
| `/api/oauth/token` | 60 a minute |
| `/api/oauth/revoke` | 60 a minute |
| `/api/oauth/register` | 10 an hour |

Requests from Claude's address range (`160.79.104.0/21`) get ten times these limits at `/api/mcp`, the token, revocation and registration endpoints, because many users share these addresses. Self-hosted installations set the ranges with `MCP_TRUSTED_EGRESS`.
Limits per key or connection and per account: see [README → Limits](../README.md#limits). Responses carry `X-RateLimit-*` headers for the account limit.
