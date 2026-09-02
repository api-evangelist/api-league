---
name: Call an API League endpoint correctly
description: Authenticate against API League, make a first call, and read the quota and error signals the platform actually returns. The prerequisite skill for every other API League skill.
api: openapi/_original/api-league-openapi.json
operations: [searchBooksAPI, randomQuoteAPI]
---

# Call an API League endpoint correctly

API League is 55 read-only HTTP GET operations behind one host, one key and one OpenAPI 3.0
contract. There is no OAuth, no scopes, no request bodies and no versioning in the URL.

## 1. Authenticate

Base URL: `https://api.apileague.com` — no version segment.

Send the key as the `x-api-key` header:

```
GET https://api.apileague.com/search-books?query=romance
x-api-key: YOUR-API-KEY
```

The provider also documents `?api-key=YOUR-API-KEY` in the query string, and leads with it in
its own examples. **Do not use it.** A key in a URL is written to proxy logs, browser history,
referrer headers and shell history. Use the header.

Keys come from https://apileague.com/console/ after a free signup. The free plan needs no card.

## 2. Make the call

Every operation is a GET with query parameters. Nothing takes a body.

```
GET https://api.apileague.com/retrieve-random-quote
x-api-key: YOUR-API-KEY
```

## 3. Read the budget off the response

Every response carries three headers. Read them on **every** call — this is the only runtime
cost signal API League gives you:

| Header | Meaning |
|---|---|
| `X-API-Quota-Request` | tokens this call just cost |
| `X-API-Quota-Used` | tokens used today, resets at midnight UTC |
| `X-API-Quota-Left` | tokens remaining today on the plan |

Billing is per **token**, not per request, and a call's token cost varies by endpoint *and* by
how its parameters are set. One request can cost many tokens. Stop when `X-API-Quota-Left`
approaches zero rather than waiting for the 402.

## 4. Handle the errors

The envelope is flat vendor JSON, not RFC 9457:

```json
{"status":"failure","code":401,"message":"Please read https://apileague.com/docs/authentication"}
```

| Status | Meaning | What to do |
|---|---|---|
| 401 | missing or invalid key | fix the key. Note: **every** unknown path on `api.apileague.com` also returns 401, so a 401 is not proof the path exists |
| 402 | daily token quota exhausted | on Free this is a hard stop until midnight UTC; on a paid plan overage billing takes over |
| 403 | not permitted for this key or plan | check the plan |
| 404 | unknown id | the id passed to a `retrieve*` operation does not exist |
| 406 | bad parameters | validate against the `pattern` / `maxLength` / `minimum` constraints in the OpenAPI before sending |
| 429 | rate **or** concurrency ceiling hit | back off — see below |

## 5. Back off yourself

**There is no `Retry-After` header and there are no `RateLimit-*` headers.** When you get a 429
the server tells you nothing about when to try again. Use your own exponential backoff with
jitter.

Two independent ceilings cause a 429, and they are not the same thing:

| Plan | Requests | Concurrent |
|---|---|---|
| Free | 60 / minute | 1 |
| Rookie | 2 / second | 5 |
| Pro | 10 / second | 10 |
| Champion | 20 / second | 20 (pricing page) / 10 (docs page — the provider's two pages disagree) |

You can exceed the concurrency ceiling without exceeding the rate: three parallel slow calls on
a 2-concurrent plan will 429 the third even though you only sent three requests.

## 6. Paginate

Search-shaped operations take `number` (page size) and `offset` (skip), and return
`total_results` (or `available`), `number` and `offset`. There is no cursor and no `Link` header.

## Do not

- **Do not cache.** API League's terms of use forbid storing or copying its data. With prior
  written permission you may cache for a maximum of one hour, then must delete and refresh.
  Do not build a durable index on this API.
- **Do not treat every GET as side-effect-free.** `storeKeyValueGETAPI` writes through a GET.
  See `api-league-key-value-store.md`.
- **Do not look for an MCP server, agent card, webhook or event stream.** None exist.
