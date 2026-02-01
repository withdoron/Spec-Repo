# Recommendation System

> LocalLane's trust-first alternative to traditional star ratings.
> "Neighbors recommending neighbors — not strangers rating strangers."

---

## Status: 📋 SPEC COMPLETE — Ready for Phase 1 Build

Last updated: 2026-01-31

---

## Philosophy

Traditional review platforms are built around complaint culture. The loudest voices are the angriest. Star ratings invite gaming, create anxiety for business owners, and reduce nuanced human experiences to a number.

LocalLane is different. Our recommendation system is designed to:

- **Amplify positive community signal** — every interaction is a neighbor lifting up a local business
- **Route concerns constructively** — problems go through proper channels, not public shaming
- **Resist gaming** — real identity, public accountability, no anonymous attacks
- **Feel human** — stories and names, not numbers and algorithms

From the Blueprint:
> "Local Lane is a premium, trusted community platform."

Trust between neighbors doesn't come from a 4.2-star average. It comes from Sarah saying "I've used them, they're great."

---

## The Three Layers

### Layer 1: The Nod 👍

**What it is:** A one-click public recommendation. "I'd recommend this business."

**How it works:**
- Logged-in user clicks "Recommend" on a business profile
- Their name and avatar appear on the business as a recommender
- One Nod per user per business (toggle on/off)
- No text, no rating, no form — just your name behind it

**Why it matters:**
- Lowest friction → highest participation
- Public identity prevents fake Nods (your face is on it)
- Aggregate count ("23 neighbors recommend") is powerful social proof
- Can't be gamed without creating many real-looking accounts

**Anti-gaming:** Requires logged-in account with display name. One per user per business. Publicly visible — your name is attached.

---

### Layer 2: The Story 📖

**What it is:** A detailed, personal account of using this business. Not a "review" — a story.

**How it works:**
- User fills out a guided form (see Form Design below)
- Asks: what service, what happened, how it turned out
- Photos encouraged but optional
- Published with the user's real name and avatar
- No star rating anywhere in the flow

**Why it matters:**
- Specificity is hard to fake — detailed stories carry inherent credibility
- Stories are shareable content ("read what Sarah said about Mountain View Carpentry")
- Helps future customers understand real experiences
- Gives businesses meaningful, actionable feedback

**Anti-gaming:** Real name required. Guided questions make generic fake stories obvious. Stories are public and attributed.

---

### Layer 3: The Vouch ✋ (Phase 2)

**What it is:** A verified, weighted endorsement. "I personally used this business and I stand behind them."

**How it works:**
- Requires answering verification questions (what service, approximate date, would you hire again)
- Carries more weight than a Nod
- Vouches from business owners or long-time members carry extra weight
- Creates a trust web across the community

**Why it's Phase 2:**
- Needs enough platform activity to make verification meaningful
- Trust weighting algorithm needs real data to calibrate
- Core UX (Nod + Story) should be validated first

---

## What Replaces Star Ratings

### On BusinessCard (Directory, Search, CategoryPage)

**Before (stars):**
```
★★★★☆  4.2 (17 reviews)
```

**After (recommendations):**
```
👍 23 neighbors recommend · 4 stories
[avatar] [avatar] [avatar] +20
```

The avatar cluster shows real faces of real community members. This is more compelling than a number because it's human.

### On BusinessProfile

**Before:** Star breakdown chart (5★: 8, 4★: 5, 3★: 3...)

**After:** Recommendation summary card:
```
┌──────────────────────────────────────┐
│  👍 23 Neighbors Recommend           │
│  [avatar cluster of recommenders]    │
│                                      │
│  📖 4 Stories Shared                 │
│  "They built our treehouse and..."   │
│  "Best mechanic in Eugene..."        │
│                                      │
│  [Recommend This Business]           │
│  [Share Your Story]                  │
└──────────────────────────────────────┘
```

---

## The "Recommend" Page (replaces WriteReview)

### Route: `/recommend?businessId={id}`

When a user clicks "Recommend This Business" or "Share Your Story" from BusinessProfile, they land on this page.

### Two-Path Layout

