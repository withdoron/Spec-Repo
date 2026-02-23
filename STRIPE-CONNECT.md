# STRIPE-CONNECT.md — Stripe Connect Integration Spec

> Defines the Stripe infrastructure for LocalLane payments.
> Revised: 2026-02-02 — Aligned with Community Pass membership model (DEC-028)
> Status: Spec — Not yet implemented

---

## Overview

LocalLane uses **Stripe** for three distinct payment flows:

1. **Business Tier Subscriptions** — Businesses pay LocalLane monthly for platform access (Stripe Billing)
2. **Community Pass Memberships** — Users pay LocalLane monthly for community access (Stripe Billing)
3. **Direct Purchases** — Users pay businesses directly for events/products (Stripe Connect with `on_behalf_of`)

The key architectural insight: LocalLane collects subscription revenue and distributes revenue share to businesses from its own operating income. Users never pay businesses through LocalLane — they either use membership access tokens (Joy Coins) or purchase directly from the business.

### What Changed (DEC-028)

This spec was originally built around a stored-value credit model (Separate Charges and Transfers). Legal research identified that model as money transmission under Oregon ORS 717. The Community Pass membership model eliminates that risk. See DECISIONS.md DEC-028 for the full decision record.

| Old Model | New Model |
|-----------|-----------|
| Separate Charges and Transfers | Billing (subscriptions) + Connect (`on_behalf_of` for direct) |
| User buys credits, redeems at businesses | User subscribes, receives access tokens |
| 15% platform fee per redemption | Platform retains portion of subscription pool |
| Per-coin fixed dollar value | Floating pool-based value per Joy Coin |
| 12-month coin expiration | Monthly expiration with small rollover |
| LocalLane as merchant of record for all | Business is merchant of record for direct purchases |

---

## Payment Flow 1: Business Tier Subscriptions

Businesses pay LocalLane for platform access. This is straightforward Stripe Billing — no connected accounts or transfers involved.

### Stripe Products Needed

| Product | Monthly Price | Notes |
|---------|--------------|-------|
| LocalLane Basic | Free | Tier 1 — listing, events (reviewed), directory presence. No Stripe product needed. |
| LocalLane Standard | $79/mo | Tier 2 — auto-publish, Community Pass pool, analytics, business network |
| LocalLane Partner | $149/mo | Tier 3 — earned status, own node, enhanced features. Admin-initiated. |

**Founding business deal:** First 3 months free at Standard tier. Implement as a 100% off coupon with 3-month duration in Stripe, applied at checkout.

### Subscription Lifecycle

| Event | Stripe Webhook | LocalLane Action |
|-------|---------------|-----------------|
| Business subscribes | `checkout.session.completed` | Set `subscription_tier` on Business entity |
| Payment succeeds | `invoice.paid` | No action — subscription continues |
| Payment fails | `invoice.payment_failed` | Email business owner, start grace period |
| Subscription canceled | `customer.subscription.deleted` | Downgrade to `basic`, disable paid features |

### Implementation

1. Create Stripe Products and Prices in Stripe Dashboard
2. Build upgrade flow: button in Business Dashboard → Stripe Checkout Session → redirect back
3. Handle webhooks for lifecycle events
4. Update `subscription_tier` field on Business entity based on webhook events

---

## Payment Flow 2: Community Pass Memberships

Users subscribe to LocalLane's community access program. Also Stripe Billing — user pays LocalLane, not individual businesses.

### Stripe Products Needed

| Product | Monthly Price | Joy Coins | Notes |
|---------|--------------|-----------|-------|
| Community Pass Base | $49/mo | 12 | Base household membership |
| Community Pass Additional | $39/mo | 12 | Add-on coin allocation (subscription add-on) |
| Community Pass Explorer | TBD | TBD | Summer seasonal — design after base proves out |

**Note:** Memberships are coin allocations, not person assignments. A household buys a base and as many additional allocations as they want. Family shares one pool.

