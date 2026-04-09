# productivity-mcp Design Spec

> MCP server providing full Microsoft 365 + Graph API access across Lucid Labs (primary) and OAK AI (secondary) accounts.

## Overview

- **Repo:** `LucidLabsAU/productivity-mcp` (private)
- **Runtime:** Node.js + TypeScript
- **MCP SDK:** `@modelcontextprotocol/sdk`
- **Replaces:** `OAK-AI-AU/mcp-outlook`

## Accounts

| Account | Slug | Auth Method | Tenant ID | Default |
|---------|------|-------------|-----------|---------|
| Lucid Labs | `lucid` | Client credentials (app + Key Vault) | `38fcc012-5a88-4df9-b912-44ad0ee73268` | **Yes** |
| OAK AI | `oakai` | Delegated device code (MSAL) | `ef3af0e7-d027-4e31-bb9b-90078fbdc135` | No |

Every tool accepts an optional `account` parameter (`"lucid"` or `"oakai"`). Omitting defaults to `lucid`.

- Lucid Labs uses `/users/{userId}/` endpoints (user ID: `82281768-cc2c-446f-89d5-ad2fcc9885c6`)
- OAK AI uses `/me/` endpoints

## App Registration

### Lucid Labs — New App: `productivity-mcp`

**Type:** Application (client credentials)
**Audience:** `AzureADMyOrg` (single-tenant, Lucid Labs only)
**Client secret:** Stored in Key Vault `lucidhub-kv-dev` as `productivity-mcp-client-secret`

#### Application Permissions (admin-consented)

| Permission | Scope | Purpose |
|------------|-------|---------|
| `Mail.ReadWrite` | Mail | Read/write/send mail |
| `Mail.Send` | Mail | Send mail on behalf of user |
| `Calendars.ReadWrite` | Calendar | Read/write calendar events |
| `Chat.Read.All` | Teams | Read Teams chats |
| `ChatMessage.Read.All` | Teams | Read chat messages |
| `ChannelMessage.Read.All` | Teams | Read channel messages |
| `Tasks.Read.All` | Planner | Read Planner tasks |
| `Tasks.ReadWrite.All` | Planner | Create/update Planner tasks |
| `Files.ReadWrite.All` | OneDrive | Read/write/upload files |
| `Sites.ReadWrite.All` | SharePoint | Read/write SharePoint sites and libraries |
| `User.Read.All` | Users | User lookups |
| `Presence.Read.All` | Activity | User presence/availability |
| `Reports.Read.All` | Activity | Usage/activity reports |
| `OnlineMeetings.Read.All` | Activity | Meeting attendance reports |
| `CallRecords.Read.All` | Activity | Call/meeting join records (webhook subscription for real-time; attendance reports work without) |
| `ChannelMessage.Send` | Teams | Send messages to Teams channels |

### OAK AI — Existing App: `mcp-outlook` (`e247b41c`)

**Type:** Delegated (device code flow)
**Changes required:** Add scopes to match Lucid capabilities:

| Scope | Status |
|-------|--------|
| `Mail.Read` | Existing |
| `Mail.ReadBasic` | Existing |
| `Mail.ReadWrite` | Existing |
| `Mail.Send` | Existing |
| `Calendars.ReadWrite` | **New** |
| `Chat.Read` | **New** |
| `ChatMessage.Read` | **New** |
| `Tasks.Read` | **New** |
| `Tasks.ReadWrite` | **New** |
| `Files.ReadWrite` | **New** |
| `Sites.Read.All` | **New** |
| `User.Read` | **New** |
| `Presence.Read` | **New** |
| `OnlineMeetingArtifact.Read.All` | **New** |
| `offline_access` | Existing |

After scope changes, user must re-authenticate (delete `~/.productivity-mcp-oakai-token.json` and re-run device code).

## Conditional Access Considerations

Lucid Labs has 18 active CA policies. Impact:

| Policy | Impact | Mitigation |
|--------|--------|------------|
| `LucidAzure-01-MFA-AllUsers` | Affects delegated flows | Client credentials bypass user-level MFA |
| `LucidAzure-02-DeviceCompliance` | Affects delegated flows | Client credentials not subject to device compliance |
| `Block legacy authentication` | Blocks non-modern auth | We use OAuth2/OIDC only — no impact |

**Client credentials for Lucid Labs avoids all user-targeted CA policies.** This is the primary reason for the hybrid auth approach.

OAK AI has fewer CA restrictions. Device code flow works fine there (MFA cached via token).

## MCP Tools

