# STRIPE-CONNECT.md — Stripe Connect Integration Spec

> Defines the Stripe Connect architecture for LocalLane payments.
> Created: 2026-02-01
> Status: Spec — Not yet implemented

---

## Overview

LocalLane uses **Stripe Connect** to collect payments from community members and distribute earnings to businesses. The platform acts as a marketplace: users buy Punch Passes on LocalLane, use punches at participating events, and businesses receive payouts minus a platform fee.

**Stripe Connect model:** Marketplace (Separate Charges and Transfers)
**Tier requirement:** Standard (Tier 2), Partner (Tier 3), or Tier 1 with Network Add-On
**Platform fee:** 15% of punch value at time of redemption

### Stripe is Platform-Wide Infrastructure

While Punch Pass transactions are the primary revenue flow at launch, Stripe serves as the payment backbone for all LocalLane revenue streams:

| Revenue Stream | Stripe Feature | Flow Direction | Covered Here? |
|---------------|---------------|----------------|---------------|
| **Punch Pass marketplace** | Connect (Separate Charges + Transfers) | User → LocalLane → Business | ✅ Primary focus |
| **Tier subscriptions** | Billing (Subscriptions) | Business → LocalLane | Phase E (basics) |
| **Network Add-On** | Billing (Subscriptions) | Business → LocalLane | Phase E (basics) |
| **Future services** | TBD | TBD | ❌ Not scoped |

**Important distinction:** Tier subscriptions and the Network Add-On are straightforward Stripe Billing — the business pays LocalLane monthly, no connected accounts or transfers involved. The Punch Pass marketplace is the complex part — that's what requires Stripe Connect with connected accounts, settlement logic, and multi-party money movement. Phase E of this spec covers subscription basics, but a separate BILLING.md may be warranted as pricing is finalized.

Any future revenue opportunities — premium tools, featured listings, API access, or services we haven't imagined yet — would all route through this same Stripe infrastructure. The architecture is designed to grow with the platform.

---

## Why Separate Charges and Transfers

Stripe Connect offers three charge models. We use **Separate Charges and Transfers** because:

| Model | How It Works | Why Not / Why Yes |
|-------|-------------|-------------------|
| **Destination Charges** | Payment goes directly to one connected account | ❌ Punch Passes aren't tied to one business at purchase time |
| **Direct Charges** | Connected account is the merchant of record | ❌ We want LocalLane on the receipt, not individual businesses |
| **Separate Charges + Transfers** | Platform collects payment, transfers later | ✅ User buys punches now, uses at different businesses later |

The key reason: when a user buys a 20-punch pack, we don't know which businesses will receive those punches. The money sits in LocalLane's account and gets transferred out as punches are redeemed — potentially to 5 different businesses over 12 months.

---

## Account Architecture

```
LocalLane Platform Account (Stripe)
│
│   This is the main Stripe account.
│   All Punch Pass purchases land here.
│   Tier subscription payments land here.
│   Platform fees stay here.
│
├── Connected Account: Yoga Studio (standard tier)
│   └── Receives transfers when their events use punches
│
├── Connected Account: Art School (standard tier)
│   └── Receives transfers when their events use punches
│
├── Connected Account: Recess (partner tier)
│   └── Receives transfers when Recess events use punches
│
└── Connected Account: Creative Alliance (partner tier)
    └── Receives transfers when TCA events use punches
```

**One platform account.** Every Tier 2+ business gets a connected account underneath it. Tier 1 businesses do not have connected accounts — they can't accept Punch Passes or receive payouts.

---

## Money Flow

### Step 1: User Buys Punch Pass

```
User → Stripe Checkout → LocalLane Platform Account

Example: User buys 20-pack for $180
- $180 charge on LocalLane's Stripe account
- Stripe processing fee (~$5.52 for 2.9% + $0.30) deducted
- ~$174.48 lands in LocalLane's available balance
- PunchPass entity created with current_balance: 20
```

**What the user sees:** "LocalLane" on their credit card statement.
**What happens in our database:** PunchPass record created, balance = 20, expiry = 12 months.

