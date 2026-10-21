<p align="center">
  <a href="https://pushizo.com"><img src="https://raw.githubusercontent.com/pushizo/.github/main/profile/assets/mark.svg" alt="Pushizo" width="56" height="56"></a>
</p>

<h1 align="center">Pushizo API — OpenAPI</h1>

<p align="center">
  <a href="README.md">English</a> · <b>Türkçe</b>
</p>

<p align="center">
  <a href="https://docs.pushizo.com/docs/api"><img src="https://img.shields.io/badge/doküman-API_referansı-5b4bdb" alt="API referansı"></a>
  <a href="https://api.pushizo.com/docs"><img src="https://img.shields.io/badge/dene-api.pushizo.com%2Fdocs-b44fc4" alt="Etkileşimli doküman"></a>
  <img src="https://img.shields.io/badge/OpenAPI-3.1-16130f" alt="OpenAPI 3.1">
</p>

Pushizo REST API'sinin herkese açık sözleşmesi: push bildirimleri (web, iOS, Android, Huawei), e-posta, SMS ve
WhatsApp, in-app mesajlar, kullanıcılar ve abonelikler, segmentler, şablonlar, kampanyalar, journey'ler, canlı sohbet
konuşmaları, webhook'lar, analitik ve daha fazlası.

| Dosya | İçerik |
|-------|--------|
| [`openapi.yaml`](openapi.yaml) | Tüm public işlemlerin OpenAPI 3.1 şeması (`/v1` sunucu API'si ve `/sdk/v1` istemci uçları) |
| [`postman_collection.json`](postman_collection.json) | Her işlem için bir istek içeren Postman koleksiyonu (`baseUrl`, `apiKey`, `appId` değişkenleri) |

Şema canlı olarak `https://api.pushizo.com/openapi.json` adresinde de sunulur; "Dene" düğmeli etkileşimli referans:
[api.pushizo.com/docs](https://api.pushizo.com/docs).

## Hızlı başlangıç

1. Panelde bir API anahtarı oluşturun: **Geliştirici → API Anahtarları** (anahtarlar `hs_live_…` biçimindedir).
2. Test cihazlarınıza ilk bildirimi gönderin:

```bash
curl -X POST https://api.pushizo.com/v1/notifications \
  -H "Authorization: Bearer $PUSHIZO_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: ilk-bildirim-1" \
  -d '{
    "app_id": "app_…",
    "target": { "test_devices_only": true },
    "headings": { "tr": "Merhaba", "en": "Hello" },
    "contents": { "tr": "Pushizo çalışıyor", "en": "Pushizo is working" },
    "url": "https://example.com"
  }'
# 202 → { "id": "msg_…", "status": "queued", "recipients_estimate": 1, "external_id_missing": [] }
```

## Şemayı kullanma

**Postman / Insomnia / Bruno:** `postman_collection.json` (ya da `openapi.yaml`) dosyasını içe aktarın, `apiKey` ve
`appId` değişkenlerini doldurun.

**Tipli istemci üretme** — çoğu dil için resmi SDK zaten var (aşağıda); yine de kendiniz üretebilirsiniz:

```bash
# TypeScript tipleri
npx openapi-typescript https://raw.githubusercontent.com/pushizo/pushizo-openapi/main/openapi.yaml -o pushizo.d.ts

# Herhangi bir dil (Kotlin, Swift, Rust, …)
npx @openapitools/openapi-generator-cli generate -i openapi.yaml -g rust -o ./pushizo-rust
```

## Kurallar

| Konu | Kural |
|------|-------|
| Taban adres | `https://api.pushizo.com` (veri bölgesi hücreleri doğru adresi `421 HS_WRONG_CELL` + `details.api_url` ile bildirir) |
| Kimlik doğrulama | `Authorization: Bearer hs_live_…`; kapsamlı anahtarlar: `notifications:write`, `users:read`, `users:write`, `events:write`, `segments:read`, `analytics:read`, `mcp:write`, `admin` |
| Biçim | JSON, `snake_case` alanlar, ISO 8601 UTC zamanlar, ön ekli ULID kimlikler (`app_`, `usr_`, `sub_`, `msg_`, `seg_`, `cmp_`, `jrn_`, …) |
| Çok dilli metin | Dile göre nesne: `{ "tr": "…", "en": "…" }` (yoksa `en`, o da yoksa ilk dil) |
| Hatalar | `{ "error": { "code": "HS_…", "message", "details", "request_id", "doc_url" } }`; her yanıtta `x-request-id` |
| Hız sınırı | `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`; `429 HS_RATE_LIMITED` ve `Retry-After` |
| Idempotency | `POST` isteklerinde `Idempotency-Key` başlığı; sonuç 24 saat saklanır |
| Sayfalama | Cursor tabanlı: `limit`, `cursor` → `next_cursor`, `has_more` |
| Güncelleme | `PATCH` merge-patch'tir (yalnızca gönderilen alanlar değişir, `null` siler) |
| Sürümleme | `/v1` yalnızca geriye uyumlu eklemeler alır; kırıcı değişiklik yeni sürümle gelir |
| Webhook'lar | `Pushizo-Signature: t=<unix>,v1=<hex>`, `v1 = HMAC-SHA256(secret, t + "." + ham_gövde)`, 5 dakika tolerans |

Ayrıntılar: [kimlik doğrulama](https://docs.pushizo.com/docs/api/authentication) ·
[hız sınırları](https://docs.pushizo.com/docs/api/rate-limits) · [idempotency](https://docs.pushizo.com/docs/api/idempotency) ·
[sayfalama](https://docs.pushizo.com/docs/api/pagination) · [sürümleme](https://docs.pushizo.com/docs/api/versioning) ·
[hata kataloğu](https://docs.pushizo.com/docs/errors) · [webhook'lar](https://docs.pushizo.com/docs/webhooks).

## Resmi SDK'lar

| Sunucu | İstemci |
|--------|---------|
| [Node.js](https://github.com/pushizo/pushizo-node) · [PHP](https://github.com/pushizo/pushizo-php) · [Python](https://github.com/pushizo/pushizo-python) · [Go](https://github.com/pushizo/pushizo-go) · [Java](https://github.com/pushizo/pushizo-java) · [.NET](https://github.com/pushizo/pushizo-dotnet) · [Ruby](https://github.com/pushizo/pushizo-ruby) | [Web](https://github.com/pushizo/pushizo-web) · [iOS](https://github.com/pushizo/pushizo-ios) · [Android](https://github.com/pushizo/pushizo-android) · [Flutter](https://github.com/pushizo/pushizo-flutter) · [React Native](https://github.com/pushizo/pushizo-react-native) · [Capacitor](https://github.com/pushizo/pushizo-capacitor) · [Cordova](https://github.com/pushizo/pushizo-cordova) · [.NET MAUI](https://github.com/pushizo/pushizo-maui) · [Unity](https://github.com/pushizo/pushizo-unity) |

Yapay zekâ istemcileri aynı API'yi [Pushizo MCP sunucusu](https://github.com/pushizo/pushizo-mcp) üzerinden
kullanabilir (`https://api.pushizo.com/mcp`).

## Destek

- Dokümantasyon: [docs.pushizo.com/docs/api](https://docs.pushizo.com/docs/api) · değişiklikler: [API değişiklik günlüğü](https://docs.pushizo.com/docs/api/changelog)
- Şema sorunları: [Issues](https://github.com/pushizo/pushizo-openapi/issues)
- Güvenlik: [security@pushizo.com](mailto:security@pushizo.com) — herkese açık issue açmayın
- Destek: [support@pushizo.com](mailto:support@pushizo.com)

Bu depo her sürümde Pushizo ana deposundan üretilir; pull request yerine issue açın. Şema ve koleksiyon MIT lisansıyla
dağıtılır ([LICENSE](LICENSE)).
