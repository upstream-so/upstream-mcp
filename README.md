# Upstream.so — MCP Server

The official **Model Context Protocol** server for [Upstream.so][home], the cloud live-streaming platform. Point an AI assistant at one URL and it can run your channels: start and stop streams, reorder playback queues, search your media library, plan schedules, and manage multistream destinations.

It is a **remote** server — nothing to install, no npm package, no local process. It mirrors the public [Upstream API][api] one-to-one, so every tool runs the same validation, ownership rules and rate limits as the REST endpoint behind it.

```
  ┌───────────────┐   MCP (Streamable HTTP)   ┌────────────────────┐
  │  your AI       │ ────────────────────────► │ studio.upstream.so │
  │  (Claude, ...) │ ◄──── tool results ────── │        /mcp        │
  └───────────────┘                            └─────────┬──────────┘
                                                         │ same code path
                                                         ▼
                                              ┌────────────────────┐
                                              │  Upstream API v1    │
                                              └────────────────────┘
```

## Endpoint

| | |
| --------- | ---------------------------------- |
| URL       | `https://studio.upstream.so/mcp`   |
| Transport | Streamable HTTP                    |
| Auth      | `Authorization: Bearer <API key>`; OAuth rollout in progress |

## Authentication

OAuth rollout is in progress. Use an API key for now. The ChatGPT and Claude sign-in steps below apply once activation is complete.

Connect with **OAuth** when your client supports it. Add the server URL, sign in directly on `studio.upstream.so`, and review the requested access. Your AI client receives an access token, never your Upstream password. Remove its access under **Profile → Connected apps**.

For clients with custom request headers, a **Personal Access Token** remains available under **Profile → API Keys**. Create a dedicated key for each client and store it in the client's secret settings. Existing API-key connections keep working.

Both methods grant access to the supported MCP tools for your account, including actions that change or delete data. OAuth's `mcp:use` scope is not a read-only mode. Enable your client's approval prompts and begin with a read-only request.

> The legacy `X-Upstream-Api-Key` header still works. New API-key connections should send `Authorization: Bearer`.

## Connecting

### ChatGPT

1. Enable **Developer mode** under **Settings → Security and login**, if your account and workspace allow it.
2. Open **Plugins**, select the plus button, and name the connection **Upstream**.
3. Enter `https://studio.upstream.so/mcp`. Choose OAuth authentication when prompted.
4. Sign in on Upstream, review the consent screen, and approve the connection.
5. Start a new conversation and select Upstream from the tools menu.