### Step 2: User Redeems Punches at an Event

```
User → Check-in at event → Punches deducted → Transfer queued

Example: User uses 2 punches at a yoga class
- Per-punch value: $180 / 20 = $9.00 per punch
- Gross value of 2 punches: $18.00
- Platform fee (15%): $2.70
- Business receives: $15.30
```

**What happens immediately:** PunchPass balance decremented, PunchPassUsage record created.
**What happens later:** Settlement process creates a Stripe Transfer to the business's connected account.

### Step 3: Settlement (Transfer to Business)

```
LocalLane Platform Account → Stripe Transfer → Business Connected Account

Settlement runs on a schedule (daily or weekly, configurable).
Groups all punch redemptions per business since last settlement.
Creates one transfer per business per settlement period.
```

**Example weekly settlement for Yoga Studio:**
- Monday: 3 punches redeemed ($27.00 gross)
- Wednesday: 2 punches redeemed ($18.00 gross)
- Friday: 1 punch redeemed ($9.00 gross)
- **Weekly total:** 6 punches, $54.00 gross
- **Platform fee (15%):** $8.10
- **Transfer to Yoga Studio:** $45.90

---

## Punch Value Calculation

Punch packs have different per-punch values based on the pack size:

| Pack | Price | Punches | Per-Punch Value |
|------|-------|---------|----------------|
| 10-pack | $100 | 10 | $10.00 |
| 20-pack | $180 | 20 | $9.00 |
| 30-pack | $255 | 30 | $8.50 |

When a user redeems punches, we need to know the per-punch value of the specific pack they're drawing from (FIFO — oldest pack first). This is already tracked in the PunchPass entity.

**Per-punch value** is calculated and stored at purchase time:

```javascript
per_punch_value = purchase_price / total_punches
```

This value is recorded on each PunchPassUsage record so settlement can calculate the correct transfer amount.

---

## Business Transparency (Punch Pass Economics)

Every business accepting Punch Passes must clearly understand the economics before opting in, and must have ongoing visibility into how their earnings are calculated. This transparency is essential to trust — if a business feels surprised or shortchanged at settlement time, that erodes the exact community trust LocalLane is built on.

### What Businesses Need to Understand

**1. Not all punches are worth the same dollar amount.**

Users buy packs at different price points, and bulk buyers get a discount:

| Pack Purchased | Per-Punch Value | 2 Punches Redeemed = |
|---------------|----------------|---------------------|
| 10-pack ($100) | $10.00 | $20.00 gross |
| 20-pack ($180) | $9.00 | $18.00 gross |
| 30-pack ($255) | $8.50 | $17.00 gross |

A yoga studio accepting 2 punches for a class might earn $20 from one attendee and $17 from another, depending on which pack the user bought. The business doesn't choose — the system uses FIFO (oldest punches first) to determine which pack is drawn from.

**2. The 15% platform fee is deducted at settlement, not at redemption.**

When a punch is used at an event, the full gross value is tracked. At settlement time (weekly), the platform fee is calculated on the total and the net amount is transferred.

**3. FIFO means the system uses the user's oldest punches first.**

If a user has punches from a 10-pack (worth $10 each) and a 30-pack (worth $8.50 each), the 10-pack punches are used first because they were purchased earlier. This is better for businesses on average, since older packs tend to be smaller (higher per-punch value).

**4. Volume discounts benefit the whole ecosystem.**

The discount on larger packs drives more users to buy bigger packs, which means more total punches in circulation, which means more foot traffic for all participating businesses. The per-punch value difference ($10 vs $8.50) is the cost of that increased volume.

### Where This Information Must Appear

| Location | What to Show | When |
|----------|-------------|------|
| **Punch Pass onboarding** | Full economics explanation before business opts in | When business enables Punch Pass acceptance |
| **Business dashboard** | Real-time earnings with per-redemption breakdown | Ongoing (FinancialWidget / EarningsDashboard) |
| **Settlement notification** | Period summary with line items | At each settlement |
| **Marketing / sales deck** | Clear economics pitch for recruiting businesses | Pre-signup |
| **Help center / FAQ** | Detailed explanation with examples | Always available |