```
┌──────────────────────────────────────────────┐
│  ← Back to {Business Name}                   │
│                                               │
│  How would you like to recommend              │
│  {Business Name}?                             │
│                                               │
│  ┌─────────────────┐  ┌─────────────────┐    │
│  │  👍 Quick Nod    │  │  📖 Share a     │    │
│  │                  │  │     Story        │    │
│  │  One click.      │  │                  │    │
│  │  Your name       │  │  Tell your       │    │
│  │  says it all.    │  │  neighbors what  │    │
│  │                  │  │  happened.       │    │
│  │  [Recommend]     │  │  [Start Writing] │    │
│  └─────────────────┘  └─────────────────┘    │
│                                               │
│  Already recommended? Your Nod is visible     │
│  on {Business Name}'s profile.                │
└──────────────────────────────────────────────┘
```

If the user has already Nodded, the left card shows a checkmark and "You recommend this business" with an option to remove it.

### Story Form (guided, not generic)

When user clicks "Start Writing":

```
┌──────────────────────────────────────────────┐
│  📖 Share Your Story about {Business Name}   │
│                                               │
│  What service or experience did you have?     │
│  [___________________________________________]│
│  e.g., "Kitchen remodel", "Oil change",       │
│  "Farm stand purchase"                        │
│                                               │
│  Tell your story *                            │
│  [___________________________________________]│
│  [                                           ]│
│  [                                           ]│
│  What happened? How did it go? Would you      │
│  go back?                                     │
│                                               │
│  Add photos (optional)                        │
│  [📷] [📷] [📷] [+]                          │
│                                               │
│  Your story will be published as:             │
│  [👤 Sarah M.] — visible to the community    │
│                                               │
│  [Share Your Story]                           │
└──────────────────────────────────────────────┘
```

