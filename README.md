# Dataprius API v2 – Documentation

REST API to work with files, folders, users and permissions of a Dataprius account.

- **Base URL:** `https://api.v2.dataprius.com`
- **Interactive docs (Postman):** https://api.dataprius.org
- **OpenAPI specification (download):**
  https://raw.githubusercontent.com/Dataprius/dataprius-api-docs/main/openapi.json
- **PHP example:** https://github.com/Dataprius/dataprius-api-php

## Get your API keys

In the Dataprius desktop app: **Start → Configuration → API Keys**.
Any user of the account (administrator or colleague) can request their keys
(`clientId` and `clientSecret`) from that window.

More information about the API and how to get your keys:
- English: https://dataprius.com/en/api-developers-zone-version-2-0.html
- Español: https://dataprius.com/api-desarrolladores.html

## Authentication

The API uses OAuth2. First get an `access_token` with your keys, then send it
as Bearer in every other request.

### 1. Get a token

Send `base64(clientId:clientSecret)` in the `Authorization` header:

```bash
curl -X GET "https://api.v2.dataprius.com/oauth/token" \
  -H "Authorization: Basic <base64(clientId:clientSecret)>"
```

Example response:

```json
{
  "token_type": "bearer",
  "created_at": 1687944522153,
  "access_token": "a492954ebad2187063fa5e09854bfd8222ad3fc3",
  "refresh_token": "405bc5a2bcb5c026afed160085952ef37bb2d709",
  "expires_in": 1687951722153
}
```

| Field | Meaning |
|---|---|
| `created_at` | Moment the token was issued (Unix time, **milliseconds**). |
| `expires_in` | Moment the `access_token` expires (Unix time, **milliseconds**). It is an absolute timestamp, not a duration: `created_at` + 2 hours. |
| `refresh_token` | Single-use token to get a new `access_token` without sending the keys again (see below). |

### 2. Use the token

Send `access_token` as Bearer in all other requests:

```bash
curl -X POST "https://api.v2.dataprius.com/folders/list/0" \
  -H "Authorization: Bearer <access_token>" \
  -H "Content-Type: application/json" \
  -d '{"Page": "1"}'
```

## Refresh an expired token

The `access_token` is valid for **2 hours** (`expires_in`). Before or after it
expires, you can get a new one with the `refresh_token`. Use the same
endpoint, the same Basic header and a JSON body:

```bash
curl -X GET "https://api.v2.dataprius.com/oauth/token" \
  -H "Authorization: Basic <base64(clientId:clientSecret)>" \
  -H "Content-Type: application/json" \
  -d '{"grant_type": "refresh_token", "refresh_token": "<refresh_token>"}'
```

The response has the same format as the first token (new `access_token`,
new `refresh_token` and new `expires_in`).

Important:
- **Each `refresh_token` can be used only once.** When you use it, both the
  old `refresh_token` and the old `access_token` stop working. Always save
  the new pair returned.
- If the `refresh_token` is expired or was already used, the API returns
  `401` with `error_code: REFRESCH_EXPIRED`. In that case request a new
  token with your keys (step 1 of Authentication).
- Send `grant_type` and `refresh_token` in the JSON **body** (not in the
  query string).

## Main endpoints

| Area | Endpoints |
|---|---|
| Auth | `GET /oauth/token` |
| Folders | `POST /folders/list/{id}`, `POST /folders/files/{id}`, `GET /folders/info/{id}`, `POST /folders/getpath`, `POST /folders/create/{id}`, `POST /folders/createpath`, `PATCH /folders/rename/{id}`, `DELETE /folders/delete/{id}` |
| Files | `POST /files/upload`, `GET /files/info/{id}`, `GET /files/download/{id}`, `PATCH /files/rename/{id}`, `DELETE /files/delete/{id}` |
| Search | `POST /search` |
| Users | `GET /users`, `POST /user`, `GET/PATCH/DELETE /user/{id}`, `POST /user/invite` |
| Permissions | `PATCH /permissions` |

Notes:
- `/folders/list/0` is the root folder.
- Lists are paginated with the `Page` body parameter.
- Deleted files and folders go to the account's recycle bin.

## About Dataprius

Dataprius is a cloud intranet for businesses: secure storage (GDPR),
real-time collaboration without sync conflicts, user permissions, secure
email and automatic backups. More at https://dataprius.com
