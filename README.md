<p align="center">
  <a href="https://pushizo.com"><img src="https://raw.githubusercontent.com/pushizo/.github/main/profile/assets/mark.svg" alt="Pushizo" width="56" height="56"></a>
</p>

<h1 align="center">Pushizo API — OpenAPI</h1>

<p align="center">
  <b>English</b> · <a href="README.tr.md">Türkçe</a>
</p>

<p align="center">
  <a href="https://docs.pushizo.com/docs/api"><img src="https://img.shields.io/badge/docs-API_reference-5b4bdb" alt="API reference"></a>
  <a href="https://api.pushizo.com/docs"><img src="https://img.shields.io/badge/try_it-api.pushizo.com%2Fdocs-b44fc4" alt="Interactive docs"></a>
  <img src="https://img.shields.io/badge/OpenAPI-3.1-16130f" alt="OpenAPI 3.1">
</p>

The public contract of the Pushizo REST API: push notifications (web, iOS, Android, Huawei), email, SMS and
WhatsApp, in-app messages, users and subscriptions, segments, templates, campaigns, journeys, live chat
conversations, webhooks, analytics and more.

| File | Contents |
|------|----------|
| [`openapi.yaml`](openapi.yaml) | OpenAPI 3.1 schema of every public operation (`/v1` server API and `/sdk/v1` client endpoints) |
| [`postman_collection.json`](postman_collection.json) | Postman collection with one request per operation (`baseUrl`, `apiKey`, `appId` variables) |

The schema is also served live at `https://api.pushizo.com/openapi.json`, and the interactive reference (with
"Try it") lives at [api.pushizo.com/docs](https://api.pushizo.com/docs).

## Quick start

1. Create an API key in the dashboard: **Developer → API Keys** (keys look like `hs_live_…`).
2. Send your first notification to your test devices:

```bash
curl -X POST https://api.pushizo.com/v1/notifications \
  -H "Authorization: Bearer $PUSHIZO_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: first-notification-1" \
  -d '{
    "app_id": "app_…",
    "target": { "test_devices_only": true },
    "headings": { "en": "Hello", "tr": "Merhaba" },
    "contents": { "en": "Pushizo is working", "tr": "Pushizo çalışıyor" },
    "url": "https://example.com"
  }'
# 202 → { "id": "msg_…", "status": "queued", "recipients_estimate": 1, "external_id_missing": [] }
```

## Use the schema

**Postman / Insomnia / Bruno:** import `postman_collection.json` (or `openapi.yaml`), then set `apiKey` and `appId`.

**Generate a typed client** — official SDKs already exist for most languages (below), but you can generate one:

```bash
# TypeScript types
npx openapi-typescript https://raw.githubusercontent.com/pushizo/pushizo-openapi/main/openapi.yaml -o pushizo.d.ts

# Any language (Kotlin, Swift, Rust, …)
npx @openapitools/openapi-generator-cli generate -i openapi.yaml -g rust -o ./pushizo-rust
```

## Conventions

| Topic | Rule |
|-------|------|
| Base URL | `https://api.pushizo.com` (data-residency cells return `421 HS_WRONG_CELL` with the right `details.api_url`) |
| Authentication | `Authorization: Bearer hs_live_…`; scoped keys: `notifications:write`, `users:read`, `users:write`, `events:write`, `segments:read`, `analytics:read`, `mcp:write`, `admin` |
| Format | JSON, `snake_case` fields, ISO 8601 UTC timestamps, prefixed ULID ids (`app_`, `usr_`, `sub_`, `msg_`, `seg_`, `cmp_`, `jrn_`, …) |
| Localized text | Objects keyed by language: `{ "en": "…", "tr": "…" }` (falls back to `en`, then the first language) |
| Errors | `{ "error": { "code": "HS_…", "message", "details", "request_id", "doc_url" } }`; every response carries `x-request-id` |
| Rate limits | `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`; `429 HS_RATE_LIMITED` with `Retry-After` |
| Idempotency | `Idempotency-Key` header on `POST`; results are kept for 24 hours |
| Pagination | Cursor based: `limit`, `cursor` → `next_cursor`, `has_more` |
| Updates | `PATCH` is merge-patch (only sent fields change, `null` clears) |
| Versioning | `/v1` only receives additive changes; breaking changes ship as a new version |
| Webhooks | `Pushizo-Signature: t=<unix>,v1=<hex>`, `v1 = HMAC-SHA256(secret, t + "." + raw_body)`, 5 minute tolerance |

Details: [authentication](https://docs.pushizo.com/docs/api/authentication) ·
[rate limits](https://docs.pushizo.com/docs/api/rate-limits) · [idempotency](https://docs.pushizo.com/docs/api/idempotency) ·
[pagination](https://docs.pushizo.com/docs/api/pagination) · [versioning](https://docs.pushizo.com/docs/api/versioning) ·
[error catalog](https://docs.pushizo.com/docs/errors) · [webhooks](https://docs.pushizo.com/docs/webhooks).

## Official SDKs

| Server | Client |
|--------|--------|
| [Node.js](https://github.com/pushizo/pushizo-node) · [PHP](https://github.com/pushizo/pushizo-php) · [Python](https://github.com/pushizo/pushizo-python) · [Go](https://github.com/pushizo/pushizo-go) · [Java](https://github.com/pushizo/pushizo-java) · [.NET](https://github.com/pushizo/pushizo-dotnet) · [Ruby](https://github.com/pushizo/pushizo-ruby) | [Web](https://github.com/pushizo/pushizo-web) · [iOS](https://github.com/pushizo/pushizo-ios) · [Android](https://github.com/pushizo/pushizo-android) · [Flutter](https://github.com/pushizo/pushizo-flutter) · [React Native](https://github.com/pushizo/pushizo-react-native) · [Capacitor](https://github.com/pushizo/pushizo-capacitor) · [Cordova](https://github.com/pushizo/pushizo-cordova) · [.NET MAUI](https://github.com/pushizo/pushizo-maui) · [Unity](https://github.com/pushizo/pushizo-unity) |

AI clients can use the same API through the [Pushizo MCP server](https://github.com/pushizo/pushizo-mcp)
(`https://api.pushizo.com/mcp`).

## Support

- Documentation: [docs.pushizo.com/docs/api](https://docs.pushizo.com/docs/api) (Turkish) · changelog: [API changelog](https://docs.pushizo.com/docs/api/changelog)
- Schema problems: [Issues](https://github.com/pushizo/pushizo-openapi/issues)
- Security: [security@pushizo.com](mailto:security@pushizo.com) — please do not open public issues
- Help: [support@pushizo.com](mailto:support@pushizo.com)

This repository is generated from the Pushizo main repository on every release; please open an issue instead of a
pull request. The schema and the collection are released under the MIT License ([LICENSE](LICENSE)).
