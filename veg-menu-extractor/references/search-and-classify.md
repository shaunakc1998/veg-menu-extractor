# Search and Classification Guide

Detailed reference for the veg-menu-extractor skill. Read this before
executing the workflow.

## Table of contents

1. Search query templates
2. Menu crawling — source priority and techniques
3. Vegetarian classification rules
4. Dietary profile adjustments
5. Glossary of hidden non-vegetarian ingredients
6. Hero dish identification
7. Server script templates
8. Counting, scoring, and categorization
9. Handling common failure modes
10. Hours and practical logistics

---

## 1. Search query templates

### Broad discovery (run at least 2)

```
best restaurants in [location]
top rated restaurants [location]
popular dinner restaurants [location] [year]
best new restaurants [location] [year]
```

### Source-targeted (run 1–2 for variety)

```
eater best restaurants [location]
infatuation best restaurants [location]
timeout best restaurants [location]
```

### Cuisine-specific (if user specifies)

```
best [cuisine] restaurants [location]
top [cuisine] restaurants near [location]
```

### Price-targeted (if user specifies)

```
best cheap eats [location]
best fine dining [location]
best mid-range restaurants [location]
```

### Veg-specialist sweep (MANDATORY — do not skip)

This is the single most important search step for finding the best
restaurant for a vegetarian user. In real-world testing, the best
restaurant was missed TWICE because this step was skipped or run too
narrowly. Both times the user had to manually ask about the restaurant
that should have been found first.

**Why it keeps failing:** General "best [cuisine] in [location]" searches
surface the most popular/reviewed places. Veg-specialist restaurants are
smaller, newer, and often in adjacent neighborhoods. They get drowned out
by mainstream places with more reviews. The only way to find them is to
search specifically for veg-friendliness.

**Step 2a — Veg-signal searches (same location, at least 2):**

```
vegetarian friendly [cuisine] [location]
[cuisine] [location] vegetarian options
[cuisine] [location] vegan friendly
best vegetarian [cuisine] [location]
[cuisine] [location] plant based
```

For general (non-cuisine-specific) searches:
```
vegetarian friendly restaurants [location]
best restaurants vegetarian options [location]
vegan friendly dining [location]
```

**Step 2b — Expanded radius search (REQUIRED, at least 1):**

Veg-specialist restaurants are rarer than mainstream ones. The best option
for a vegetarian is often 10–15 minutes away in an adjacent city or
neighborhood. Always expand the search area for veg-signal queries:

| User says | Also search |
|---|---|
| A city (e.g., "San Jose") | The metro area: "South Bay," "Sunnyvale," "Santa Clara" |
| A neighborhood (e.g., "West Village") | The city: "Manhattan," "NYC" |
| A metro area (e.g., "Bay Area") | Already broad — use as-is |
| A small town | The nearest city or county |

Expanded radius queries:
```
vegetarian [cuisine] [broader area]
vegan [cuisine] near [location]
best vegetarian [cuisine] [metro area]
[cuisine] vegetarian [adjacent city 1]
[cuisine] vegetarian [adjacent city 2]
```

**If a veg-specialist is found outside the stated area**, include it with
a distance note: "⚠️ Located in [City], ~15 min from [user's area] — worth
the drive for the best veg options in the region."

**Real failures this prevents:**
- Mumu Hot Pot (Emeryville) missed when user searched "Bay Area hotpot" —
  drowned out by louder BBQ+hotpot combos
- Oh Baby Sushi (Sunnyvale) missed when user searched "Japanese San Jose
  Milpitas" — in adjacent city, invisible to location-specific searches

Both were the #1 best restaurant for the vegetarian user. Both were only
found when the user manually asked. This sweep prevents that.

### Cuisine-aware subtype searches

Some cuisines have subtypes that are structurally more veg-friendly. When
the user asks for a specific cuisine, search for the veg-favorable subtype
too — don't just search for the umbrella term.

