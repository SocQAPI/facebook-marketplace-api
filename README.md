<!-- Generated from the SocQ API catalog and docs. Do not edit by hand. -->
# Facebook Marketplace API

[![SocQ Facebook Marketplace API](https://img.shields.io/badge/API-SocQ%20Facebook%20Marketplace%20API-F64C31)](https://socq.ai/apis/facebook-marketplace?utm_source=github&utm_medium=readme&utm_campaign=socq-facebook-marketplace-api)
[![API documentation](https://img.shields.io/badge/Docs-docs.socq.ai-111827)](https://docs.socq.ai/api-manual/facebook-marketplace/overview)
[![License: MIT](https://img.shields.io/badge/License-MIT-2563EB)](LICENSE)
[![Check examples](https://github.com/SocQAPI/facebook-marketplace-api/actions/workflows/check.yml/badge.svg)](https://github.com/SocQAPI/facebook-marketplace-api/actions/workflows/check.yml)

3 endpoints for public Facebook Marketplace data through one SocQ API key, with runnable cURL, Node.js and Python examples and a valid request body for every endpoint.

[Browse Facebook Marketplace APIs](https://socq.ai/apis/facebook-marketplace?utm_source=github&utm_medium=readme&utm_campaign=socq-facebook-marketplace-api) · [Get an API key](https://socq.ai/dashboard/api-key?utm_source=github&utm_medium=readme&utm_campaign=socq-facebook-marketplace-api) · [Documentation](https://docs.socq.ai/api-manual/facebook-marketplace/overview) · [All SocQ examples](https://github.com/SocQAPI/socq-examples)

## Endpoints

Prices are in SocQ credits; 1 credit = $0.005.

| Endpoint | publicId | What it returns | Price | Launched |
| --- | --- | --- | --- | --- |
| [Facebook Marketplace Location Search API](#facebook-marketplace-location-search-api) | `facebook-marketplace/location-search` | Find Facebook Marketplace location candidates from a city, region, neighborhood, or place-name query, returning names, place types, addresses, and coordinates in normalized records. | 0.3 credits ($0.0015) per result | 2026-07-20 |
| [Facebook Marketplace Search API](#facebook-marketplace-search-api) | `facebook-marketplace/search` | Search public Facebook Marketplace listings by keyword and coordinates, returning titles, prices, locations, images, status fields, and source URLs as normalized records. | 0.7 credits ($0.0035) per result | 2026-07-20 |
| [Facebook Marketplace Item API](#facebook-marketplace-item-api) | `facebook-marketplace/item` | Collect public Facebook Marketplace item details from known listing URLs, returning normalized listing, seller, price, location, media, attribute, delivery, and availability fields. | 0.6 credits ($0.003) per result | 2026-07-20 |

## Quick start

1. Create an API key at [socq.ai/dashboard/api-key](https://socq.ai/dashboard/api-key?utm_source=github&utm_medium=readme&utm_campaign=socq-facebook-marketplace-api).
2. Export it. The examples read only `SOCQ_API_KEY`.

```bash
git clone https://github.com/SocQAPI/facebook-marketplace-api.git
cd facebook-marketplace-api
export SOCQ_API_KEY="your-api-key"
```

3. Run any endpoint by its publicId. Without a second argument the script sends `payloads/<slug>.json`; pass a JSON file or an inline JSON object to change the input.

```bash
# Node.js 20+
node examples/node/socq.mjs facebook-marketplace/location-search

# Python 3.8+ (standard library only)
python3 examples/python/socq.py facebook-marketplace/location-search payloads/location-search.json

# cURL + jq
bash examples/curl/socq.sh facebook-marketplace/location-search '{"query":"Austin"}'
```

Each script submits the task, polls it until it succeeds or fails, follows every result page, and writes the records to `output/<slug>-<task_id>.json`. Run `npm run check` to validate the payloads and scripts offline.

Never expose `SOCQ_API_KEY` in browser code, mobile apps, public repositories, screenshots, fixtures or logs.

## How the API works

Base URL `https://api.socq.ai`. Every request sends the API key as a bearer token:

```http
Authorization: Bearer <SOCQ_API_KEY>
Content-Type: application/json
```

All endpoints are asynchronous:

```text
POST /v1/<publicId>  ->  data.task_id  ->  GET /v1/tasks/{task_id}  ->  queued | running | succeeded | failed
```

1. **Submit** the JSON body to `POST https://api.socq.ai/v1/<publicId>`. The response returns `data.task_id` immediately; it means the task was created, not that data is ready.
2. **Poll** `GET https://api.socq.ai/v1/tasks/{task_id}` until `data.status` is `succeeded` or `failed` (a failed task carries `data.error_message`).
3. **Read results.** A succeeded task returns normalized records in `data.results.items`. Use `limit` (default 50, max 500) and pass `data.results.next_cursor` as `cursor` while `data.results.has_more` is true. `fields` selects up to 50 comma-separated fields or dot paths.
4. **Raw files (optional).** `GET https://api.socq.ai/v1/tasks/{task_id}/files` lists raw JSONL exports.

```bash
curl "https://api.socq.ai/v1/tasks/$TASK_ID?limit=100" \
  -H "Authorization: Bearer $SOCQ_API_KEY"

curl "https://api.socq.ai/v1/tasks/$TASK_ID?limit=100&cursor=$NEXT_CURSOR" \
  -H "Authorization: Bearer $SOCQ_API_KEY"
```

`callback_url` (where an endpoint accepts it) makes SocQ POST to your URL when the task reaches a terminal state; keep polling as a fallback.

**Billing.** A submit may reserve credits from `results_limit` or the number of submitted inputs; when the task finishes SocQ settles the charge from the actual result count and returns unused reserved credits. `402` means insufficient credits.

**Errors.** `400` invalid body or unsupported field, `401` missing or invalid key, `402` insufficient credits, `403` IP allowlist or task owned by another account, `404` endpoint or task not found, `422` input validation failed, `429` rate or credit-window limit (respect `Retry-After`), `500` submission failed. A client-side timeout does not cancel a task: resume polling the same `task_id` instead of submitting again.

Reference: [Authentication](https://docs.socq.ai/api-manual/agent-api/authentication) · [Task status and results](https://docs.socq.ai/api-manual/agent-api/status) · [Task files](https://docs.socq.ai/api-manual/agent-api/files) · [Callbacks](https://docs.socq.ai/api-manual/agent-api/callbacks) · [Errors](https://docs.socq.ai/api-manual/agent-api/errors)

## Endpoint reference

### Facebook Marketplace Location Search API

Find Facebook Marketplace location candidates from a city, region, neighborhood, or place-name query, returning names, place types, addresses, and coordinates in normalized records.

- **Request:** `POST https://api.socq.ai/v1/facebook-marketplace/location-search`
- **publicId:** `facebook-marketplace/location-search`
- **Price:** 0.3 credits ($0.0015) per result
- **Launched:** 2026-07-20
- **Example body:** [`payloads/location-search.json`](payloads/location-search.json)

Use a city, region, or place name to find location coordinates for Marketplace Search.

This endpoint does not accept `results_limit`. A task saves every location candidate returned for the query.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `query` | string | Yes | Search query. |

```bash
curl -X POST "https://api.socq.ai/v1/facebook-marketplace/location-search" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "query": "Austin"
}'
```

Full docs: [https://docs.socq.ai/api-manual/facebook-marketplace/location-search](https://docs.socq.ai/api-manual/facebook-marketplace/location-search) · Landing page: [socq.ai/apis/facebook-marketplace/location-search](https://socq.ai/apis/facebook-marketplace/location-search?utm_source=github&utm_medium=readme&utm_campaign=socq-facebook-marketplace-api)

### Facebook Marketplace Search API

Search public Facebook Marketplace listings by keyword and coordinates, returning titles, prices, locations, images, status fields, and source URLs as normalized records.

- **Request:** `POST https://api.socq.ai/v1/facebook-marketplace/search`
- **publicId:** `facebook-marketplace/search`
- **Price:** 0.7 credits ($0.0035) per result
- **Launched:** 2026-07-20
- **Example body:** [`payloads/search.json`](payloads/search.json)

Search public listings around a geographic coordinate. Use [Marketplace Location Search](https://docs.socq.ai/api-manual/facebook-marketplace/location-search) to find coordinates when needed.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `availability` | string | No | - |
| `condition` | string | No | - |
| `date_listed` | string | No | - |
| `delivery_method` | string | No | - |
| `latitude` | number | Yes | (value -90 to 90) |
| `longitude` | number | Yes | (value -180 to 180) |
| `max_price` | number | No | (value at least 0) |
| `min_price` | number | No | (value at least 0) |
| `query` | string | Yes | Search query. |
| `radius_km` | number | No | (greater than 0) |
| `results_limit` | integer | No | (value 1 to 2000) |
| `sort_by` | string | No | - |

```bash
curl -X POST "https://api.socq.ai/v1/facebook-marketplace/search" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "query": "bike",
  "latitude": 30.2677,
  "longitude": -97.7475,
  "results_limit": 100,
  "radius_km": 65,
  "min_price": 100,
  "max_price": 500,
  "sort_by": "creation_time_descend",
  "delivery_method": "local_pickup",
  "condition": "used_good",
  "date_listed": "last_7_days",
  "availability": "available"
}'
```

Full docs: [https://docs.socq.ai/api-manual/facebook-marketplace/search](https://docs.socq.ai/api-manual/facebook-marketplace/search) · Landing page: [socq.ai/apis/facebook-marketplace/search](https://socq.ai/apis/facebook-marketplace/search?utm_source=github&utm_medium=readme&utm_campaign=socq-facebook-marketplace-api)

### Facebook Marketplace Item API

Collect public Facebook Marketplace item details from known listing URLs, returning normalized listing, seller, price, location, media, attribute, delivery, and availability fields.

- **Request:** `POST https://api.socq.ai/v1/facebook-marketplace/item`
- **publicId:** `facebook-marketplace/item`
- **Price:** 0.6 credits ($0.003) per result
- **Launched:** 2026-07-20
- **Example body:** [`payloads/item.json`](payloads/item.json)

Collect detailed listing information from one or more public Marketplace item URLs.

Each URL is processed independently and can produce at most one listing. If an item is unavailable or not found, the remaining URLs continue processing.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `urls` | array of string | Yes | Public profile or content URLs. (items at least 1) |

```bash
curl -X POST "https://api.socq.ai/v1/facebook-marketplace/item" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "urls": [
    "https://www.facebook.com/marketplace/item/978385068564936/"
  ]
}'
```

Full docs: [https://docs.socq.ai/api-manual/facebook-marketplace/item](https://docs.socq.ai/api-manual/facebook-marketplace/item) · Landing page: [socq.ai/apis/facebook-marketplace/item](https://socq.ai/apis/facebook-marketplace/item?utm_source=github&utm_medium=readme&utm_campaign=socq-facebook-marketplace-api)

## Repository contents

| Path | Purpose |
| --- | --- |
| [`examples/curl/socq.sh`](examples/curl/socq.sh) | Submit, poll and paginate with cURL and jq |
| [`examples/node/socq.mjs`](examples/node/socq.mjs) | Same workflow in Node.js 20+ (no dependencies) |
| [`examples/python/socq.py`](examples/python/socq.py) | Same workflow in Python 3.8+ (standard library) |
| [`payloads/`](payloads) | One documented request body per endpoint |
| [`endpoints.json`](endpoints.json) | Endpoint list, prices and request schemas used by the scripts and checks |
| [`fixtures/`](fixtures) | Synthetic submit and task response shapes |
| [`docs/production-notes.md`](docs/production-notes.md) | Retries, timeouts, pagination and data handling |
| [`scripts/check.mjs`](scripts/check.mjs) | Offline check: payloads, syntax and secrets |

## Responsible use and platform scope

- Use only publicly accessible Facebook Marketplace data and the fields supported by each endpoint.
- Do not use the examples to access restricted content, login-only surfaces, authentication controls or non-public personal information.
- SocQ is not an official API of the represented platform and is not affiliated with or endorsed by it. Platform names and trademarks belong to their respective owners.
- Before production use, assess the laws, platform terms, privacy obligations and retention requirements that apply to your organization and use case.
- Collect only the fields needed for a defined purpose, restrict access, set retention periods, and support correction or deletion workflows where required.

This section describes the public-data boundary; it is not legal advice.

## License

[MIT](LICENSE)