### Mail (6 tools)

| Tool | Method | Endpoint | Read/Write |
|------|--------|----------|------------|
| `mail_list_folders` | GET | `/users/{id}/mailFolders` | Read |
| `mail_list_messages` | GET | `/users/{id}/messages` or `/users/{id}/mailFolders/{folderId}/messages` | Read |
| `mail_read_message` | GET | `/users/{id}/messages/{messageId}` | Read |
| `mail_search` | GET | `/users/{id}/messages?$search=` | Read |
| `mail_create_draft` | POST | `/users/{id}/messages` | Write |
| `mail_send_draft` | POST | `/users/{id}/messages/{messageId}/send` | Write |

**Parameters carried from mcp-outlook:** folder filtering, pagination (nextLink), unread filter, attachment listing/download, HTML/text body, to/cc/bcc, importance.

### Calendar (4 tools)

| Tool | Method | Endpoint | Read/Write |
|------|--------|----------|------------|
| `calendar_list_events` | GET | `/users/{id}/calendarView` | Read |
| `calendar_get_event` | GET | `/users/{id}/events/{eventId}` | Read |
| `calendar_create_event` | POST | `/users/{id}/events` | Write |
| `calendar_respond_event` | POST | `/users/{id}/events/{eventId}/accept` (or decline/tentative) | Write |

**Parameters:** date range, select fields (subject, start, end, attendees, location, isAllDay, showAs, organiser), filter cancelled, calculate duration.

### Teams (4 tools)

| Tool | Method | Endpoint | Read/Write |
|------|--------|----------|------------|
| `teams_list_chats` | GET | `/users/{id}/chats` | Read |
| `teams_read_chat` | GET | `/users/{id}/chats/{chatId}/messages` | Read |
| `teams_search_messages` | GET | `/users/{id}/chats/{chatId}/messages?$filter=` | Read |
| `teams_send_message` | POST | `/users/{id}/chats/{chatId}/messages` | Write |

**Note:** `Chat.Read.All` and `ChatMessage.Read.All` are application permissions — they can read any user's chats. Scoped to Keith's user ID only.

### Planner (4 tools)

| Tool | Method | Endpoint | Read/Write |
|------|--------|----------|------------|
| `planner_list_plans` | GET | `/users/{id}/planner/plans` | Read |
| `planner_list_tasks` | GET | `/planner/plans/{planId}/tasks` | Read |
| `planner_create_task` | POST | `/planner/tasks` | Write |
| `planner_update_task` | PATCH | `/planner/tasks/{taskId}` | Write |

**Note:** PATCH requires `If-Match` header with `@odata.etag` (optimistic concurrency). Tool must fetch current etag before update.

### OneDrive (3 tools)

| Tool | Method | Endpoint | Read/Write |
|------|--------|----------|------------|
| `onedrive_list_files` | GET | `/users/{id}/drive/root/children` or `/users/{id}/drive/root:/{path}:/children` | Read |
| `onedrive_download_file` | GET | `/users/{id}/drive/items/{itemId}/content` | Read |
| `onedrive_upload_file` | PUT | `/users/{id}/drive/root:/{path}:/content` | Write |

**Parameters:** path-based browsing, folder traversal, download to local path, upload from local path. Small files via PUT, large files via upload session.

### SharePoint (3 tools)

| Tool | Method | Endpoint | Read/Write |
|------|--------|----------|------------|
| `sharepoint_list_sites` | GET | `/sites?search=` | Read |
| `sharepoint_list_library` | GET | `/sites/{siteId}/drive/root/children` or by path | Read |
| `sharepoint_download_file` | GET | `/sites/{siteId}/drive/items/{itemId}/content` | Read |

**Note:** SharePoint document libraries are exposed as drives via Graph. Same drive API as OneDrive but scoped to site.

### Activity (3 tools)

| Tool | Method | Endpoint | Read/Write |
|------|--------|----------|------------|
| `activity_work_summary` | GET (multiple) | Aggregates calendar + email + chat + attendance | Read |
| `activity_user_presence` | GET | `/users/{id}/presence` | Read |
| `activity_meeting_attendance` | GET | `/users/{id}/onlineMeetings/{meetingId}/attendanceReports` | Read |

**`activity_work_summary`** mirrors lucid-hub's `activity.ts` pattern:
- Takes a date range
- Parallel-fetches calendar events, received emails, sent emails, Teams chats
- Cross-references calendar events with meeting attendance (who actually joined, duration, join/leave times)
- Groups by date
- Returns daily summaries: meeting count (invited vs attended), email count (received/sent), chat count, external domains contacted, meeting details with attendance