### Dashboard Earnings Detail (Business View)

The EarningsDashboard should show granular redemption data so businesses always understand their numbers:

```
RECENT PUNCH ACTIVITY
─────────────────────────────────────────────────────────
Date        Event              Punches   Value    Pack Source
Jan 28      Tue Yoga 10am      2         $18.00   20-pack ($9/punch)
Jan 29      Thu Yoga 10am      1         $10.00   10-pack ($10/punch)
Jan 30      Sat Family Yoga    3         $25.50   30-pack ($8.50/punch)
─────────────────────────────────────────────────────────
Week Total                     6         $53.50

Platform fee (15%):                      -$8.03
Net payout:                              $45.47
Settlement: Feb 3 (Monday)
```

The key columns are **Value** and **Pack Source** — these make it immediately obvious why two identical redemptions can produce different dollar amounts.

### Onboarding Consent Screen

Before a business can enable Punch Pass acceptance, they must see and acknowledge:

```
ACCEPTING PUNCH PASSES — HOW IT WORKS
──────────────────────────────────────

✅ Users buy Punch Packs on LocalLane (10, 20, or 30 packs)
✅ Users redeem punches at your events
✅ You receive weekly payouts to your bank account

WHAT YOU EARN PER PUNCH:
• Punches from 10-packs: $10.00 each
• Punches from 20-packs: $9.00 each
• Punches from 30-packs: $8.50 each

The system uses the user's oldest punches first (FIFO),
so you won't know the exact value until redemption.

LocalLane retains a 15% platform fee from each settlement.

Example: 10 punches redeemed this week at mixed values = $92.50 gross
Platform fee (15%): -$13.88
Your payout: $78.62

[ ] I understand how Punch Pass earnings are calculated

                                    [Enable Punch Pass]
```

### Marketing Deck Talking Points

For recruiting new businesses, the pitch should address punch economics head-on:

**"What will I earn per punch?"**
Between $8.50 and $10.00 per punch, depending on the pack the user purchased. Our bulk discounts encourage users to buy more punches, which means more visits to your business.

**"Why do different punches have different values?"**
Volume pricing. A user buying 30 punches commits to visiting local businesses 30 times — that's tremendous value for the community. The small per-punch discount (15% off the base rate) is what drives that commitment.

**"What's the platform fee?"**
15% of the gross punch value. This covers payment processing, platform operations, and community programming including youth scholarships.

**"When do I get paid?"**
Weekly settlements every Monday. You'll see every redemption itemized in your dashboard with the exact value before payout.

**"What if a punch is from a discounted pack?"**
You still earn 85% of that punch's value. Even at the lowest rate (30-pack), you earn $7.23 per punch after the platform fee. Compare that to credit card processing fees on a $10 ticket — you're keeping more.

---

## Business Onboarding (Connect Onboarding)

When a business upgrades to Standard tier (or is created at Standard+), they go through Stripe Connect onboarding to link their bank account.

### Onboarding Flow

1. Business owner clicks "Set Up Payments" in their dashboard
2. LocalLane creates a connected account via Stripe API
3. Stripe generates an Account Link (hosted onboarding URL)
4. Business owner is redirected to Stripe's hosted onboarding
5. They enter: legal business name, EIN or SSN, bank account, identity verification
6. Stripe redirects back to LocalLane dashboard
7. LocalLane stores the connected account ID on the Business entity
8. Business can now accept Punch Passes and receive payouts

### Business Entity Changes

```javascript
// New fields on Business entity
{
  stripe_account_id: string | null,     // 'acct_xxx' — Stripe connected account ID
  stripe_onboarding_complete: boolean,  // Has business finished Stripe onboarding?
  stripe_payouts_enabled: boolean,      // Can business receive payouts?
  stripe_charges_enabled: boolean,      // Is account fully verified?
}
```

### Onboarding States

