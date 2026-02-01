# Meal Planning & Grocery Integration — Feature Concept

> Helping families eat well using what's local, seasonal, and on sale.
> A future member-tier feature connecting recipes, farmers markets, and local grocery.

---

## Status: 🔮 VISION CONCEPT — Captured for Future Development

Last updated: 2026-02-01

---

## The Idea

Doron's insight: "I have a set of recipes I essentially utilize and slowly add more to it as my cooking expands or the seasons change. It would be great to have a meal planning tool by selecting what meals I want for the week, or have it help me select based on what's at the farmers market, or what's on sale."

This isn't a generic recipe app. It's a **community-connected meal planning tool** that ties into LocalLane's unique strengths:

- **Farmers market integration** — What's available this Saturday? Build meals around it.
- **Sale ad scanning** — What's on sale at Safeway this week? Plan budget-friendly meals.
- **Local source prioritization** — "This recipe needs tomatoes. Mountain View Farms has them Saturday, or they're $2.49/lb at Safeway."
- **Seasonal awareness** — Suggest recipes that use what's in season locally.

---

## Why It Belongs on LocalLane

This feature is uniquely powerful on LocalLane because of the existing infrastructure:

| LocalLane Asset | How Meal Planning Uses It |
|----------------|--------------------------|
| **Market Basket** (member feature) | Pre-order ingredients from farmers market vendors |
| **Business Directory** | Link to local farms, butchers, specialty food shops |
| **Events** | "Mountain View Farms at Saturday Market" — know what's available |
| **Trust/Recommendations** | "47 neighbors recommend this vendor's produce" |
| **Punch Pass** | Use punches at participating food vendors |

No other meal planning tool can say "your neighbor's farm has fresh basil this week, and 23 people in your community recommend them." That's the LocalLane advantage.

---

## Core Features (Conceptual)

### My Recipe Book

Personal recipe collection that grows over time.

- **Add recipes** — manual entry, URL import (parse recipe from a link), or photo scan
- **Tag recipes** — cuisine type, season, prep time, dietary needs, "weeknight easy," etc.
- **Ingredient lists** — structured ingredients that map to shopping lists
- **Notes** — personal modifications, family preferences ("kids like this with extra cheese")

This is the foundation. Everything else builds on having a personal recipe library.

### Weekly Meal Planner

Plan meals for the week, generate a unified shopping list.

- **Calendar view** — drag recipes into Mon-Sun slots (dinner, lunch, etc.)
- **Smart suggestions** — "You haven't used this recipe in a while" or "This uses similar ingredients to what you're already buying"
- **Auto-generate shopping list** — combine ingredients across all planned meals, deduplicate, organize by store section
- **Serving adjustment** — "This recipe serves 4, I need 6"

### Farmers Market Integration

**Ties directly into Market Basket.**

- **Saturday preview** — See what vendors are listing for the upcoming market
- **Recipe suggestions** — "Fresh peaches are available — here are 3 recipes from your book that use peaches"
- **Market Basket auto-fill** — Select the market ingredients you want, pre-order through Market Basket
- **Seasonal discovery** — "It's August — your neighbors are buying corn, tomatoes, and basil. Try this recipe."

### Sale Ad Integration

**The Safeway scanner concept.**

- **PDF/image scanning** — Upload or link to weekly sale ads from local grocery stores
- **AI extraction** — Parse sale items, prices, dates from the ad
- **Match to recipes** — "Chicken thighs are $1.99/lb this week — here are 5 recipes from your book"
- **Budget planning** — "Your meal plan this week would cost ~$85. If you swap Tuesday's salmon for the chicken on sale, it drops to ~$62."
- **Multi-store comparison** — "Avocados are $1.29 at Safeway, but the farmers market has them for $1.50 with 12 neighbor recommendations"

### Smart Ingredient Sourcing

The magic feature — where to buy what.

```
Your Shopping List for This Week:
─────────────────────────────────
🌲 Mountain View Farms (Saturday Market)
   ☐ Tomatoes (heirloom, 2 lbs)  — $4.00
   ☐ Fresh basil (1 bunch)        — $2.50
   ☐ Zucchini (3 medium)          — $3.00
   Pre-order via Market Basket? [Yes]

🏪 Safeway (On Sale This Week)
   ☐ Chicken thighs (3 lbs)       — $5.97 (SALE)
   ☐ Pasta (1 lb)                 — $1.29
   ☐ Olive oil (if needed)        — $6.99

🏠 Bob's Butcher Shop (Locally Owned)
   ☐ Ground beef (2 lbs)          — $9.98
   47 neighbors recommend
   
Estimated Total: $33.73
```

