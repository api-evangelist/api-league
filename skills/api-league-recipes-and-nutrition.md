---
name: Find a recipe and compute its nutrition
description: Search API League's recipe index with dietary and nutrient filters, retrieve the full recipe by id, and compute nutrition for an arbitrary ingredient list.
api: openapi/api-league-food-openapi.yml
operations: [searchRecipesAPI, retrieveRecipeInformationAPI, computeNutritionAPI, searchDrinksAPI]
---

# Find a recipe and compute its nutrition

Read `api-league-getting-started.md` first.

The Food category is the richest filter surface on the platform — `searchRecipesAPI` alone
accepts 88 query parameters — and it is one of only four categories with a real
search → retrieve round trip.

## Step 1 — search

`searchRecipesAPI` — `GET /search-recipes`

Every parameter is optional. The ones worth knowing:

- **Text and taxonomy**: `query`, `cuisines`, `exclude-cuisines`, `meal-type`, `diet`,
  `intolerances`, `equipment`
- **Ingredients**: `include-ingredients`, `exclude-ingredients`, `fill-ingredients`
- **Practicality**: `max-time`, `min-servings`, `max-servings`
- **Macros**: `min-calories`/`max-calories`, and the same min/max pairs for `carbs`, `protein`,
  `fat`, `sugar`, `fiber`, `saturated-fat`, `cholesterol`, `alcohol`, `caffeine`
- **Micros**: min/max pairs for `folate`, `folic-acid`, `iodine`, `iron`, `zinc`, `magnesium`,
  `manganese`, `phosphorus`, `potassium`, `sodium`, `selenium`, `copper`, `calcium`, `choline`,
  `fluoride`, and vitamins `a`, `c`, `d`, `e`, `k`, `b1`, `b2`, `b3`, `b5`, `b6`, `b12`
- **Paging and order**: `sort`, `sort-direction`, `offset`, `number`

```
GET https://api.apileague.com/search-recipes?query=curry&diet=vegetarian&max-time=30&number=10
x-api-key: YOUR-API-KEY
```

Returns `offset`, `number`, `recipes`, `total_results`.

**`add-recipe-information=true` is the cost decision.** It inflates each search hit with the
full recipe payload, which means you may not need step 2 at all — but it makes the search call
more expensive whether or not you use the extra data. Set it when you know you want detail for
most results; leave it off and retrieve selectively when you want detail for a few.

## Step 2 — retrieve one recipe

`retrieveRecipeInformationAPI` — `GET /retrieve-recipe`

`id` is **required**; the id comes from a `searchRecipesAPI` hit. `add-wine-pairing` is optional.

```
GET https://api.apileague.com/retrieve-recipe?id=12345
x-api-key: YOUR-API-KEY
```

Returns `id`, `title`, `servings`, `images`, `dietary_properties`, `price_per_serving`, `times`,
`nutrition`, `taste`, `cuisines`, `meal_types`, `occasions`.

`nutrition` is embedded — you do **not** need `computeNutritionAPI` for a recipe that came from
the index. A 404 here means the id does not exist.

## Step 3 — nutrition for an arbitrary ingredient list

`computeNutritionAPI` — `GET /compute-nutrition`

This is for text you supply, not for an indexed recipe. `ingredients` is **required**;
`servings` and `reduce-oils` are optional.

```
GET https://api.apileague.com/compute-nutrition?ingredients=2%20cups%20rice%0A1%20tbsp%20olive%20oil&servings=4
x-api-key: YOUR-API-KEY
```

Returns `nutrients`, `properties`, `flavonoids`, `ingredient_breakdown`, `caloric_breakdown`,
`weight_per_serving`.

## Drinks

`searchDrinksAPI` — `GET /search-drinks` — mirrors the recipe filter model with drink-specific
additions: `glass-types`, `flavors`, `min-alcohol-percent`/`max-alcohol-percent`,
`min-caffeine`/`max-caffeine`. Envelope: `offset`, `number`, `drinks`, `total_results`.

## Cross-category caution

There is **no** identifier that crosses categories in API League. A recipe id is meaningless to
the Books, Games or Art endpoints. Do not attempt joins across categories.

## Compliance

Do not persist recipe or nutrition data. The terms of use forbid storing API League data; a
one-hour cache is permitted only with prior written permission.