| Cuisine | Veg-favorable subtype | Why | Search to add |
|---|---|---|---|
| Hotpot / Korean | Dedicated hotpot (not BBQ+hotpot) | Individual pots, menus built around broth/veg/tofu, no meat cross-contamination | `hotpot [location]` (not just "korean bbq hotpot") |
| Indian | South Indian, "pure veg" | More inherently vegetarian dishes | `south indian [location]`, `pure veg [location]` |
| Japanese | Izakaya, Buddhist/shojin, veg sushi | More cooked veg options, some specialize in veg rolls | `japanese vegetarian [location]`, `vegan sushi [location]` |
| Italian | Pasta-focused, Roman | Cacio e pepe, carciofi, more veg-first dishes | `pasta restaurant [location]` |
| Mexican | Regional Mexican, Oaxacan | Bean/cheese/mole traditions, less meat-centric | `mexican vegetarian [location]` |
| Thai | Thai with veg options | Some cook without fish sauce on request | `thai vegetarian [location]` |
| Chinese | Buddhist vegetarian, Sichuan | Tofu/mushroom traditions, mapo tofu | `chinese vegetarian [location]` |

### Ranking candidates

When trimming from 8–12 candidates down to the requested count:

**Prioritize rating quality over review volume.** A 4.8⭐ restaurant with
500 reviews is a much stronger signal than a 4.2⭐ with 2,000 reviews.
High-rated restaurants with moderate review counts are often newer, more
intentional places — exactly the type that tend to have thoughtful veg
options. Don't drop them just because they're less "famous."

**Ranking heuristic (apply after menu analysis):**
1. Veg-friendliness score (from Step 4) — the primary sort
2. Star rating — tiebreaker between equal veg scores
3. Review count — only use to break ties between equal ratings
4. Cuisine diversity — prefer a mix over 5 restaurants of the same type

**Never drop a veg-specialist restaurant during trimming.** If a restaurant
has a dedicated veg menu, labeled veg items, or a veg combo — it stays in
the final list regardless of review count or how well-known it is. These
are the most valuable results for the user.

### When results are thin

- Broaden from neighborhood to city
- Try: `restaurants near [landmark]`
- Try: `[location] restaurants doordash`

---

## 2. Menu crawling — source priority

Try in order. Stop once you have 8+ dish names. Budget 2–4 calls per
restaurant.

**Important — cuisine-aware source selection:** Many Asian restaurants
(Chinese, Korean, Japanese, Thai, Vietnamese) use image-based menus, PDFs,
or JavaScript-heavy sites that web_fetch can't read. For these cuisines,
**skip straight to delivery platforms** (Source 3) if the restaurant
website doesn't yield text content on the first try. Don't waste 3 attempts
on an image menu — DoorDash/UberEats almost always have the full text menu.

### Source 1: Restaurant's own website

Search: `[restaurant name] [city] menu`

URL patterns to try: `/menu`, `/food`, `/dinner-menu`, `/food-menu`,
`/our-menu`, `/menus/dinner`

Check for multiple menus (dinner, lunch, brunch, bar). Check for a
separate vegetarian/vegan menu page.

If web_fetch returns mostly images, navigation, or boilerplate → go to
Source 3 (delivery platforms) immediately, not Source 2.

### Source 2: Yelp

Search: `[restaurant name] [city] site:yelp.com`

URLs: `yelp.com/menu/[slug]` or `yelp.com/biz/[slug]`

Good for restaurants with text-based menus. Less useful for Asian
restaurants since Yelp menu coverage is inconsistent for these.

### Source 3: Delivery platforms (go-to for image menus)

Search: `[restaurant name] [city] doordash` (or ubereats, grubhub)

Often the most complete and current menus — and critically, they always
have **text-based** item listings even when the restaurant's own site uses
images or PDFs. For AYCE restaurants (hotpot, BBQ, buffet), delivery
platforms may not list the full in-restaurant menu since AYCE doesn't
translate to delivery. In that case, combine delivery menu data with
review-sourced info.

### Source 4: Google search snippets / aggregators

Search: `[restaurant name] [city] full menu items`

### Source 5: Review sites and blogs

Search: `[restaurant name] menu items what to order`

Use for hero dish identification and supplementary data. Always label
review-sourced info as such. Especially useful for AYCE restaurants
where reviewers often list exactly what's on the belt or buffet.

### Common technical failures