**Implementation:** Base is a standalone subscription. Additional allocations are subscription items added to the same subscription. Stripe handles this natively with multi-item subscriptions.

### Subscription Lifecycle

| Event | Stripe Webhook | LocalLane Action |
|-------|---------------|-----------------|
| User subscribes | `checkout.session.completed` | Create CommunityPass entity with monthly allocation |
| Monthly renewal | `invoice.paid` | Reset coin balance to allocation, unused flow to scholarship |
| Payment fails | `invoice.payment_failed` | Grace period — coins frozen but not deleted |
| Subscription canceled | `customer.subscription.deleted` | Expire remaining coins, update user status |

### Joy Coin Management

- Joy Coins are movement tokens within the membership, not stored value
- Monthly allocation set by number of subscription items (12 coins each)
- Household shares one coin pool — no individual assignments
- Unused coins at month's end flow to scholarship pool (not rollover)
- No cash value, no refund on unused coins
- Businesses set their own coin price (1-3 coins per activity)

### CommunityPass Entity (Replaces PunchPass)

| Field | Purpose |
|-------|---------|
| `coins_remaining` | Current Joy Coin balance for the household |
| `coins_allocated` | Total monthly allocation (12 per subscription item) |
| `membership_type` | `base` or `additional` |
| `household_id` | Links all allocations for one household |
| `stripe_subscription_id` | Stripe Subscription ID |
| `subscription_status` | active / past_due / canceled |
| `billing_cycle_anchor` | Date monthly allocation resets |
| `coins_to_scholarship` | Unused coins flowing to scholarship pool at month end |

**Legacy fields to remove:**
- `per_punch_value` — no longer applicable (pool-based)
- `pack_type` — replaced by coin allocation model
- `stripe_checkout_session_id` — replaced by subscription ID

### Monthly Revenue Share Calculation

At month's end, LocalLane calculates business payouts:

1. Total net membership revenue (after Stripe fees) for the month
2. 75% allocated to business pool
3. Pool distributed proportionally by scan volume

**Example with 100 members generating $8,000 net:**
- Business pool: $6,000
- Business A received 500 scans (25% of total scans) → $1,500
- Business B received 300 scans (15%) → $900
- Business C received 200 scans (10%) → $600
- Remaining businesses share the rest proportionally

**Payout method:** Stripe Connect transfers to connected accounts. Monthly cycle.

**Note:** Revenue share agreements must be in place before first payout. See legal checklist.

---

## Payment Flow 3: Direct Purchases

Users without a Community Pass — or buying something not covered — purchase directly from the business. The business is the merchant of record.

### Stripe Connect Model: `on_behalf_of`

Unlike the old Separate Charges and Transfers model, direct purchases use `on_behalf_of`:

- Payment goes directly to the business's connected Stripe account
- Business is merchant of record (their name on receipt)
- LocalLane facilitates the UI but never holds the funds
- **Zero commission** — the business already pays tier fees for platform access

This completely eliminates money transmission concerns for direct purchases. LocalLane is a software platform that helps businesses sell, not an intermediary handling funds.

### Why Not Separate Charges and Transfers?

The old spec used SC&T because stored-value credits were bought from LocalLane and redeemed later at unknown businesses. Under the Community Pass model, that flow is handled by the subscription + revenue share pool (not Stripe transfers per transaction). Direct purchases are simple: user pays business. `on_behalf_of` is the correct model.

### Direct Purchase Flow

1. User sees event/product: "Saturday Pottery Workshop — $35 or 3 Joy Coins"
2. User taps "Buy Now — $35"
3. LocalLane creates Stripe Checkout Session with `on_behalf_of: business.stripe_account_id`
4. User pays on Stripe-hosted checkout
5. Business receives $35 minus Stripe processing fee
6. LocalLane receives nothing from this transaction

### Dual Path UX

Every Community Pass offering shows two options:
- **"Use Joy Coins"** → Deduct from Joy Coins balance, create check-in record
- **"Buy Now — $XX"** → Stripe Checkout to business directly

Non-Community-Pass offerings show only the direct purchase option.

