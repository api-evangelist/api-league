---
name: Search, screenshot and process images
description: Use API League's media operations — royalty-free image, icon and vector search, page screenshots, image rescaling and dominant-colour detection — including the three that return raw bytes instead of JSON.
api: openapi/api-league-media-openapi.yml
operations: [searchRoyaltyFreeImagesAPI, searchIconsAPI, vectorSearchAPI, screenshotAPI, rescaleImageAPI, detectMainImageColorAPI]
---

# Search, screenshot and process images

Read `api-league-getting-started.md` first.

The Media category is where API League stops returning JSON. Three of its six operations return
**raw bytes**, and one returns a bare JSON **array**. A generic response handler written against
the rest of the platform will break here.

## Response shapes — check before you parse

| Operation | Content-Type | Shape |
|---|---|---|
| `searchRoyaltyFreeImagesAPI` | `application/json` | object with `images` |
| `searchIconsAPI` | `application/json` | object with `icons` |
| `vectorSearchAPI` | `application/json` | object with `vectors` |
| `screenshotAPI` | `application/octet-stream` | **raw bytes** |
| `rescaleImageAPI` | `application/octet-stream` | **raw bytes** |
| `detectMainImageColorAPI` | `application/json` | **bare array**, not an object |

## Search

```
GET https://api.apileague.com/search-images?query=mountain%20sunrise&number=10
x-api-key: YOUR-API-KEY
```

- `searchRoyaltyFreeImagesAPI` — `/search-images`, `query` required, `number` optional
- `searchIconsAPI` — `/search-icons`, `query` required, `only-public-domain` and `number` optional
- `vectorSearchAPI` — `/search-vectors`, `query` required, `offset` and `number` optional

`searchIconsAPI` is the one with a licence lever: set `only-public-domain=true` when the result
will be redistributed. "Royalty-free" on `/search-images` is the provider's own framing — verify
the licence at the source before publishing anything you got back.

Note that only `vectorSearchAPI` accepts `offset`. Image and icon search cannot be paged.

## Screenshot a page

`screenshotAPI` — `GET /take-screenshot`

`url`, `width` and `height` are **all required** — there is no default viewport.

```
GET https://api.apileague.com/take-screenshot?url=https://example.com&width=1280&height=800
x-api-key: YOUR-API-KEY
```

The response body is the image itself. Do not attempt to JSON-parse it. On failure you get the
usual JSON error envelope instead, so **branch on the response `Content-Type`**, not on the
status code alone.

Screenshots are slow. On Free (1 concurrent request) a single screenshot occupies your only slot
for its full duration and every other call will 429 until it finishes.

## Rescale an image

`rescaleImageAPI` — `GET /rescale-image`

`url`, `width`, `height` and `crop` are **all required**. Returns raw bytes.

## Detect the dominant colour

`detectMainImageColorAPI` — `GET /detect-color?url=…`

Returns a bare JSON **array**. Along with `detectLanguageAPI`, it is one of only two operations
on the whole platform that does not wrap its result in an object.

## Cost

Screenshot and rescale do real rendering work and are among the most expensive operations here.
Read `X-API-Quota-Request` from the first call of each type before batching — on the Free plan's
50 tokens per day, a handful of screenshots can be the whole allowance.