| Situation | What to do |
|---|---|
| JavaScript-rendered menu | Try delivery platforms first, then Yelp |
| PDF menu | Note "Menu is PDF — check link"; try delivery platforms |
| Image-only menu | Go straight to DoorDash/UberEats; skip Yelp |
| 403 Forbidden | Try alternate source |
| Menu behind reservation wall | Note the restriction |
| AYCE menu not on delivery apps | Use reviews + restaurant website together |

---

## 3. Vegetarian classification rules

These are the baseline rules for **lacto-ovo vegetarian** (the default
profile). See Section 4 for adjustments to other dietary profiles.

### Definitely vegetarian ✅

- Dishes explicitly labeled (V), vegetarian, or plant-based
- Salads with no meat/seafood (garden, beet, caprese, grain bowls)
- Pasta/noodles with vegetable, cheese, cream, or tomato sauces
- Vegetable soups where no animal broth is indicated
- Cheese/mushroom/vegetable pizzas and flatbreads
- Vegetable curries, dal, paneer, chana masala
- Tofu and tempeh dishes (unless Thai/Vietnamese — check fish sauce)
- Egg dishes: omelets, frittatas, shakshuka
- Bread, bruschetta, hummus, guacamole, fries, onion rings
- Veggie burgers (lentil, black bean, etc.)
- Most desserts: cakes, pies, ice cream, soufflés, puddings

### Vegan 🌱

Only label when confident from the description — no dairy, eggs, honey,
butter:
- Dishes labeled "vegan"
- Vegetable salads with vinaigrette (not creamy dressing)
- Bean/lentil soups with vegetable broth
- Sorbet, fruit plates, chia puddings with plant milk

### Can be made vegetarian ⚠️

Only when the modification is reasonable and the dish still makes sense:

**Good modifications:**
- Caesar salad → "no anchovies in dressing"
- Cobb salad → "no bacon, no chicken"
- Pasta with optional meat → "without [protein]"
- Burrito/taco with filling choices → "choose beans/vegetables"
- Pizza with one meat topping → "substitute mushrooms"

**Don't suggest:** removing the main protein from a dish built around it.

**Fine dining:** Don't suggest modifications. Instead: "Call ahead to ask
about vegetarian tasting menu accommodations."

### Verify with restaurant ❓

For the default vegetarian profile, flag these. See Section 4 for which
profiles can skip these flags.

**Thai / Vietnamese / Southeast Asian:**
- Pad thai, curries, stir-fries — fish sauce / shrimp paste
- Coconut curries — shrimp paste in base

**Japanese:**
- Miso soup, edamame — dashi (fish stock)
- Tempura batter — sometimes contains fish

**French / European:**
- French onion soup — beef broth
- Risotto — chicken stock
- Gratins — chicken stock base

**Mexican / Latin American:**
- Refried beans — lard
- Rice — chicken broth
- Tamales — lard in masa

**General:**
- Fried rice — oyster sauce
- Caesar dressing — anchovies
- Worcestershire-based sauces — anchovies
- "Vegetable" soups at non-veg restaurants — animal stock

### Not vegetarian ❌

Skip entirely:
- Meat, poultry, game, fish, shellfish, seafood
- Dishes containing bacon, pancetta, prosciutto, guanciale, lardons,
  anchovies as a listed ingredient (unless flagged as "can be made veg")

---

## 4. Dietary profile adjustments

Apply these adjustments on top of the base classification rules:

### Vegan

- Remove all items with dairy (cheese, cream, butter, yogurt, milk)
- Remove all items with eggs
- Remove items with honey
- Items that are vegetarian but not vegan → mark as "⚠️ Contains [dairy/
  eggs] — ask if a vegan version is available"
- Many desserts drop off (ice cream, crème brûlée, etc.)
- Be extra careful with bread (butter, eggs), pasta (eggs), sauces (cream)

### Pescatarian

- Include fish and seafood dishes as valid options
- Skip fish sauce, anchovy, dashi, oyster sauce warnings entirely
- Still exclude meat and poultry
- The "verify" list shrinks dramatically for this profile

### Flexitarian / "not strict"

- Include all vegetarian items
- Remove "verify with restaurant" flags for minor ingredients:
  fish sauce, dashi, oyster sauce, Worcestershire, chicken broth