---

## Revenue Share Distribution

Community Pass membership revenue is LocalLane's subscription income. A portion of that revenue is distributed to participating businesses based on how many passholder visits they received.

### How It Works

1. Each month, LocalLane calculates total Community Pass membership revenue
2. A defined portion goes to the business revenue-share pool
3. Each business's share is proportional to their check-in count vs. total check-ins
4. LocalLane creates Stripe Transfers from its own account to each business's connected account
5. This is a B2B payment from LocalLane to the business — not money transmission

### Key Distinction

This is NOT user-to-business money transmission. This is:
- LocalLane collecting subscription revenue (its own income)
- LocalLane paying businesses from its operating revenue (B2B payment)
- Similar to how a gym chain pays individual locations from corporate membership fees

### Settlement Schedule

| Schedule | When | Notes |
|----------|------|-------|
| Monthly | End of billing month | Default — simplest, lowest Stripe transfer fees |
| Weekly | Every Monday | Potential Partner perk for faster access to funds |
| Manual | Admin triggers | Testing, special cases, dispute resolution |

Start with monthly. Add weekly as an option later if businesses request faster access.

### Settlement Data Model

```javascript
// PunchPassUsage (already exists — add fields)
{
  // Existing fields...
  settled: boolean,               // Has this usage been included in a settlement?
  settlement_id: string | null,   // Reference to PunchPassTransaction
}

// PunchPassTransaction (repurpose for settlements)
{
  id: UUID,
  business_id: UUID,
  type: 'revenue_share',          // Changed from 'settlement'
  period_start: ISO datetime,
  period_end: ISO datetime,
  total_checkins: number,          // Check-ins at this business in period
  share_amount: number,            // Amount transferred (cents)
  stripe_transfer_id: string,      // 'tr_xxx'
  status: 'pending' | 'completed' | 'failed',
  created_date: ISO datetime,
}
```

### Revenue Share Visibility

Businesses see in their dashboard:
- Monthly check-in count from Community Pass members
- Revenue share received (after transfer completes)
- Pending revenue share for current month (estimate)

Businesses do NOT see:
- Per-coin pool value (creates anxiety if it fluctuates)
- Other businesses' check-in counts or shares
- Total pool size

---

## Business Onboarding (Connect Onboarding)

**This section is largely unchanged from the original spec.** Stripe Connect onboarding for businesses is the same regardless of charge model.

### Onboarding Flow

1. Business owner clicks "Set Up Payments" in their dashboard
2. LocalLane creates a connected account via Stripe API
3. Stripe generates an Account Link (hosted onboarding URL)
4. Business owner redirected to Stripe's hosted onboarding
5. They enter: legal business name, EIN or SSN, bank account, identity verification
6. Stripe redirects back to LocalLane dashboard
7. LocalLane stores the connected account ID on the Business entity
8. Business can now receive direct purchases and revenue share payouts

### Business Entity Fields

```javascript
{
  stripe_account_id: string | null,     // 'acct_xxx'
  stripe_onboarding_complete: boolean,
  stripe_payouts_enabled: boolean,
  stripe_charges_enabled: boolean,
}
```

### Onboarding States

| State | Dashboard Shows |
|-------|----------------|
| No connected account | "Set Up Payments" button |
| Account created, onboarding incomplete | "Complete Payment Setup" button |
| Onboarding complete, payouts disabled | "Verification Pending" badge |
| Fully active | Green "Payments Active" badge |

### Tier Gating

| Tier | Direct Purchases | Community Pass Revenue Share |
|------|-----------------|----------------------------|
| Basic (Tier 1) | ❌ | ❌ |
| Standard (Tier 2) | ✅ | ✅ (must opt in to Community Pass network) |
| Partner (Tier 3) | ✅ | ✅ (included) |

Standard and Partner businesses must complete Stripe Connect onboarding to receive either direct payments or revenue share.

---

## Webhook Handling

Stripe sends webhooks for payment events. LocalLane needs a Base44 server function to process these.