| State | What It Means | Dashboard Shows |
|-------|--------------|-----------------|
| No connected account | Business hasn't started onboarding | "Set Up Payments" button |
| Account created, onboarding incomplete | Started but didn't finish | "Complete Payment Setup" button |
| Onboarding complete, payouts disabled | Stripe reviewing identity | "Verification Pending" badge |
| Fully active | Can receive payouts | Green "Payments Active" badge |

### Tier Gating

Punch Pass acceptance requires Tier 2+, OR Tier 1 with the Network Add-On ($18/month). This means a basic business that just wants to accept punch passes can do so without upgrading to the full Standard feature set.

| Tier | Punch Pass Access | How |
|------|------------------|-----|
| Basic (Tier 1) | ❌ by default | Upgrade to Standard or buy Network Add-On |
| Basic + Network Add-On | ✅ Punch Pass only | $18/month — accepts punches, no other Tier 2 features |
| Standard (Tier 2) | ✅ included | Network Add-On bundled with Standard subscription |
| Partner (Tier 3) | ✅ included | Network Add-On bundled with Partner subscription |

```javascript
// Updated tier check for Punch Pass eligibility
const { tierLevel, canUsePunchPass } = useOrganization(business);
// canUsePunchPass is true if:
//   tierLevel >= 2  OR
//   tierLevel === 1 AND business.has_network_addon === true

if (!canUsePunchPass) {
  // Show two options:
  // 1. "Upgrade to Standard" (full feature unlock)
  // 2. "Add Punch Pass Network" ($18/month add-on)
  return <PunchPassUpgradeOptions />;
}

if (!business.stripe_account_id) {
  return <SetUpPaymentsButton />;
}

if (!business.stripe_onboarding_complete) {
  return <CompleteOnboardingButton />;
}

// Payments active — show earnings dashboard
return <EarningsDashboard />;
```

---

## Tier Subscription Payments

In addition to Punch Pass transactions, Stripe handles tier subscription billing.

### Subscription Flow

1. Business owner clicks "Upgrade to Standard"
2. LocalLane creates a Stripe Checkout Session for the subscription
3. User enters payment info on Stripe-hosted checkout
4. On success, webhook fires → LocalLane updates `subscription_tier` to `'standard'`
5. Features unlock immediately

### Stripe Products Needed

| Product | Price | Interval | Notes |
|---------|-------|----------|-------|
| LocalLane Standard | $X/month | Monthly | Tier 2 subscription |
| LocalLane Partner | $Y/month | Monthly | Tier 3 subscription (earned, not self-serve) |
| Network Add-On | $18/month | Monthly | Punch Pass network access (included in Partner) |

**Note:** Exact pricing TBD. These products will be created in Stripe when ready.

### Subscription Lifecycle

| Event | Stripe Webhook | LocalLane Action |
|-------|---------------|-----------------|
| Subscription created | `checkout.session.completed` | Set `subscription_tier` to `'standard'` |
| Payment succeeds | `invoice.paid` | No action (subscription continues) |
| Payment fails | `invoice.payment_failed` | Email business owner, grace period |
| Subscription canceled | `customer.subscription.deleted` | Downgrade to `'basic'`, disable Punch Pass |
| Trial ends | `customer.subscription.trial_will_end` | Email reminder |

---

## Settlement System

Settlement is the process of calculating how much each business earned from Punch Pass redemptions and creating Stripe Transfers.

### Settlement Logic

```
For each business with unsettled PunchPassUsage records:
  1. Query all PunchPassUsage since last settlement
  2. For each usage record:
     - Look up per_punch_value from the PunchPass record
     - Calculate: gross_value = punches_deducted × per_punch_value
  3. Sum gross_value for all usage records
  4. Calculate platform_fee = total_gross × 0.15
  5. Calculate transfer_amount = total_gross - platform_fee
  6. Create Stripe Transfer to business's connected account
  7. Record settlement in PunchPassTransaction entity
  8. Mark PunchPassUsage records as settled
```

### Settlement Schedule Options

| Schedule | When | Best For |
|----------|------|----------|
| **Daily** | Every night at midnight | High-volume businesses |
| **Weekly** | Every Monday | Default for most businesses |
| **Manual** | Admin triggers | Testing, special cases |