- Keep flags only for actual meat pieces (bacon bits, etc.)
- Can note: "Contains fish sauce (OK for your profile)"

### Egg-free vegetarian

- Remove egg dishes (omelets, frittatas, shakshuka, egg sandwiches)
- Flag dishes where eggs are likely but not explicit (pasta dough,
  some desserts, some breads)
- Many brunch menus will be very limited for this profile — note that

### Jain vegetarian

- Remove all root vegetables: onion, garlic, potatoes, carrots, beets,
  turnips, radishes
- Remove mushrooms (some Jain traditions avoid them)
- Remove eggs
- This is the most restrictive profile — note that options will be
  limited and the user should call ahead

### Allergen stacking

When the user has allergies on top of their dietary profile, apply both
filters. Present items that pass both filters, and flag items that are
vegetarian but contain the allergen.

**Vegetarian + Gluten-free:**
- Remove: pasta (unless GF), bread, soy sauce (use tamari), seitan,
  beer batter, flour-based sauces, regular noodles
- Flag: "⚠️ Contains gluten" on items with flour, wheat, barley, rye
- Watch: many Asian sauces contain wheat-based soy sauce
- Hotpot-specific: most broths are GF but verify; rice noodles and
  rice cake are safe; udon and ramen are not
- Server script: "I'm vegetarian and gluten-free — could you check if
  the [dish] contains any wheat, soy sauce, or flour?"

**Vegetarian + Nut-free:**
- Remove: pesto (pine nuts), Thai curries with peanuts, dishes with
  cashew cream, many Indian gravies (often use cashew or almond paste)
- Flag: "⚠️ Contains nuts" on items with any tree nuts or peanuts
- Watch: desserts (often have hidden nuts), pad thai (peanuts),
  Middle Eastern food (pistachios, walnuts)
- Server script: "I have a nut allergy — are there any peanuts, tree
  nuts, or nut-based pastes in the [dish]?"

**Vegetarian + Soy-free:**
- Remove: tofu, tempeh, edamame, miso soup, soy sauce, teriyaki
- This eliminates most Asian veg options — note the limitation clearly
- Server script: "I can't have soy — does the [dish] contain tofu,
  soy sauce, or any soy products?"

**General allergen handling:**
- Note in the dining tip if a restaurant has a published allergen menu
  or is known for accommodating allergies
- If a restaurant uses QR-code ordering (common in hotpot/AYCE), note
  that allergen filters are sometimes available in the ordering system
- When allergen + dietary combo leaves very few options, say so honestly
  and suggest calling ahead

---

## 5. Glossary of hidden non-vegetarian ingredients

These terms mean the dish is **not vegetarian** even if the name sounds
innocent. Catching these is one of the skill's highest-value features.

| Term | What it is | Example |
|---|---|---|
| **Guanciale** | Cured pork jowl | "Parmesan agnolotti, guanciale, arugula" |
| **Pancetta** | Italian cured pork belly | "Brussels sprouts with pancetta" |
| **Lardons** | Small strips of pork fat | "Frisée salad with lardons" |
| **Prosciutto** | Dry-cured ham | "Prosciutto-wrapped melon" |
| **Nduja** | Spicy spreadable salami | "Nduja butter" on bread |
| **Coppa / Capicola** | Cured pork neck | Charcuterie boards |
| **Speck** | Smoked prosciutto | "Speck and fontina pizza" |
| **Bresaola** | Air-dried beef | "Bresaola carpaccio" |
| **Suet** | Hard beef/mutton fat | British pastry/puddings |
| **Tallow** | Rendered beef fat | Some frying oils |
| **Schmaltz** | Rendered chicken fat | "Schmaltz mashed potatoes" |
| **Dashi** | Japanese fish stock | Miso soup, ramen |
| **Bonito** | Dried fish flakes | Okonomiyaki, Japanese dishes |
| **Fish sauce / Nam pla** | Fermented fish | Thai/Vietnamese dishes |
| **Oyster sauce** | Oyster-based condiment | Chinese stir-fries |
| **Shrimp paste** | Fermented shrimp | Thai curry pastes |
| **Anchovy** | Small preserved fish | Caesar dressing, puttanesca |
| **Worcestershire** | Contains anchovies | Some sauces, Bloody Mary |
| **XO sauce** | Dried shrimp/scallop | Chinese dishes |
| **Colatura** | Italian fish sauce | Southern Italian pasta |
| **Garum** | Fermented fish sauce | Some modern fine dining |
| **Gelatin** | Animal-derived gelling | Panna cotta, some mousses |
| **Lard** | Rendered pork fat | Refried beans, pie crust |
| **Ragù / Ragu** | Usually meat sauce | Bolognese, short rib ragu |
| **Jus** | Meat drippings | "Served with jus" |
| **Demi-glace** | Concentrated meat stock | French sauces |
| **Fond** | Meat stock base | French cooking sauces |
| **Sugo** | Can mean meat sauce | "Sugo di carne" |