This uses a custom MCP connection. It does not require a published ChatGPT app. See [OpenAI's connection guide](https://developers.openai.com/plugins/deploy/connect-chatgpt) for current availability and settings.

### Claude

1. Open **Customize → Connectors** and select **+ → Add custom connector**.
2. Name it **Upstream** and enter `https://studio.upstream.so/mcp`.
3. Add the connector, select **Connect**, and sign in on Upstream to review and approve access.
4. Enable Upstream from the conversation's Connectors menu.

Custom connectors depend on your plan. Team and Enterprise owners must first add the connector for their organization. See [Claude's custom connector guide](https://support.claude.com/en/articles/11175166-get-started-with-custom-connectors-using-remote-mcp).

### Clients using API keys

**Claude Code**

```bash
claude mcp add --transport http --scope user upstream \
  https://studio.upstream.so/mcp \
  --header "Authorization: Bearer YOUR_UPSTREAM_TOKEN"
```

**VS Code** — `mcp.json`

```json
{
  "servers": {
    "upstream": {
      "type": "http",
      "url": "https://studio.upstream.so/mcp",
      "headers": {
        "Authorization": "Bearer ${input:upstream-token}"
      }
    }
  }
}
```

**Kimi Code** — `~/.kimi-code/mcp.json`

```json
{
  "mcpServers": {
    "upstream": {
      "url": "https://studio.upstream.so/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_UPSTREAM_TOKEN"
      }
    }
  }
}
```

Clients that support remote Streamable HTTP and custom request headers can use the API-key examples above. Never paste a real key into a prompt, shared project configuration, support ticket, or directory submission. OAuth clients discover authentication from the server; callback compatibility depends on the client.

Verify the connection with a read-only call: ask the assistant to list your streams.

## Tools

Tools use the same account ownership, validation, and rate limits as the public API.

### Streams

| Tool | What it does |
| ---- | ------------ |
| `list_streams` | List the streams owned by the account, newest first, paginated. |
| `get_stream` | Get one stream by UUID: settings, status, tags, destinations. |
| `create_stream` | Create a stream — name, platform (`youtube`, `twitch`, `kick`, `custom`, `customsrt`), stream key. |
| `update_stream` | Partially update a stream; only provided fields change. |
| `delete_stream` | Permanently delete a stream and its configuration. |
| `start_stream` | Start broadcasting to the stream's platform. |
| `stop_stream` | Stop a running stream. This interrupts a live broadcast for viewers. |

Scheduling is part of a stream rather than a separate tool: `create_stream` and `update_stream` both take a `schedule` object covering start and end times, repeats, and continuous mode with its own run and break durations.

### Playback queues

Each stream owns queues by type: `video`, `audio`, `audio-secondary`, `external-videos`.

| Tool | What it does |
| ---- | ------------ |
| `get_stream_queue` | Get the ordered items of one queue. |
| `add_to_stream_queue` | Append media UUIDs to a queue — or `{url, name}` objects for `external-videos`. |
| `remove_from_stream_queue` | Remove one item. The media file itself is not deleted. |
| `reorder_stream_queue` | Replace a queue's playback order with a full ordered list of ids. |

### Media library

| Tool | What it does |
| ---- | ------------ |
| `list_media` | List and search the library, paginated. Filter by type, name or folder. |
| `get_media` | Get one file, including its processing state. |
| `update_media` | Update name and audio tags (title, artist, album, year, genre). |
| `delete_media` | Permanently delete a file. It also disappears from any queue referencing it. |
| `create_upload_ticket` | Mint a TUS upload ticket so the client can upload files itself. |
| `revoke_upload_tickets` | Revoke active upload tickets for the account. |

A freshly uploaded file is *processing* until `get_media` reports `is_processing: false`. It can only be queued after that.

### Folders

| Tool | What it does |
| ---- | ------------ |
| `list_folders` | List folders, optionally filtered by parent. |
| `get_folder` | Get one folder by UUID. |
| `create_folder` | Create a folder, optionally inside a parent. |
| `update_folder` | Rename a folder, or move it under another parent. |
| `delete_folder` | Delete a folder and its subfolders. Contained files move to the library root. |
| `move_files_to_folder` | Move up to 100 files into a folder, or back to the root. |

### Multistream destinations

| Tool | What it does |
| ---- | ------------ |
| `list_destinations` | List the extra platforms a stream is restreamed to. |
| `add_destination` | Add a destination — name, platform, stream key. |
| `update_destination` | Partially update a destination. |
| `remove_destination` | Remove a destination from the stream. |

### Tags and account

| Tool | What it does |
| ---- | ------------ |
| `list_tags` | List the account's tags. |
| `create_tag` | Create a tag. Names are unique per account. |
| `update_tag` | Update a tag's name, colour or icon. |
| `delete_tag` | Delete a tag. Streams keep working; they just lose it. |
| `get_account` | Get the authenticated account: id, name, email, plan and limits. |

## Uploading files

MCP carries JSON, not file bytes, so `create_upload_ticket` hands the upload back to the client: it returns a TUS endpoint and a 24-hour token, and the assistant's own shell moves the bytes.

Reuse one ticket per batch when practical. Minting a new ticket renews the same family without revoking in-flight uploads. Use `revoke_upload_tickets` to invalidate active tickets. Runnable examples in several languages live in [upstream-upload-examples][examples].

## Rate limits and errors

Tool calls share the REST API's budget of **60 requests/minute** per account. On HTTP 429, back off.

Resources belong to the token's account only: a foreign id returns 403, an unknown id returns 404. Destructive tools carry MCP annotations, so clients that render confirmations will ask before deleting or stopping something.

## Reference

- [API documentation][api] · [OpenAPI spec][openapi] · [Postman collection][postman]
- [Model Context Protocol][mcp]
- [Connection setup](https://upstream.so/mcp/) · [Publishing and directory submissions](docs/publishing.md)

## About Upstream

Upstream runs live video in the cloud, without OBS or a machine of your own left switched on:

- [24/7 live streaming][247] — loop a library as a channel that never goes down
- [Pre-recorded live streaming][prerec] — schedule finished videos to go live
- [Live Studio][studio] — browser production with up to 10 guests
- [Multistreaming][multi] — one feed to YouTube, Twitch, Kick, TikTok and more

## Licence

MIT — see [LICENSE](LICENSE).

[home]: https://upstream.so/
[api]: https://studio.upstream.so/docs/api
[openapi]: https://studio.upstream.so/docs/api.openapi
[postman]: https://studio.upstream.so/docs/api.postman
[mcp]: https://modelcontextprotocol.io/
[examples]: https://github.com/upstream-so/upstream-upload-examples
[247]: https://upstream.so/24-7-live-streaming/
[prerec]: https://upstream.so/pre-recorded-live-streaming/
[studio]: https://upstream.so/live-studio/
[multi]: https://upstream.so/multistreaming/
