---
name: Analyze a body of text
description: Run API League's text pipeline over a document — language, sentiment, entities, readability and quality scoring, spelling correction — in the order that avoids paying for the same work twice.
api: openapi/api-league-text-openapi.yml
operations: [detectLanguageAPI, detectSentimentAPI, extractEntitiesAPI, scoreReadabilityAPI, scoreTextAPI, correctSpellingAPI, listWordSynonymsAPI, tagPartOfSpeechAPI, stemTextAPI, extractDatesAPI]
---

# Analyze a body of text

Thirteen stateless transforms. Every one takes the text as a query parameter and returns a
fresh result — nothing is stored, nothing is addressable, and there is no batch endpoint.

Read `api-league-getting-started.md` first.

## Order matters

**Detect the language before anything else.** `correctSpellingAPI` requires a `language`
parameter, and passing the wrong one produces confident nonsense.

```
GET https://api.apileague.com/detect-language?text=Bonjour%20tout%20le%20monde
x-api-key: YOUR-API-KEY
```

`detectLanguageAPI` returns a JSON **array**, not an object — it is one of only two operations
on the platform that does (the other is `detectMainImageColorAPI`). Do not write a generic
response handler that assumes an object envelope.

## The pipeline

| Step | Operation | Path | Required params | Returns |
|---|---|---|---|---|
| 1 | `detectLanguageAPI` | `/detect-language` | `text` | array of language candidates |
| 2 | `correctSpellingAPI` | `/correct-spelling` | `text`, `language` | `corrected_text` |
| 3 | `detectSentimentAPI` | `/detect-sentiment` | `text` | `document`, `sentences` |
| 4 | `extractEntitiesAPI` | `/extract-entities` | `text` | `entities` |
| 5 | `extractDatesAPI` | `/extract-dates` | `text` | `dates` |
| 6 | `scoreReadabilityAPI` | `/score-readability` | `text` | `readability` |
| 7 | `scoreTextAPI` | `/score-text` | `title`, `text` | `number_of_words`, `number_of_sentences`, `readability`, `skimmability`, `interestingness`, `style`, `total_score` |

## Do not call both scoring operations

`scoreTextAPI` already returns `readability` as one of its seven sub-scores. If you need the
full quality profile, call `scoreTextAPI` alone. Call `scoreReadabilityAPI` only when
readability is the *only* thing you want and you do not have a title to pass. Calling both is
paying twice for the same number.

Note `scoreTextAPI` requires **both** `title` and `text`. There is no title-less variant.

## Word-level helpers

`listWordSynonymsAPI` (`/list-synonyms?word=`), `tagPartOfSpeechAPI` (`/tag-pos?text=`),
`stemTextAPI` (`/stem-text?text=`), `singularizeWordAPI` (`/singularize-word?word=`) and
`pluralizeWordAPI` (`/pluralize-word?word=`). Each is a single-purpose transform. Stemming and
POS tagging return `original`/`stemmed` and `tagged_text` respectively.

## Watch the length limits

Text parameters carry `maxLength` constraints in the OpenAPI (many are 300 characters for
`query`-style params, larger for `text`). Exceeding one returns **406**, not a truncated result.
Chunk long documents yourself and reconcile the results — there is no server-side chunking.

## Cost

Token cost varies per endpoint *and* with parameter values. When analysing a corpus, run the
full pipeline over one document first and read `X-API-Quota-Request` off each response to build
a real per-document cost before you loop.