---

## 6. Hero dish identification

The goal is to find the 1–2 vegetarian dishes per restaurant that a
vegetarian would actually be *excited* about — not just tolerant of.

### Signals that a dish is a hero

- **Reviewers call it out by name.** "The mushroom risotto is the best
  thing on the menu" or "even meat-eaters order the cauliflower."
- **It's clearly intentional.** A composed vegetarian main with its own
  identity (turmeric-roasted cauliflower with lentil curry) vs an
  afterthought (plain pasta with marinara).
- **It has complexity.** Multiple components, interesting technique,
  unique ingredients — not just "garden salad."
- **It's a main, not a side.** A hero dish should be something you'd
  build a meal around.

### Where to find hero dish signals

- The Infatuation, Eater, TimeOut reviews often name specific dishes
- Yelp reviews mentioning vegetarian experiences
- "What to order at [restaurant]" articles
- Search: `[restaurant name] best vegetarian dish`

### How to present hero dishes

Mark with ⭐ and add a brief note about why:

```
⭐ Hero dish: Turmeric-Roasted Cauliflower ($22) — "The lentil-garbanzo
curry underneath makes this a full meal. Multiple reviewers highlight
it as a standout."
```

If no dish stands out, don't force it. Just omit the hero dish line.

---

## 7. Server script templates

Instead of just saying "verify with restaurant," give the user the actual
question to ask. This is one of the most practically useful features.

### General format

Scripts should be:
- **Polite and specific** — not "is this vegetarian?" (too vague)
- **Name the ingredient** they're worried about
- **Short enough to actually say** out loud to a server

### Template scripts by situation

**For fish sauce / shrimp paste (Thai, Vietnamese):**
"Is the [dish name] made with fish sauce? I'm vegetarian, so I'd need
it without any fish products."

**For broth/stock (soups, risotto, French onion):**
"What kind of stock is the [dish name] made with? I'm looking for
vegetable-based rather than chicken or beef."

**For Caesar dressing:**
"Is the Caesar dressing made with anchovies? If so, could I get a
different dressing?"

**For hidden pork (refried beans, lard):**
"Are the refried beans made with lard or vegetable oil? I don't eat pork."

**For dashi (Japanese):**
"Is the miso soup made with dashi? I'm looking for a version without
fish stock."

**For general "is this veg?" checks:**
"I'm vegetarian — are there any meat products, fish sauce, or animal
stock in the [dish name]?"

**For fine dining / tasting menus (call ahead):**
"Hi, I have a reservation for [date]. One guest is vegetarian — do you
offer a vegetarian tasting menu, or can the chef make substitutions
for the meat and fish courses?"

**For vegan (call ahead or at table):**
"I'm vegan — could you let me know which dishes are made without any
dairy, eggs, or butter? I'm happy with any modifications the kitchen
can make."

---

## 8. Counting, scoring, and categorization

### Category mapping

| Restaurant section | Output category |
|---|---|
| Appetizers, Starters, Small Plates, Shareables, Snacks | Starters |
| Entrées, Mains, Large Plates, Specialties | Mains |
| Sides, Accompaniments, Extras, Add-ons | Sides |
| Desserts, Sweets, After Dinner | Desserts |
| Salads (small / shareable) | Starters |
| Salads (entrée-sized) | Mains |
| Soups | Starters |
| Pizzas, Pasta (entrée portion) | Mains |
| Brunch items | Label "[Brunch]" and best-fit |
| Bar snacks, Lounge items | Label "[Bar/Lounge]" and best-fit |