**`activity_meeting_attendance`** provides per-meeting detail:
- Attendee list with join time, leave time, duration
- Attendance status (attended, absent, etc.)
- Uses `OnlineMeetings.Read.All` + meeting join URL from calendar event

## Architecture

```
LucidLabsAU/productivity-mcp/
├── src/
│   ├── index.ts                    # MCP server bootstrap + all tool registrations
│   ├── auth/
│   │   ├── lucid.ts                # Client credentials: Key Vault secret → app token
│   │   ├── oakai.ts                # Device code: MSAL cache → user token
│   │   └── resolver.ts             # Account param → correct auth + endpoint prefix
│   ├── graph/
│   │   ├── client.ts               # Shared fetch wrapper, error handling, pagination
│   │   ├── mail.ts                 # 6 mail tools
│   │   ├── calendar.ts             # 4 calendar tools
│   │   ├── teams.ts                # 4 Teams tools
│   │   ├── planner.ts              # 4 Planner tools
│   │   ├── onedrive.ts             # 3 OneDrive tools
│   │   ├── sharepoint.ts           # 3 SharePoint tools
│   │   └── activity.ts             # 3 activity/aggregation tools
│   └── types.ts                    # Shared Graph response types
├── auth.js                         # Standalone OAK AI auth script
├── package.json
├── tsconfig.json
├── .gitignore
├── README.md
└── CLAUDE.md
```

### Key Design Decisions

1. **Single MCP server, not two.** One `node` process serves both accounts. The `account` param routes to the correct auth provider and endpoint prefix.

2. **`resolver.ts` centralises routing.** Given an account slug, it returns `{ getToken(), userPrefix }` where `userPrefix` is either `/users/82281768.../` (Lucid) or `/me/` (OAK AI).

3. **Pagination handled in `client.ts`.** All list endpoints follow `@odata.nextLink` with a configurable max-pages limit (default 5, ~500 items). Tools expose `nextLink` param for manual pagination.

4. **Activity aggregation runs parallel fetches.** `Promise.all` across calendar, email, chat, attendance. Individual failures return empty arrays (graceful degradation).

5. **Token caching:**
   - Lucid: In-memory, refreshed on expiry (client credentials tokens last ~1hr)
   - OAK AI: File-based at `~/.productivity-mcp-oakai-token.json`, MSAL handles refresh

## Wiring

### Claude Code — Global workspace MCP config

File: `~/.claude/projects/-Users-keithoak-Documents-GitHub/.mcp.json`

```json
{
  "mcpServers": {
    "productivity-mcp": {
      "command": "node",
      "args": ["/Users/keithoak/Documents/GitHub/LucidLabsAU/productivity-mcp/dist/index.js"],
      "env": {
        "AZURE_CONFIG_DIR": "/Users/keithoak/.azure-lucid"
      }
    }
  }
}
```

This makes the MCP available in all sessions under `~/Documents/GitHub/`.

### VS Code (optional, later)

Can add to `LucidLabsAU/.mcp.json` for Copilot access.

## Dependencies

```json
{
  "@modelcontextprotocol/sdk": "^1.27.1",
  "@azure/msal-node": "^5.0.6",
  "@azure/identity": "^4.0.0",
  "@azure/keyvault-secrets": "^4.8.0",
  "zod": "^4.3.6"
}
```

No `node-fetch` — use native `fetch` (Node 22+, which fnm provides).

## Setup Steps (one-time)

1. Create app registration `productivity-mcp` in Lucid Labs Entra ID
2. Add application permissions listed above
3. Admin-consent the permissions
4. Create client secret, store in `lucidhub-kv-dev` as `productivity-mcp-client-secret`
5. Update OAK AI app (`e247b41c`) scopes to include calendar, Teams, Planner, OneDrive, SharePoint, presence
6. Create GitHub repo `LucidLabsAU/productivity-mcp` (private)
7. Build and wire into Claude Code

## Migration

- `OAK-AI-AU/mcp-outlook` archived after productivity-mcp is operational
- `cdsb-dispute` project MCP config updated to point to new server
- Corrupted session wiring restored via global MCP config

## Security

- Client secret never in code — fetched from Key Vault at runtime
- `AZURE_CONFIG_DIR` env var scopes `DefaultAzureCredential` to Lucid Labs
- OAK AI token cache file permissions: user-only (`0600`)
- No secrets in repo, `.env` files gitignored
- Application permissions scoped to minimum required per tool category
