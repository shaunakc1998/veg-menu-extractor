# 🌱 Veg Menu Extractor — Claude Skill

**Find the best restaurants anywhere, then extract only the vegetarian options from their menus.**

Built for mixed dining groups where one person is vegetarian. Instead of searching for "vegetarian restaurants" (which limits everyone), this skill finds the *best restaurants period* — Italian, steakhouse, Thai, whatever — and pulls out exactly what the vegetarian person can eat.

Uses only Claude's built-in `web_search` and `web_fetch`. No API keys, no paid services, completely free.

---

## What it does

You say:
> "I'm vegetarian and dining with meat-eating colleagues in the West Village, NYC. Find upscale Italian places where I'll have more than just a salad."

You get:

| # | Restaurant | Cuisine | Verdict | Veg items | Hero dish |
|---|-----------|---------|---------|-----------|-----------|
| 1 | L'Artusi | Italian | 🟢 Great (4/5) | 8 items | ⭐ Mushroom Ragu |
| 2 | Via Carota | Italian | 🟢 Veg paradise (5/5) | 12 items | ⭐ Cacio e Pepe |
| 3 | Rosemary's | Italian | 🟢 Great (4/5) | 7 items | ⭐ Truffle Focaccia |

...followed by full per-restaurant breakdowns with categorized dishes, prices, hero dish highlights, server scripts for ambiguous items, reservation tips, and a bottom-line recommendation.

## Key features

### 🚦 Quick verdicts
Every restaurant gets a veg-friendliness score (1–5) with an emoji: 🟢 Great → 🟡 Workable → 🟠 Slim → 🔴 Rough. Scan the summary table in 5 seconds, then drill into whichever restaurants interest you.

### 🍽️ Hero dish highlights
Not all veg options are created equal. The skill identifies the 1–2 dishes that a vegetarian would actually be *excited* about — pulled from reviews, menu descriptions, and food writing — and marks them with ⭐.

### 🗣️ "What to say to your server" scripts
Instead of just "verify with restaurant," you get the exact sentence to say:
> *"Is the Caesar dressing made with anchovies? I don't eat fish."*
> *"What kind of stock is the risotto made with? I'm looking for vegetable-based."*

### 🥗 Dietary profile support
"Vegetarian" means different things to different people. The skill handles:
- **Lacto-ovo vegetarian** (default)
- **Vegan** — filters out dairy, eggs, honey
- **Pescatarian** — includes fish, skips fish sauce warnings
- **Flexitarian** — "fish sauce is fine, just no meat"
- **Egg-free vegetarian**
- **Jain vegetarian** — no root vegetables, no eggs
- **Custom** — "I don't eat meat but oyster sauce is OK"

### 🔍 Hidden ingredient detection
Catches the stuff most people miss on menus:
- **Guanciale** (cured pork jowl) hiding in pasta dishes
- **Anchovy** in Caesar dressing and puttanesca
- **Dashi** (fish stock) in miso soup
- **Fish sauce** in Thai/Vietnamese dishes
- **Lard** in refried beans and tamales
- 30+ hidden non-veg ingredients in the glossary

### 💬 Smart follow-ups
After the initial results, you can ask:
- "What about that second restaurant?" — pulls from existing data
- "Any vegan options at L'Artusi?" — re-filters without re-searching
- "Show me just the top 3" — re-ranks by score
- "Compare Restaurant A vs B" — side-by-side
- "What should I order?" — suggested order from hero dishes

### 🍳 Prix fixe / tasting menu support
Fine dining restaurants get a course-by-course breakdown instead of being forced into Starters/Mains/Sides. Includes a tip to call ahead for vegetarian tasting menu accommodations.

---

## Installation

### Option 1: Download the `.skill` file
1. Go to [Releases](../../releases) and download `veg-menu-extractor.skill`
2. In Claude.ai, go to **Settings → Skills**
3. Click **Add Skill** and upload the `.skill` file

### Option 2: Manual install
1. Clone this repo
2. Copy the `veg-menu-extractor/` folder to your Claude skills directory

---

## Skill structure

```
veg-menu-extractor/
├── SKILL.md                           # Main playbook (317 lines)
│   ├── Workflow (6 steps)
│   ├── Output templates (à la carte + prix fixe)
│   ├── 12 rules
│   ├── Edge case handling
│   └── Follow-up handling
└── references/
    └── search-and-classify.md         # Deep reference (426 lines)
        ├── Search query templates
        ├── Menu crawling source priority
        ├── Classification rules (5 categories)
        ├── Dietary profile adjustments (6 profiles)
        ├── Hidden ingredient glossary (30+ terms)
        ├── Hero dish identification guide
        ├── Server script templates
        ├── Scoring system
        └── Failure mode handling
```

---

## Example prompts

**Broad city search:**
> "What are the best restaurants in Savannah, GA with good vegetarian options?"

**Mixed group planning:**
> "Planning dinner for 6 people in Chicago — 2 are vegetarian. Find somewhere everyone will love. Mid-range, any cuisine."

**Specific constraints:**
> "I'm vegan, visiting Portland next week. Find dinner spots with actual vegan mains, not just salads. Fine dining OK."

**Single restaurant:**
> "What vegetarian options does Canlis in Seattle have?"

**Comparison:**
> "Compare L'Artusi vs Via Carota vs Rosemary's for a vegetarian dining in West Village NYC."

**Follow-up:**
> "Actually I'm vegan — re-filter those results."

---

## How it works

1. **Discover** — Runs 3–5 web searches for top-rated restaurants (any cuisine, not biased toward veg)
2. **Crawl** — Fetches actual menus from restaurant websites, Yelp, or delivery platforms
3. **Classify** — Applies vegetarian classification rules with a 30-term hidden ingredient glossary
4. **Score** — Rates each restaurant 1–5 for veg-friendliness
5. **Highlight** — Identifies hero dishes from reviews and menu descriptions
6. **Present** — Quick-scan table → detailed breakdowns → server scripts → bottom-line recommendation

---

## Known limitations

- **PDF / image menus** can't be parsed by `web_fetch`. Falls back to Yelp or delivery platforms.
- **JavaScript-heavy restaurant sites** sometimes return blank. Yelp is a reliable fallback.
- **Menu freshness** — menus change. The skill notes the source date and recommends confirming.
- **International menus** — classification accuracy drops in non-English languages.

---

## Contributing

Found a hidden non-veg ingredient that should be in the glossary? A menu crawling pattern that works better? Open a PR or issue.

---

## License

MIT — use it however you want.