### Veg count

- **Numerator:** Items classified as vegetarian + vegan + "can be made veg"
- **Denominator:** Total distinct menu items across all categories
- Display: `📊 Veg count: X / Y total items`
- Partial menus: `📊 Veg count: X identified (partial menu)`
- Prix fixe: `📊 Vegetarian choices in X of Y courses`

### Veg-friendliness score

| Score | Verdict | Criteria |
|-------|---------|----------|
| 5/5 | 🟢 **Veg paradise** | 10+ options, dedicated veg mains, kitchen is veg-aware |
| 4/5 | 🟢 **Great for veg** | 6–9 options including 2+ substantial mains |
| 3/5 | 🟡 **Workable** | 3–5 options, at least 1 real main |
| 2/5 | 🟠 **Slim pickings** | 1–2 options, mostly sides or salads |
| 1/5 | 🔴 **Rough** | 0–1 items, would need major modifications |

**Score bumps (+1):** Dedicated veg menu section, (V)/(VG) labels, reviews
praising veg options, substantial veg mains.

**Automatic 5/5 signals (any one of these = veg paradise):**
- Restaurant offers a dedicated vegetarian combo/set meal
- Menu has labeled vegan/vegetarian soup bases, broths, or sauces
- Restaurant has a full separate vegetarian menu
- Significant portion (30%+) of the menu is natively vegetarian
- Individual cooking vessels (hotpot, fondue) eliminating cross-contamination

These signals indicate the kitchen *thought about* vegetarians as a real
customer, not an afterthought. This is the difference between "we have
salad" and "we built a vegetarian experience." Restaurants with these
signals should always be ranked above those with more total veg items but
no intentional design — a place with 8 items and a dedicated veg combo
beats a place with 15 items that are all "sides you can technically eat."

**Score drops (-1):** Only sides/salads, heavy "can be made veg" reliance,
many "verify" items suggesting low veg awareness.

---

## 9. Handling common failure modes

| Problem | What to do |
|---|---|
| web_fetch returns JS-only page | Try delivery platforms first, then Yelp |
| web_fetch returns 403 | Try different source; note block |
| Menu is PDF or image | Try DoorDash/UberEats immediately; they always have text |
| No online menu anywhere | Include with "Menu not available online" |
| 100+ item menu | Classify them all — that's the skill's value |
| 0 veg items | Include with verdict 🔴 and note |
| Single restaurant request | Skip Step 2, go straight to Steps 3–5 |
| Only brunch menu found | Note "Dinner menu not available online" |
| Seasonal/rotating menu | Note "Menu changes seasonally — verify" |
| Prices not shown | Omit price; note "prices not listed" |
| Menu in another language | Do your best; note language |
| Veg items found only in reviews | Include with "Sourced from reviews" note |
| User changes dietary profile | Re-filter existing data through new profile |
| AYCE menu not on delivery apps | Combine reviews + restaurant site for item list |
| User has allergen + dietary combo | Apply both filters; flag items that pass one but not other |
| Broad location (e.g., "Bay Area") | Ask for neighborhood or pick 2–3 popular dining areas |

---

## 10. Hours and practical logistics

Restaurant hours matter — don't recommend a place that's closed when the
user wants to eat.

### What to include

When crawling a restaurant, look for hours in:
- The restaurant's website (usually in header/footer)
- Google search snippets (often show hours directly)
- Yelp business info

Include hours in the output template: `🕐 Hours: [e.g., Tue–Sun 5–10pm,
closed Mon]`

### What to flag

- **Closed on the user's day:** If the user says "dinner tomorrow" and
  tomorrow is Monday, flag restaurants closed on Mondays
- **Limited hours:** "Dinner only Wed–Sat" or "Lunch only" — note this
- **Seasonal closures:** Some restaurants close for weeks; note if mentioned
- **Wait times:** If reviews consistently mention long waits, note it in
  the dining tip: "Expect 45–60 min wait without reservation"
- **Reservation required vs walk-in:** Note which approach works

### When hours aren't available

If you can't find hours, note "🕐 Hours not confirmed — check before going"
rather than omitting the field. Users seeing no hours field won't think
to check; seeing the note reminds them.