This is the north star. A shopping list that's organized by *where to buy it*, prioritizing local sources, showing trust signals, and integrating sale prices.

---

## User Tier Placement

| Component | Free | Member |
|-----------|------|--------|
| My Recipe Book (basic — manual add, 10 recipes) | ✅ | ✅ |
| My Recipe Book (full — unlimited, URL import, photo scan) | ❌ | ✅ |
| Weekly Meal Planner | ❌ | ✅ |
| Farmers Market integration | ❌ | ✅ |
| Sale Ad scanning | ❌ | ✅ |
| Smart Ingredient Sourcing | ❌ | ✅ |
| Shopping List (basic — from single recipe) | ✅ | ✅ |
| Shopping List (combined — from meal plan) | ❌ | ✅ |

Free users get a taste — they can save a few recipes and generate a shopping list from one. Member users get the full planning + sourcing experience.

---

## Technical Considerations (High Level)

### Recipe Data Model

```javascript
Recipe {
  id: UUID,
  user_id: UUID,
  title: string,
  description: string,
  source_url: string | null,      // Where it came from (if imported)
  servings: number,
  prep_time_minutes: number,
  cook_time_minutes: number,
  ingredients: [
    { name: string, quantity: number, unit: string, section: string }
  ],
  instructions: string[],
  tags: string[],                 // 'weeknight', 'summer', 'kid-friendly', etc.
  season: string[],               // 'spring', 'summer', 'fall', 'winter'
  photos: string[],
  notes: string,
  last_planned: date,             // When it was last on the meal plan
  times_planned: number,          // How often it's been used
}
```

### Ingredient Matching Challenge

Matching recipe ingredients ("2 large tomatoes") to store items ("Heirloom Tomatoes, per lb") requires fuzzy matching and unit conversion. This is a known hard problem but solvable with:
- Ingredient normalization database
- AI-assisted matching for unusual items
- User correction/confirmation that improves over time

### Sale Ad Parsing

Grocery sale ads (Safeway, WinCo, etc.) are typically PDF flyers or images. Parsing requires:
- OCR / PDF text extraction
- Item name + price extraction
- Sale date range identification
- Categorization (produce, meat, dairy, etc.)

This could be done with AI vision models reading the ad images. Claude API could handle this as a member feature.

---

## What This Is NOT

- **Not a full cooking app** — No step-by-step cooking mode, no video integration
- **Not a diet/nutrition tracker** — No calorie counting, no macro tracking
- **Not a delivery service** — Shopping lists tell you where to go, they don't deliver
- **Not a national feature** — This only works because LocalLane knows your local vendors and markets

---

## Dependencies

| Dependency | Status | Required For |
|------------|--------|-------------|
| Market Basket | 🔮 Future (see USER-TIERS.md) | Farmers market pre-ordering |
| Stripe Connect | 📋 Spec'd | Payment for market orders |
| Business inventory listings | 🔮 Future | Vendors listing available items |
| User tiers | 🔮 Future | Free vs Member feature gating |
| AI/Claude API integration | 🔮 Future | Recipe URL parsing, sale ad scanning, ingredient matching |

---

## Build Sequence (When the Time Comes)

1. **My Recipe Book** — Personal recipe storage (standalone value even without other features)
2. **Weekly Meal Planner** — Calendar + shopping list generation
3. **Farmers Market Integration** — Connect to Market Basket vendor listings
4. **Sale Ad Scanning** — AI-powered weekly ad parsing
5. **Smart Ingredient Sourcing** — The unified "where to buy" shopping list

Each phase delivers standalone value. You don't need sale ad scanning to have a useful meal planner.

---

## Competitive Landscape

| Existing Tool | What It Does | What It Lacks |
|--------------|-------------|---------------|
| Mealime, Paprika | Recipe + meal planning | No local sourcing, no community |
| Instacart, Amazon Fresh | Grocery delivery | Not local-first, no farmers market |
| Flipp | Sale ad aggregation | No recipe connection, no local farms |
| Eat This Much | AI meal planning | No local ingredient sourcing |

**LocalLane's edge:** The only tool that combines meal planning with hyper-local sourcing, community trust signals, and farmers market integration. This can't be replicated by a national app.

---

*This concept is part of the LocalLane Spec-Repo. Captured for future development.*
