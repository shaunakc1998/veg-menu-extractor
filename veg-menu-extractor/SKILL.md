---
name: veg-menu-extractor
description: >
  Finds top-rated restaurants in any location and extracts only the vegetarian
  dishes from each menu — using web_search and web_fetch only, no paid APIs.
  Use this skill when the user asks about vegetarian options at restaurants,
  wants veg-friendly dining in a city or neighborhood, asks "where can a
  vegetarian eat in [place]?", wants to compare vegetarian menus, is planning
  dinner with mixed veg/non-veg diners, or says things like "veg options near
  me", "vegetarian menu items in [location]", "restaurants with good veg food
  in [area]", "extract vegetarian dishes", "veg menu", or "plant-based options
  at restaurants". Also trigger when someone names a specific restaurant and
  wants its veg dishes listed. Targets ANY restaurant type — not just
  vegetarian restaurants — because the core value is helping mixed dining
  groups. Even if the user just mentions wanting to eat out as a vegetarian
  or finding food options somewhere, use this skill.
---

# Veg Menu Extractor

Find great restaurants in any location and surface only the plant-forward
options from their menus. Handles vegetarian, vegan, pescatarian, and custom
dietary needs. Uses `web_search` and `web_fetch` exclusively — no API keys,
no paid services, no external dependencies.

## The problem this solves

When a vegetarian dines with non-vegetarians, the group doesn't need a
"vegetarian restaurant" — they need a great restaurant of any cuisine where
the vegetarian also has solid options. Scanning every menu by hand is tedious.
This skill automates that — and goes further by rating each restaurant's
veg-friendliness, highlighting hero dishes, and giving practical tips for
what to actually say to the server.

## Before you begin

Read `references/search-and-classify.md` for the full search strategy,
menu-crawling patterns, classification rules, hidden-ingredient glossary,
and server script templates. That file is the authoritative reference.

## Workflow

### Step 1 — Gather inputs and dietary profile