Key design choices:
- **"What service"** replaces the generic "Review Title" — it contextualizes the story
- **"Tell your story"** replaces "Write your review" — framing matters
- **Identity reminder** at the bottom — "your name is on this" deters bad faith
- **No stars anywhere** — the entire flow is narrative, not numeric
- Submitting a Story also auto-creates a Nod (you obviously recommend them if you're writing a story)

---

## Handling Negative Experiences

### No Public Negative Reviews

LocalLane does not publish negative reviews. This is a deliberate design choice aligned with the platform's mission of "spreading good news and promoting a healthy community."

### "Flag a Concern" Flow

If someone has a negative experience, they can:

1. Click "Had a different experience?" link (subtle, below the recommend options)
2. This opens a private form that goes to LocalLane moderation
3. The form asks: what happened, when, what resolution would you like
4. LocalLane mediates between the customer and business
5. If a pattern emerges (multiple flags), LocalLane investigates

**Why this is better:**
- Protects businesses from bad-faith attacks
- Gives businesses a chance to make it right privately
- Patterns still surface to platform admins
- Mirrors how real community trust works — you talk to people, not post on a wall
- Removes the incentive for competitors to leave fake negative reviews

### Admin Visibility

Admins can see:
- Flag count per business (visible in Admin Panel)
- Flag details and resolution status
- Flagged businesses that need attention

---

## Data Model

### Recommendation Entity (replaces/extends Review)

```javascript
Recommendation {
  id: UUID,
  business_id: UUID,           // Which business
  user_id: UUID,               // Who recommended (required — no anonymous)
  user_name: string,           // Display name at time of recommendation
  user_avatar: string,         // Avatar URL at time of recommendation
  
  // Type
  type: 'nod' | 'story',      // Phase 2 adds 'vouch'
  
  // Story fields (null for nods)
  service_used: string,        // "What service did you use?"
  content: string,             // The story itself
  photos: string[],            // Photo URLs
  
  // Status
  is_active: boolean,          // Can be toggled off (un-Nod)
  
  // Timestamps
  created_date: Date,
  updated_date: Date,
}
```

### Concern Entity (new — for private flags)

```javascript
Concern {
  id: UUID,
  business_id: UUID,
  user_id: UUID,
  user_name: string,
  
  // Details
  description: string,         // What happened
  desired_resolution: string,  // What they'd like
  approximate_date: string,    // When it happened
  
  // Admin handling
  status: 'new' | 'reviewing' | 'resolved' | 'dismissed',
  admin_notes: string,
  resolved_date: Date,
  
  // Timestamps
  created_date: Date,
}
```

### Business Entity Changes

Replace rating fields with recommendation counts:

```javascript
// REMOVE (or deprecate):
average_rating: number,
review_count: number,

// ADD:
nod_count: number,             // Total active Nods
story_count: number,           // Total active Stories
recommendation_count: number,  // nod_count + story_count (denormalized)
concern_count: number,         // Admin-only visibility
```

### Migration Strategy

Existing Review records can be migrated:
- Reviews with rating 4-5 → convert to Story type recommendations
- Reviews with rating 1-3 → convert to Concerns (private)
- `rating` field becomes unused but doesn't need deletion
- `average_rating` on Business can be kept temporarily for sorting fallback

---

## Trust Signal on BusinessCard

### Current BusinessCard shows:
```
★★★★☆  4.2 (17 reviews)
```

### New BusinessCard shows:
```
👍 23 recommended · 4 stories
```

Or if no recommendations yet:
```
New to LocalLane
```

### Implementation

Replace `StarRating` + rating text with a `TrustSignal` component:

```jsx
// TrustSignal component
<div className="flex items-center gap-2 text-sm">
  <ThumbsUp className="h-4 w-4 text-amber-500" />
  <span className="text-slate-300">
    {nod_count + story_count} recommended
  </span>
  {story_count > 0 && (
    <>
      <span className="text-slate-600">·</span>
      <span className="text-slate-400">{story_count} stories</span>
    </>
  )}
</div>
```

---

## Affected Files

### Pages to Rebuild
- `src/pages/WriteReview.jsx` → rename to `Recommend.jsx` (new two-path layout)
- `src/pages/BusinessProfile.jsx` → replace reviews tab with recommendations tab

### Components to Create
- `src/components/recommendations/TrustSignal.jsx` — replaces StarRating on cards
- `src/components/recommendations/RecommendationCard.jsx` — replaces ReviewCard
- `src/components/recommendations/NodAvatars.jsx` — avatar cluster display
- `src/components/recommendations/StoryCard.jsx` — story display
- `src/components/recommendations/ConcernForm.jsx` — private flag form (Phase 1.5)

### Components to Update
- `src/components/business/BusinessCard.jsx` — swap StarRating for TrustSignal
- `src/components/reviews/StarRating.jsx` — keep for now (used in legacy data), deprecate later

### Routes to Update
- WriteReview route → Recommend route
- BusinessProfile "Write a Review" link → "Recommend" link

---

## Rollout Plan

### Phase 1 (Build Now)
- [ ] Create Recommendation entity in Base44
- [ ] Build Recommend page with Nod + Story paths
- [ ] Build TrustSignal component
- [ ] Build RecommendationCard / StoryCard components
- [ ] Update BusinessProfile recommendations tab
- [ ] Update BusinessCard to show TrustSignal
- [ ] Update Business entity with recommendation counts
- [ ] Dark theme everything from the start

### Phase 1.5 (Soon After)
- [ ] Build ConcernForm (private flag)
- [ ] Add "Had a different experience?" link
- [ ] Admin view for concerns
- [ ] Concern resolution workflow

### Phase 2 (Later)
- [ ] Vouch system with verification questions
- [ ] Trust weighting (voucher reputation)
- [ ] Business owner responses to stories
- [ ] Notification system (new recommendations)
- [ ] "Recommended by" feed/activity stream

---

## Design Notes

### Tone
- "Recommend" not "Review"
- "Story" not "Review"
- "Neighbors" not "Users"
- "Had a different experience?" not "Write a negative review"

### Colors (Gold Standard)
- Nod/recommend icon: `text-amber-500` (ThumbsUp)
- Story icon: `text-amber-500` (BookOpen or similar)
- Avatar cluster: real photos on `bg-slate-800` circles
- Concern link: `text-slate-500` (subtle, not prominent)

### Empty States
- No recommendations yet: "Be the first to recommend {business name}"
- Business is new: "New to LocalLane" badge (neutral, not negative)

---

## Decision Record

See [DECISIONS.md](./DECISIONS.md#dec-021-replace-star-ratings-with-recommendation-system) — DEC-021.

---

*This spec is part of the LocalLane Spec-Repo. Reference before implementing.*