Start with **weekly** for simplicity. Can add daily as an option for high-volume Partner businesses later.

### Settlement Data Model

```javascript
// PunchPassUsage (already exists — add fields)
{
  // Existing fields...
  per_punch_value: number,        // Dollar value per punch at time of redemption
  gross_value: number,            // punches_deducted × per_punch_value
  settled: boolean,               // Has this usage been included in a settlement?
  settlement_id: string | null,   // Reference to PunchPassTransaction
}

// PunchPassTransaction (already exists — clarify usage)
{
  id: UUID,
  business_id: UUID,
  type: 'settlement',             // Type of transaction
  period_start: ISO datetime,     // Settlement period start
  period_end: ISO datetime,       // Settlement period end
  total_punches: number,          // Total punches in this settlement
  gross_amount: number,           // Total gross value (cents)
  platform_fee: number,           // Platform fee amount (cents)
  net_amount: number,             // Amount transferred to business (cents)
  stripe_transfer_id: string,     // 'tr_xxx' — Stripe Transfer ID
  status: 'pending' | 'completed' | 'failed',
  created_date: ISO datetime,
}
```

---

## Webhook Handling

Stripe sends webhooks for payment events. LocalLane needs a server-side endpoint to process these.

### Required Webhooks

| Webhook Event | When It Fires | LocalLane Action |
|--------------|--------------|-----------------|
| `checkout.session.completed` | Punch Pass purchased OR tier subscription started | Create/update PunchPass, update subscription_tier |
| `invoice.paid` | Recurring subscription payment succeeds | Log payment, keep tier active |
| `invoice.payment_failed` | Subscription payment fails | Email owner, start grace period |
| `customer.subscription.deleted` | Subscription canceled | Downgrade tier to 'basic' |
| `account.updated` | Connected account status changes | Update stripe_payouts_enabled |
| `transfer.created` | Settlement transfer initiated | Update PunchPassTransaction status |
| `transfer.failed` | Settlement transfer failed | Alert admin, retry logic |

### Webhook Implementation

This requires a **Base44 server function** (similar to the existing `validatePunchPass` function):

```javascript
// functions/stripeWebhook.ts (Base44 server function)
// Receives Stripe webhook events
// Verifies webhook signature
// Routes to appropriate handler based on event type
```

---

## Platform Fee Structure

| Fee Type | Amount | Who Pays | When |
|----------|--------|----------|------|
| **Platform fee** | 15% of punch value | Business (deducted from transfer) | At settlement |
| **Stripe processing fee** | ~2.9% + $0.30 | LocalLane (from platform account) | At Punch Pass purchase |
| **Stripe Connect fee** | $2/month per active connected account | LocalLane | Monthly |
| **Stripe Transfer fee** | $0.25 per transfer (if applicable) | LocalLane | At settlement |

**Revenue math example (monthly):**

```
100 users buy 20-packs: 100 × $180 = $18,000 in Punch Pass sales
Stripe processing fee: ~$552 (2.9% + $0.30 per transaction)

2,000 punches redeemed across 15 businesses
Gross value: 2,000 × $9.00 = $18,000
Platform fee (15%): $2,700
Transfers to businesses: $15,300

Platform revenue: $2,700 - $552 (Stripe fees) - $30 (Connect fees) = ~$2,118/month
```

---

## UI Changes

### Punch Pass Purchase Page (User-Facing)

Replace the current demo mode with real Stripe Checkout:

```
Current (demo):  "Purchase" button → toast("Coming soon!")
New (real):      "Purchase" button → Stripe Checkout Session → Payment → PunchPass created
```

The user selects a pack (10, 20, or 30), clicks "Buy Now," and is redirected to Stripe's hosted checkout page. After payment, they return to LocalLane with their punch balance updated.

### Business Dashboard — FinancialWidget (Business-Facing)

Replace the placeholder FinancialWidget with real earnings data:

| Section | Content |
|---------|---------|
| **Earnings Summary** | Total earned (all time), This month, Pending settlement |
| **Recent Activity** | List of recent punch redemptions with date, event, punches, value |
| **Settlement History** | Past settlements with date, amount, status |
| **Payment Setup** | Stripe onboarding status, bank account status |

