# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

This repo holds **configuration and fixture data only** — there is no application
source code, build system, or test suite here. It configures an external generic
mock HTTP server (started by pointing it at `mock.json`) used to stand in for a real
backend during testing of other applications.

Currently configured to emulate **Search Ads 360** (REST API,
`google.golang.org/api/searchads360/v0`), so a Go app under test can call SA360
endpoints against `http://localhost:3333` instead of the live Google API.

## Files

- `mock.json` — top-level server config: `port`, `readTimeout`, `writeTimeout`,
  `logLevel`, and `endpoints[]`. Each endpoint entry supports:
  - `methods` — HTTP methods to match
  - `path` — exact path to match (Google "custom method" colon syntax like
    `:search` is literal, not a wildcard)
  - `status` — response status code
  - `json` — inline response body, or `jsonPath` — path to a response body file
  - `delay`, `proxy`, `allowCors` — optional extras (delay in ms, proxy passthrough,
    CORS allow-list) seen in earlier demo configs but not currently used
- `searchads360_customers.json` — response body for
  `GET /v0/customers:listAccessibleCustomers` (`ListAccessibleCustomersResponse` shape)
- `searchads360_search.json` — response body for
  `POST /v0/customers/{customerId}/searchAds360:search`
  (`SearchSearchAds360Response` shape), with rows carrying `customer` + `campaign` +
  `metrics` data

## Conventions to preserve when editing fixtures

- Match Google JSON API conventions: all `int64`-typed fields (`id`, `impressions`,
  `clicks`, `costMicros`, `totalResultsCount`, etc.) are serialized as **strings**,
  not numbers.
- The customer id is hardcoded into both `mock.json`'s search endpoint path and the
  fixture JSON bodies (`searchads360_customers.json`, `searchads360_search.json`) —
  keep all three in sync when changing it.
- Endpoint paths are matched literally; there is no documented request-body/query
  matching, so different responses for the same path require separate endpoint
  entries (e.g. a different path or method) rather than conditional logic.

## Testing changes

There's no build/test tooling in this repo. To verify a config change, start the mock
server against `mock.json` and curl the endpoint paths directly, e.g.:

```bash
curl http://localhost:3333/v0/customers:listAccessibleCustomers
curl -X POST http://localhost:3333/v0/customers/<id>/searchAds360:search -d '{"query":"..."}'
```

To exercise it from a Go client using
`google.golang.org/api/searchads360/v0`, point the service at the mock and skip
real auth:

```go
svc, err := searchads360.NewService(ctx,
    option.WithEndpoint("http://localhost:3333"),
    option.WithoutAuthentication(),
)
```
