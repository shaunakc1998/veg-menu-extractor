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

### When results are thin

- Broaden from neighborhood to city
- Try: `restaurants near [landmark]`
- Try: `[location] restaurants doordash`

---

## 2. Menu crawling — source priority

Try in order. Stop once you have 8+ dish names. Budget 2–4 calls per
restaurant.

### Source 1: Restaurant's own website

Search: `[restaurant name] [city] menu`

URL patterns to try: `/menu`, `/food`, `/dinner-menu`, `/food-menu`,
`/our-menu`, `/menus/dinner`

Check for multiple menus (dinner, lunch, brunch, bar). Check for a
separate vegetarian/vegan menu page.

### Source 2: Yelp

Search: `[restaurant name] [city] site:yelp.com`

URLs: `yelp.com/menu/[slug]` or `yelp.com/biz/[slug]`

### Source 3: Delivery platforms

Search: `[restaurant name] [city] doordash` (or ubereats, grubhub)

Often the most complete and current menus.

### Source 4: Google search snippets / aggregators

Search: `[restaurant name] [city] full menu items`

### Source 5: Review sites and blogs

Search: `[restaurant name] menu items what to order`

Use for hero dish identification and supplementary data. Always label
review-sourced info as such.

### Common technical failures

| Situation | What to do |
|---|---|
| JavaScript-rendered menu | Try Yelp or delivery platform |
| PDF menu | Note "Menu is PDF — check link"; try Yelp |
| Image-only menu | Note "Menu is image — check link"; try Yelp |
| 403 Forbidden | Try alternate source |
| Menu behind reservation wall | Note the restriction |

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

**Score drops (-1):** Only sides/salads, heavy "can be made veg" reliance,
many "verify" items suggesting low veg awareness.

---

## 9. Handling common failure modes

| Problem | What to do |
|---|---|
| web_fetch returns JS-only page | Try Yelp or delivery platform |
| web_fetch returns 403 | Try different source; note block |
| Menu is PDF | Note it; try Yelp |
| Menu is image | Note it; try Yelp |
| No online menu anywhere | Include with "Menu not available online" |
| 100+ item menu | Classify them all — that's the skill's value |
| 0 veg items | Include with verdict 🔴 and note |
| Single restaurant request | Skip Step 2, go straight to Steps 3–5 |
| Only brunch menu found | Note "Dinner menu not available online" |
| Seasonal/rotating menu | Note "Menu changes seasonally — verify" |
| Prices not shown | Omit price; note "prices not listed" |
| Menu in another language | Do your best; note language |
| Veg items found only in reviews | Include with "Sourced from reviews" note |
| User changes dietary profile mid-conversation | Re-filter existing data through new profile |
