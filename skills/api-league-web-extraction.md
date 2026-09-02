---
name: Search the web and extract a page
description: Run a web search through API League, extract the main content of a result URL, and qualify a domain or an email address before acting on it.
api: openapi/api-league-web-openapi.yml
operations: [searchWebAPI, extractContentFromAWebPageAPI, extractAuthorsAPI, extractPublishDateAPI, retrievePageRankAPI, verifyEmailAddressAPI]
---

# Search the web and extract a page

Read `api-league-getting-started.md` first.

Six URL-shaped operations. All are read-only GETs; none of them stores anything.

## Step 1 — search

`searchWebAPI` — `GET /search-web`

`query` is **required**; `number` is optional.

```
GET https://api.apileague.com/search-web?query=openapi%20overlay%20specification&number=10
x-api-key: YOUR-API-KEY
```

Returns a `results` array. Note there is **no `offset`** here — unlike the category search
endpoints, web search is not paginated. `number` is the only lever, so ask for what you need in
one call rather than trying to page.

## Step 2 — extract the page

`extractContentFromAWebPageAPI` — `GET /extract-content`

`url` is **required**.

```
GET https://api.apileague.com/extract-content?url=https://example.com/article
x-api-key: YOUR-API-KEY
```

Returns `title`, `main_text`, `main_html`, `images`. Take `main_text` for analysis and
`main_html` when you need the structure preserved.

If the page is a **news article**, prefer `extractNewsAPI` from the News category instead — it
returns authors, publish date, language, images and videos in a single call, where
`extract-content` gives you text and you then pay separately for the rest.

## Step 3 — fill the gaps

- `extractAuthorsAPI` — `GET /extract-authors?url=…` → `authors`
- `extractPublishDateAPI` — `GET /extract-publish-date?url=…` → `publish_date`

Call these only for the field you are actually missing. Each is a separate billed call against
the same URL.

## Qualify before you act

`retrievePageRankAPI` — `GET /retrieve-page-rank`

Takes a **`domain`**, not a URL. Returns `page_rank`, `position`, `percentile`. Use it to weight
or filter search results before spending extraction tokens on low-value pages.

```
GET https://api.apileague.com/retrieve-page-rank?domain=example.com
x-api-key: YOUR-API-KEY
```

`verifyEmailAddressAPI` — `GET /verify-email?email=…`

Returns `email`, `domain`, `first_name`, `middle_name`, `last_name`, `full_name`, `username`,
`image`, `result`, `disposable`, `accept_all`, `free_provider`. Check `disposable` and
`accept_all` before treating a verification as meaningful — an `accept_all` domain accepts
anything and tells you nothing about the specific mailbox.

**This operation returns personal data.** Handle the name and image fields under your own
privacy obligations, and note that API League's terms forbid storing the response.

## Errors specific to this flow

- **404** — the target URL or domain could not be fetched.
- **406** — a malformed URL, or a `query` over its `maxLength`.
- **429** — extraction operations are slow, so a small number of parallel calls can exhaust the
  concurrency ceiling (1 on Free, 5 on Rookie) long before the per-second rate. Serialise
  extraction rather than fanning out.