Collect from the user (ask only for what they haven't provided):

| Input              | Required | Default            |
|--------------------|----------|--------------------|
| Location           | Yes      | —                  |
| Dietary profile    | No       | "vegetarian"       |
| Cuisine preference | No       | any                |
| Price range        | No       | any                |
| Min rating         | No       | 4.0 stars          |
| Meal period        | No       | dinner             |
| Number of results  | No       | 5–8                |
| Other filters      | No       | none               |

If the user gives a simple prompt like "veg options in downtown Austin," use
sensible defaults and go — don't interrogate them.

#### Dietary profiles

"Vegetarian" means different things to different people. Detect or ask about
the user's profile and tailor classification accordingly:

| Profile | What they avoid | Common in |
|---------|----------------|-----------|
| **Lacto-ovo vegetarian** (default) | Meat, poultry, fish, seafood | Most Western vegetarians |
| **Vegan** | All animal products (dairy, eggs, honey) | — |
| **Pescatarian** | Meat, poultry (fish/seafood OK) | — |
| **Flexitarian / "veg-curious"** | Prefers veg but open to fish sauce, broth, etc. | Many real-world diners |
| **Jain vegetarian** | Meat, fish, eggs, root vegetables (onion, garlic) | Indian community |
| **Egg-free vegetarian** | Meat, fish, eggs (dairy OK) | Some South/East Asian |
| **Custom** | Whatever the user specifies | "I don't eat meat but fish sauce is fine" |

**Cues to detect profile (no need to ask):**
- "I'm vegan" → vegan profile
- "I eat fish" → pescatarian
- "Fish sauce is fine" / "I'm not strict" → flexitarian
- "No eggs" → egg-free vegetarian
- No specification → default to lacto-ovo vegetarian

Adjust classification in Step 4 based on the profile. For example:
- A flexitarian doesn't need "verify fish sauce" flags
- A vegan needs dairy/egg items filtered out, not just meat
- A pescatarian can eat the swordfish — include it

### Step 2 — Discover top-rated restaurants

Run **3–5 varied `web_search` queries** to get diverse results. See
`references/search-and-classify.md` for the full query template list.

**Search for ALL cuisine types** — Italian, steakhouse, Thai, Mexican, Indian,
Japanese, American, Mediterranean, fusion, everything. Don't bias toward
"vegetarian-friendly" places. A steakhouse with great sides is fair game.

From results, extract for each restaurant: name, cuisine type, rating, price
range, address. Deduplicate by name. Discard fast food chains (unless
requested). Keep 8–12 candidates, then trim after menu analysis.

### Step 3 — Crawl each restaurant's menu

For each restaurant, locate and retrieve the actual menu. Try sources in the
priority order listed in the reference guide. Budget **2–4 tool calls per
restaurant** (1 search + 1–3 fetches).

**Key things to watch for:**
- **Multiple menus:** Focus on the user's meal period. Note if a secondary
  menu (bar, lounge) has additional veg options.
- **Separate vegetarian/vegan menus:** Search `[restaurant] vegetarian menu`
  if the main menu seems sparse.
- **Menu symbols:** (V), (VG), (GF), 🌱 — use these but verify with
  descriptions.
- **Prix fixe / tasting menus:** Present by course, not by category.

### Step 4 — Classify, score, and find hero dishes

**Classify** every item using the rules and glossary in the reference guide,
filtered through the user's dietary profile.

**Score** each restaurant's veg-friendliness on a 1–5 scale:

| Score | Verdict | Criteria |
|-------|---------|----------|
| ⭐⭐⭐⭐⭐ | 🟢 **Veg paradise** | 10+ veg options, dedicated veg mains, kitchen clearly veg-aware |
| ⭐⭐⭐⭐ | 🟢 **Great for veg** | 6–9 veg options including 2+ substantial mains |
| ⭐⭐⭐ | 🟡 **Workable** | 3–5 veg options, at least 1 real main (not just sides/salads) |
| ⭐⭐ | 🟠 **Slim pickings** | 1–2 veg options, mostly sides or salads |
| ⭐ | 🔴 **Rough** | 0–1 veg items, would need major modifications |

Factors that bump the score up:
- Dedicated vegetarian/vegan menu section
- Menu uses (V)/(VG) labels showing awareness
- Reviews mention veg accommodations positively
- Substantial mains, not just "salad and fries"

Factors that drop the score:
- Only sides and salads qualify
- Heavy reliance on "can be made veg" modifications
- Lots of "verify with restaurant" items (suggests low veg awareness)

**Identify hero dishes** — the 1–2 standout vegetarian items per restaurant.
These are dishes that:
- Are highlighted in reviews as especially good
- Are clearly designed as intentional vegetarian dishes, not afterthoughts
- Would excite a vegetarian, not just feed them

Mark hero dishes with ⭐ in the output. If no dish stands out, don't force it.

### Step 5 — Format and present

**Always start with a quick-scan summary table** so the user can immediately
see the landscape before diving into details:

```
## Quick scan — [Location] veg-friendly dining

| # | Restaurant | Cuisine | Verdict | Veg items | Hero dish |
|---|-----------|---------|---------|-----------|-----------|
| 1 | [Name]    | Italian | 🟢 Great | 8 items  | ⭐ Mushroom ragu |
| 2 | [Name]    | Thai    | 🟡 Workable | 4 items | ⭐ Green curry (verify fish sauce) |
| 3 | [Name]    | American | 🟠 Slim | 2 items  | — |

💡 Top pick for mixed groups: [Name] — [one-sentence reason]
```

**Then, for each restaurant in order, show the full breakdown:**

For **à la carte restaurants:**

```
---

### 1. [Restaurant Name] — 🟢 Great for veg
[Cuisine] | [Rating] ⭐ | [Price $/$$/$$$] | [Address]
🔗 [Menu link] | 📅 [Dinner / Brunch / Lunch]

**⭐ Hero dish:** [Dish name] ($XX) — [why it's special]

**Starters:**
- [Dish] ($XX) — [brief description] [Vegan]
- [Dish] ($XX) — [brief description]

**Mains:**
- [Dish] ($XX) — [brief description]
- [Dish] ($XX) — [brief description] ⚠️ Can be made veg: [modification]

**Sides:**
- [Dish] ($XX) — [brief description]

**Desserts:**
- [Dish] ($XX) — [brief description]

❓ **Ask your server:**
- "[Dish] — is the Caesar dressing made with anchovies? I don't eat fish."
- "[Dish] — is the risotto made with vegetable stock or chicken stock?"

📊 Veg count: X / Y total items

💡 **Dining tip:** [practical advice, e.g. "Reserve 2+ weeks out. Sit at the
bar for walk-ins. Ask about the off-menu veg pasta — reviews mention it."]
```

For **prix fixe / tasting menus:**

```
---

### 1. [Restaurant Name] — 🟡 Workable
[Cuisine] | [Rating] ⭐ | [Price $XXX/person] | [Address]
🔗 [Menu link] | Format: [X]-course prix fixe

**Vegetarian options by course:**

**Course 1 (choose one of 3):**
- [Veg option] — [description] ✅
- [X non-veg options also available]

**Course 2 (choose one of 4):**
- [Veg option] — [description] ✅

**Dessert:** All 3 options are vegetarian ✅

📊 Vegetarian choices in X of Y courses

💡 **Dining tip:** Call ahead and say: "I'm vegetarian — do you offer a
dedicated vegetarian tasting menu or can the chef make substitutions?"
Most serious kitchens will prepare a full vegetarian progression with
advance notice. Book at least [X] days ahead.
```

**Close with a final recommendation:**

```
---

## Bottom line

**For your mixed group, I'd go with [Restaurant Name].**
[2–3 sentences: why it's the best balance of great food for everyone,
strong veg options, and practical logistics (reservations, location, etc.)]

**Runner-up:** [Name] — [one sentence why]

📝 Menus sourced [month year]. Confirm current offerings with restaurants.
```

### Step 6 — Handle follow-ups

The user will often come back with follow-up questions. Handle these
without re-running the full workflow:

**"What about [restaurant name]?"** — If you already crawled it, pull from
your earlier data. If it's a new restaurant, just do Steps 3–5 for that one.

**"Any vegan options at [restaurant]?"** — Re-filter your existing data
through a vegan lens. No new searches needed.

**"Show me just the top 3"** — Re-rank by veg-friendliness score and present
the top 3 with their existing details.

**"What about [different neighborhood]?"** — Run Steps 2–5 for the new area.

**"Compare [Restaurant A] vs [Restaurant B]"** — Side-by-side comparison
using data you already have (or crawl any new ones).

**"I'm actually vegan, not just vegetarian"** — Re-filter all existing data
through vegan criteria. Note items that were vegetarian but not vegan.

**"What should I order?"** — Highlight the hero dishes and give a suggested
order: "Start with [starter], get [main] as your entrée, and if you have
room, [dessert] is vegan."

## Rules

1. **Never invent menu items.** Only list dishes confirmed from crawled data.
2. **Don't skip steakhouses or BBQ joints.** If they have 3+ veg options,
   include them.
3. **No fast food chains** unless explicitly asked.
4. **Don't bias toward vegetarian restaurants.** Search for the best
   restaurants period.
5. **Always show the menu source link** for verification.
6. **Flag ambiguous items** with specific server scripts, not just "verify."
7. **Handle web_fetch failures gracefully.** Never pretend you read a menu.
8. **Note menu freshness.** Menus change.
9. **Label meal periods.** Don't mix brunch and dinner without labeling.
10. **Use reviews to find hero dishes.** A dish that reviewers rave about is
    worth calling out, even if the menu description is plain.
11. **Respect the dietary profile.** A vegan user doesn't want to see cheese
    dishes. A flexitarian doesn't need fish sauce warnings.
12. **Give actionable server scripts.** Don't just say "verify" — give the
    actual question to ask, worded politely and specifically.

## Edge cases

**Single restaurant lookup:** Skip Step 2. Go straight to Steps 3–5.

**Vegan-only:** Follow same workflow but filter to vegan items only. The list
will be shorter — that's OK. Note items that *could* be vegan with minor
changes (e.g., "ask for no cheese").

**Pescatarian:** Include fish/seafood dishes as options. Skip fish sauce and
anchovy warnings since those are fine.

**"I'm not strict" / Flexitarian:** Include everything that's vegetarian, and
remove the "verify with restaurant" flags for minor ingredients like fish
sauce, broth, Worcestershire. Keep flags only for actual meat chunks.

**International location:** Workflow still applies. Note translation
ambiguities. Vegetarian terminology varies by country — search in the local
language too if needed.

**Comparing specific restaurants:** Skip discovery. Run Steps 3–5 for each
named restaurant and present a side-by-side comparison.

**User uploads a menu:** You can't extract text from images/PDFs via
web_fetch. Search for the restaurant's online menu instead.
