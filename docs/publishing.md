# Publishing Upstream MCP

Publish only after the advertised connection works over public HTTPS. The setup
page is `https://upstream.so/mcp/`; the MCP endpoint is
`https://studio.upstream.so/mcp`.

## Release checks

- An unauthenticated MCP request returns `401` with OAuth discovery metadata,
  rather than a browser challenge or a CDN error.
- Protected-resource and authorization-server metadata return valid JSON.
- An owner-controlled test account can connect, approve access, list streams,
  refresh its token, and revoke the connection. Revoked tokens must stop working.
- Existing API-key connections still work.
- The setup page, README, and `server.json` describe the deployed behavior.

Do not include API keys, access tokens, account data, or private application source
in a directory submission. Use a dedicated test account for an authenticated scan.
The public repository contains connection documentation and registry metadata;
the hosted server implementation is not distributed here.

## Submission details

| Field | Value |
| --- | --- |
| Name | Upstream |
| Description | Manage live streams, playback queues, media, schedules, and multistream destinations. |
| Website | https://upstream.so/mcp/ |
| Server | https://studio.upstream.so/mcp |
| Transport | Streamable HTTP |
| Authentication | OAuth sign-in; API keys remain available for compatible clients |
| Repository | https://github.com/upstream-so/upstream-mcp |
| First request | List my Upstream streams. Do not change anything. |

## Where to submit

1. **[Official MCP Registry](https://modelcontextprotocol.io/registry/quickstart).**
   Publish `server.json` with the official publisher after validation. The existing
   `so.upstream/upstream` namespace needs proof of control over `upstream.so` through
   [DNS or HTTP authentication](https://modelcontextprotocol.io/registry/authentication).
   GitHub authentication only covers `io.github.*` namespaces; do not rename an
   existing server to avoid verification. Keep publishing credentials outside Git.
   OAuth is discovered from the running endpoint, so the remote configuration does
   not require users to supply a static Authorization header.
2. **[Glama connectors](https://glama.ai/mcp/connectors).** Use the remote connector
   submission, not the source-code hosting flow. Glama documents dynamic client
   registration, which can avoid sharing test credentials. Verify its exact OAuth
   callback against the server allowlist before submitting. Only healthy connectors
   are indexed. See [Glama's submission guide](https://glama.ai/mcp/faq).
3. **[Smithery](https://smithery.ai/new).** Its current
   [publishing guide](https://smithery.ai/docs/build/publish) requires Streamable
   HTTP and OAuth for protected remote servers. It describes Client ID Metadata
   Documents, so test that authentication flow before promising compatibility.
   Smithery can proxy requests through its gateway. Review that behavior before
   connecting a test account.
4. **[MCP.so](https://mcp.so/submit?type=server).** Submit the same canonical website,
   repository, endpoint, and description. Check for an existing listing first. Paid
   promotion is separate from ordinary submission and requires an explicit decision.

These are distribution targets, not a guarantee of backlinks or search rankings.
Record the resulting listing URLs and actual review status after submitting. The
ChatGPT and Claude app directories require separate review; an MCP directory entry
does not establish approval by either provider.

Documentation checked on 2026-10-01. Recheck each submission flow before publishing.