### Business Dashboard — Payment Setup

New section in BusinessDashboardDetail or as a step in onboarding:

| State | UI |
|-------|-----|
| Tier 1 | Lock icon + "Upgrade to Standard to accept payments" |
| Tier 2+, no Stripe account | "Set Up Payments" button → Stripe Connect onboarding |
| Onboarding incomplete | "Complete Payment Setup" banner with resume link |
| Verification pending | Yellow "Verification Pending" badge |
| Active | Green "Payments Active" badge + bank account last 4 |

### Admin Panel — Settlement Dashboard

New admin section showing:

| View | Content |
|------|---------|
| **Settlement Overview** | Next settlement date, pending amount, business count |
| **Settlement History** | Past settlements with business, amount, status |
| **Manual Settlement** | Button to trigger settlement for a specific business |
| **Failed Transfers** | List of failed transfers with retry option |

---

## Existing Code Touchpoints

### What Already Exists

| Component/Entity | Status | Connect Integration Needed |
|-----------------|--------|---------------------------|
| `@stripe/react-stripe-js` | ✅ Installed | Use for Checkout redirect |
| `@stripe/stripe-js` | ✅ Installed | Use for `loadStripe()` |
| `PunchPass` entity | ✅ Exists (demo mode) | Add `stripe_checkout_session_id`, `per_punch_value` |
| `PunchPassUsage` entity | ✅ Exists | Add `per_punch_value`, `gross_value`, `settled` |
| `PunchPassTransaction` entity | ✅ Exists | Repurpose for settlement records |
| `PunchPass.jsx` page | ✅ Exists (demo mode) | Replace demo purchase with real checkout |
| `FinancialWidget.jsx` | ✅ Exists (placeholder) | Replace with real earnings dashboard |
| `useOrganization()` hook | ✅ Exists | Already has `canUsePunchPass` flag |
| `CheckIn.jsx` | ✅ Exists | No changes needed (punch deduction already works) |
| `validatePunchPass` function | ✅ Exists | No changes for MVP (settlement is separate) |

### What Needs to Be Created

| Component | Type | Purpose |
|-----------|------|---------|
| `functions/stripeWebhook.ts` | Base44 server function | Process Stripe webhook events |
| `functions/createCheckoutSession.ts` | Base44 server function | Create Stripe Checkout for Punch Pass purchase |
| `functions/createConnectAccount.ts` | Base44 server function | Create connected account for business |
| `functions/createAccountLink.ts` | Base44 server function | Generate Stripe onboarding link |
| `functions/runSettlement.ts` | Base44 server function | Calculate and execute settlement transfers |
| `PaymentSetup.jsx` | React component | Stripe Connect onboarding UI in dashboard |
| `EarningsDashboard.jsx` | React component | Real earnings display (replaces FinancialWidget) |
| `SettlementAdmin.jsx` | React component | Admin settlement management |

---

## Implementation Phases

### Phase A: Platform Setup

**Goal:** Get Stripe account configured for Connect.

1. Rename existing Stripe account to "LocalLane" (or create new one)
2. Enable Stripe Connect in dashboard
3. Complete platform profile (business name, icon, brand color)
4. Set up webhook endpoint URL
5. Create Stripe products and prices for tier subscriptions
6. Store Stripe API keys in Base44 environment variables

**No code changes.** All done in Stripe Dashboard.

### Phase B: Punch Pass Purchase (Real Payments)

**Goal:** Users can buy Punch Passes with real money.

1. Create `createCheckoutSession` server function
2. Create `stripeWebhook` server function (handle `checkout.session.completed`)
3. Update `PunchPass.jsx` to call checkout instead of demo toast
4. Add `per_punch_value` to PunchPass entity on creation
5. Add `stripe_checkout_session_id` to PunchPass entity
6. Remove "Demo Mode" badges

**This phase makes money flow IN to LocalLane.**

### Phase C: Business Connect Onboarding

**Goal:** Businesses can link their bank accounts.

