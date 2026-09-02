---
name: Use the API League key-value store safely
description: Read and write API League's key-value store — the one operation on the platform that changes state, and the one where a GET is not safe to retry blindly.
api: openapi/api-league-storage-openapi.yml
operations: [storeKeyValueGETAPI, readKeyValueFromStoreAPI]
---

# Use the API League key-value store safely

Read `api-league-getting-started.md` first.

Two operations. This is the **only** part of API League that holds state, and the only place
where the platform's "everything is a GET" design becomes a hazard.

## Read

`readKeyValueFromStoreAPI` — `GET /read-key-value`

`key` is **required**. Returns `value`.

```
GET https://api.apileague.com/read-key-value?key=my-key
x-api-key: YOUR-API-KEY
```

A **404** means the key does not exist.

## Write

`storeKeyValueGETAPI` — `GET /store-key-value`

`key` and `value` are both **required**. Returns `status`.

```
GET https://api.apileague.com/store-key-value?key=my-key&value=my-value
x-api-key: YOUR-API-KEY
```

## The warning

**This is a write behind an HTTP GET.** Every safety assumption an agent normally makes about
GET is wrong here:

- **Do not prefetch it.** Anything that speculatively follows GET URLs — a crawler, a link
  preloader, an HTTP cache warming pass, a retry-everything wrapper — will silently overwrite
  stored data.
- **Do not put it in a "safe to retry" bucket.** It is idempotent per key by last-write-wins, so
  a retry of the *same* call is harmless, but a retry of a *stale* call will clobber a newer
  value.
- **Do not log the URL.** The value is in the query string, alongside the API key if you used
  the query-string auth form. Use the `x-api-key` header and redact the `value` parameter from
  logs.

## There is no undo

API League publishes **no** reversal operation for this store:

- no delete-key
- no undo or rollback
- no versioning or history of a stored value
- no documented retention or recovery window

The only rollback available is one you build yourself:

```
1. readKeyValueFromStoreAPI(key)     -> keep the current value
2. storeKeyValueGETAPI(key, new)     -> write
3. on failure: storeKeyValueGETAPI(key, old)   -> restore from what you kept
```

If you did not do step 1, the previous value is gone. **Read before you write, every time.**

## What is not documented

The provider publishes no key namespace rules, no value size limit, no key count limit, no TTL
and no isolation guarantee beyond the API key. Treat the store as best-effort scratch space
tied to your key, not as a database. Do not put anything in it you cannot reconstruct.

## Compliance

API League's terms of use forbid storing API League data outside the platform, and separately
disclaim all warranties on availability. Both apply here: this store is neither a backup nor a
system of record.
