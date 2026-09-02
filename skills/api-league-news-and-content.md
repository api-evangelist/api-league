---
name: Research a news topic and extract the article
description: Search API League's news index, pull the day's top headlines for a country, then extract the full text, authors and publish date of a specific article URL.
api: openapi/api-league-news-openapi.yml
operations: [searchNewsAPI, topNewsAPI, extractNewsAPI, extractAuthorsAPI, extractPublishDateAPI]
---

# Research a news topic and extract the article

Read `api-league-getting-started.md` first — auth, quota headers and backoff apply to every
step here.

## Step 1 — search the index

`searchNewsAPI` — `GET /search-news`

The useful filters are `text`, `language`, `source-countries`, `earliest-publish-date`,
`latest-publish-date`, `news-sources`, `authors`, `categories`, `entities`, `location-filter`,
and a sentiment band via `min-sentiment` / `max-sentiment`. Paginate with `number` and `offset`.

```
GET https://api.apileague.com/search-news?text=api%20regulation&language=en&number=20
x-api-key: YOUR-API-KEY
```

Response envelope: `offset`, `number`, `available`, `news`.

**`available` is the total, not `total_results`.** The news envelope differs from the books and
recipes envelopes — do not assume one field name across categories.

## Step 2 — or take the day's top headlines

`topNewsAPI` — `GET /retrieve-top-news`

`source-country` and `language` are **required**. `date` and `headlines-only` are optional.

```
GET https://api.apileague.com/retrieve-top-news?source-country=us&language=en
x-api-key: YOUR-API-KEY
```

Returns `top_news`, `language`, `country`. Set `headlines-only=true` when you only need titles —
it is the cheaper shape.

## Step 3 — extract one article

`extractNewsAPI` — `GET /extract-news`

Both `url` and `analyze` are **required**. `analyze=true` costs more tokens; only set it when
you need the derived fields.

```
GET https://api.apileague.com/extract-news?url=https://example.com/story&analyze=false
x-api-key: YOUR-API-KEY
```

Returns `title`, `text`, `url`, `images`, `videos`, `publish_date`, `authors`, `language`.

## Step 4 — only if extract-news was not enough

`extractAuthorsAPI` (`GET /extract-authors?url=…`) and `extractPublishDateAPI`
(`GET /extract-publish-date?url=…`) each take a `url` and return just that one field.

**Do not call these routinely.** `extractNewsAPI` already returns `authors` and `publish_date`;
calling all three on the same URL spends three sets of tokens for one article. Reach for them
only when you have a URL that is not a news article, or when `extract-news` returned those
fields empty.

## Cost discipline

News operations are among the more expensive on the platform because they do retrieval and
parsing. Read `X-API-Quota-Request` after the first call of each type to learn its real cost
before looping over a result set.

## Errors specific to this flow

- **404** — a `url` that could not be fetched or does not exist.
- **406** — `text` exceeds its `maxLength`, or a date parameter is not in the expected format.
- **402** — you are out of tokens for the day; on Free nothing more will succeed until midnight UTC.