### Required Webhooks

| Webhook Event | When It Fires | LocalLane Action |
|--------------|--------------|-----------------|
| `checkout.session.completed` | Tier subscription started OR Community Pass purchased OR Direct purchase completed | Route by metadata: update tier, create CommunityPass, or confirm purchase |
| `invoice.paid` | Recurring subscription payment succeeds | Log payment, maintain active status |
| `invoice.payment_failed` | Subscription payment fails | Email owner, start grace period |
| `customer.subscription.deleted` | Subscription canceled | Downgrade tier OR expire punches (by metadata) |
| `account.updated` | Connected account status changes | Update `stripe_payouts_enabled` on Business |
| `transfer.created` | Revenue share transfer initiated | Update settlement record status |
| `transfer.failed` | Revenue share transfer failed | Alert admin, retry logic |

### Webhook Implementation

```javascript
// functions/stripeWebhook.ts (Base44 server function)
// Receives Stripe webhook events
// Verifies webhook signature
// Routes to handler based on event type and metadata
// Metadata distinguishes: tier_subscription | community_pass | direct_purchase
```

---

## UI Changes

### Community Pass Page (User-Facing)

Replace the current demo mode with real Community Pass subscription:

```
Current (demo):  "Purchase" button → toast("Coming soon!")
New (real):      "Subscribe" button → Stripe Checkout (subscription) → CommunityPass created
```

Show membership tiers, monthly allocation, and "what you get" for each level. Include direct link to "What's Open" (Community Pass offerings in the community).

### Business Dashboard — Community Pass Section

New section replacing the old FinancialWidget concept:

| Section | Content |
|---------|---------|
| **Community Pass Status** | Opted in? Which offerings are designated? |
| **Check-in Activity** | Monthly check-in count from passholders |
| **Revenue Share** | Current month estimate, past month actual, payment history |
| **Payment Setup** | Stripe onboarding status |

### Admin Panel — Revenue Share Dashboard

New admin section:

| View | Content |
|------|---------|
| **Pool Overview** | Current month membership revenue, pool size, businesses participating |
| **Distribution Preview** | What each business would receive if settled today |
| **Settlement History** | Past distributions with business, amount, status |
| **Manual Settlement** | Button to trigger distribution |
| **Failed Transfers** | List of failed transfers with retry option |

---

## Existing Code Touchpoints

### What Already Exists

| Component/Entity | Status | Changes Needed |
|-----------------|--------|----------------|
| `@stripe/react-stripe-js` | ✅ Installed | Use for Checkout redirect |
| `@stripe/stripe-js` | ✅ Installed | Use for `loadStripe()` |
| `JoyCoins` entity (formerly PunchPass) | ✅ Exists (demo mode) | Add subscription fields, remove `per_punch_value` |
| `JoyCoinTransactions` (formerly PunchPassUsage) | ✅ Exists | Add `settled`, `settlement_id` |
| Revenue share entity (formerly PunchPassTransaction) | ✅ Exists | Repurpose for revenue share records |
| `PunchPass.jsx` page | ✅ Exists (demo mode) | Replace with Community Pass subscription flow |
| `FinancialWidget.jsx` | ✅ Exists (placeholder) | Replace with Community Pass section |
| `useOrganization()` hook | ✅ Exists | Already has `canUseJoyCoins` flag |
| `CheckIn.jsx` | ✅ Exists | No changes for MVP — Joy Coin deduction already works |
| `validateJoyCoins` function | ✅ Exists | No changes for MVP |

### What Needs to Be Created