1. Create `createConnectAccount` server function
2. Create `createAccountLink` server function
3. Add `stripe_account_id` and related fields to Business entity
4. Build `PaymentSetup.jsx` component
5. Add onboarding state display in dashboard
6. Handle `account.updated` webhook

**This phase connects businesses to Stripe.**

### Phase D: Settlement System

**Goal:** Businesses receive their earnings.

1. Add `per_punch_value`, `gross_value`, `settled` to PunchPassUsage entity
2. Update punch deduction logic to calculate and store gross_value
3. Create `runSettlement` server function
4. Build admin settlement dashboard
5. Build business earnings dashboard (replace FinancialWidget)
6. Handle `transfer.created` and `transfer.failed` webhooks
7. Set up scheduled settlement (cron or manual trigger)

**This phase makes money flow OUT to businesses.**

### Phase E: Tier Subscriptions

**Goal:** Businesses pay for Standard tier via Stripe.

1. Create Stripe products/prices for tier subscriptions
2. Build upgrade checkout flow
3. Handle subscription webhooks (created, paid, failed, canceled)
4. Wire subscription status to `subscription_tier` field
5. Build subscription management UI in dashboard

**This phase makes tier upgrades generate revenue.**

---

## Environment Variables

These need to be set in Base44:

| Variable | Purpose | Where to Find |
|----------|---------|---------------|
| `STRIPE_SECRET_KEY` | Server-side API calls | Stripe Dashboard → API Keys |
| `STRIPE_PUBLISHABLE_KEY` | Client-side Stripe.js | Stripe Dashboard → API Keys |
| `STRIPE_WEBHOOK_SECRET` | Webhook signature verification | Stripe Dashboard → Webhooks |
| `STRIPE_CONNECT_CLIENT_ID` | Connect OAuth (if using OAuth onboarding) | Connect Settings |

---

## Security Considerations

- **Never expose `STRIPE_SECRET_KEY` to the frontend.** All Stripe API calls with the secret key happen in Base44 server functions.
- **Always verify webhook signatures** using `STRIPE_WEBHOOK_SECRET`.
- **Store connected account IDs** on the Business entity, but never store bank account details directly.
- **Stripe handles PCI compliance** — we never touch credit card numbers. Stripe Checkout is hosted by Stripe.
- **Settlement amounts must match usage records.** The admin dashboard should show reconciliation data.

---

## Future Considerations (Not in Scope)

### Critical — Must Resolve Before Scale

| Consideration | Why It Matters | Questions to Answer |
|---------------|---------------|-------------------|
| **Refund handling** | Users will ask for refunds on unused punches. Need clear policy before any real money flows. | Full refund? Pro-rated? Time limit? What about partially used packs? Who eats the Stripe processing fee on refunds? |
| **Expired punch handling** | Punches expire after 12 months (FIFO). When they expire, is that revenue for LocalLane or an accounting liability? | Does LocalLane keep expired punch money? Do we offer extensions? Notify users before expiration? Legal implications of holding unredeemed funds (state unclaimed property laws)? |

### Important — Revisit When Relevant

| Consideration | Notes |
|---------------|-------|
| **Same-owner fee exemption** | When Doron runs both the platform and a business, shouldn't pay Stripe fees to himself. Could use internal ledger credits or skip Stripe transfer for platform-owned businesses. |
| **Instant payouts** | Stripe offers instant payouts (for a fee). Could be a Partner-tier perk. |
| **Dispute handling** | If a user disputes a Punch Pass charge with their bank, LocalLane handles the dispute (we're merchant of record). Need dispute response templates and process. |
| **Tax reporting** | Stripe Connect generates 1099 forms for connected accounts earning over $600/year. Businesses need to understand this at onboarding. |
| **Multi-currency** | Not needed for Portland launch. Would need if expanding internationally. |
| **Partial punch redemption** | Can a business charge 0.5 punches? Current system assumes whole numbers. |
| **Pack gifting / transfer** | Can a user gift their punch pack to another user? Creates accounting complexity. |

---

*This document is the source of truth for Stripe Connect integration. Update as decisions are made during implementation.*