| Component | Type | Purpose |
|-----------|------|---------|
| `functions/stripeWebhook.ts` | Base44 server function | Process all Stripe webhook events |
| `functions/createCheckoutSession.ts` | Base44 server function | Create Checkout for subscriptions and direct purchases |
| `functions/createConnectAccount.ts` | Base44 server function | Create connected account for business |
| `functions/createAccountLink.ts` | Base44 server function | Generate Stripe onboarding link |
| `functions/calculateRevenueShare.ts` | Base44 server function | Monthly pool calculation and distribution |
| `CommunityPassPage.jsx` | React component | Replaces PunchPass.jsx with subscription flow |
| `PaymentSetup.jsx` | React component | Stripe Connect onboarding UI in dashboard |
| `CommunityPassDashboard.jsx` | React component | Business-side check-in and revenue share view |
| `RevenueShareAdmin.jsx` | React component | Admin settlement management |

---

## Implementation Phases

### Phase A: Platform Setup (No Code)

**Goal:** Get Stripe account configured.

1. Create/rename Stripe account to "LocalLane"
2. Enable Stripe Connect in dashboard
3. Complete platform profile
4. Set up webhook endpoint URL
5. Create Stripe Products and Prices for business tier subscriptions
6. Store API keys in Base44 environment variables

### Phase B: Business Tier Subscriptions

**Goal:** Businesses can subscribe and pay for their tier.

1. Create `createCheckoutSession` for tier subscriptions
2. Create `stripeWebhook` handler (handle `checkout.session.completed` for tiers)
3. Build upgrade flow in Business Dashboard
4. Handle subscription lifecycle webhooks (paid, failed, canceled)
5. Update `subscription_tier` based on webhook events

**This phase makes business subscription revenue flow.**

### Phase C: Business Connect Onboarding

**Goal:** Businesses can link bank accounts for receiving payments.

1. Create `createConnectAccount` server function
2. Create `createAccountLink` server function
3. Add Stripe fields to Business entity
4. Build `PaymentSetup.jsx` component
5. Handle `account.updated` webhook

**This phase connects businesses to Stripe for future payments.**

### Phase D: Community Pass Subscriptions

**Goal:** Users can subscribe to Community Pass.

1. Create Stripe Products for Community Pass tiers
2. Build `CommunityPassPage.jsx` with subscription selection
3. Create checkout flow for Community Pass subscription
4. Handle subscription webhooks for punch allocation
5. Build monthly reset logic (new allocation, expire unused)
6. Remove "Demo Mode" badges from Joy Coins / Community Pass UI

**This phase makes Community Pass revenue flow.**

### Phase E: Direct Purchases

**Goal:** Users can buy directly from businesses.

1. Create `on_behalf_of` checkout flow for direct purchases
2. Add "Buy Now" option alongside "Use Joy Coins" in event/product UI
3. Handle direct purchase completion webhook
4. Business receives payment minus Stripe processing fee

**This phase enables the dual-path (Joy Coins or direct) experience.**

### Phase F: Revenue Share Distribution

**Goal:** Businesses receive their share of the Community Pass pool.

1. Build `calculateRevenueShare` server function
2. Build admin revenue share dashboard
3. Build business-side Community Pass dashboard
4. Create Stripe Transfers to connected accounts
5. Handle transfer webhooks (created, failed)
6. Set up scheduled monthly distribution (or manual trigger)

**This phase closes the loop — money flows to businesses.**

---

## Legal Prerequisites (Before Money Flows)

Before enabling any real payment processing:

- [ ] Professional legal review confirming Community Pass is not money transmission (~$100)
- [ ] Revenue share agreement template for participating businesses
- [ ] 1099-NEC preparation (collect business TIN during Connect onboarding)
- [ ] Terms of Service update for Community Pass terms
- [ ] Privacy policy update for payment data handling

See LEGAL-RESEARCH.md action checklist for details.

---

## Stripe Fee Summary

| Fee | Who Pays | When |
|-----|----------|------|
| Processing (~2.9% + $0.30) | Deducted from each charge | At purchase |
| Connect fee ($2/month per active account) | LocalLane | Monthly |
| Transfer fee ($0.25 per transfer, if applicable) | LocalLane | At revenue share distribution |

---

*This spec is part of the LocalLane Spec-Repo. It supersedes the previous version (Separate Charges and Transfers model). The Community Pass concept and revenue model details are in the private repo. Reference before implementing any payment features.*
