# STRIPE-CONNECT.md — Stripe Connect Integration Spec

> Defines the Stripe Connect architecture for LocalLane payments.
> Created: 2026-02-01
> Status: Spec — Not yet implemented

---

## Overview

LocalLane uses **Stripe Connect** to collect payments from community members and distribute earnings to businesses. The platform acts as a marketplace: users buy Punch Passes on LocalLane, use punches at participating events, and businesses receive payouts minus a platform fee.

**Stripe Connect model:** Marketplace (Separate Charges and Transfers)
**Tier requirement:** Standard (Tier 2) and Partner (Tier 3) only
**Platform fee:** 15% of punch value at time of redemption

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

```javascript
// Only show payment setup for Tier 2+
const { tierLevel } = useOrganization(business);

if (tierLevel < 2) {
  // Show lock icon: "Upgrade to Standard to accept payments"
  return <LockedFeature tier="standard" feature="Accept Payments" />;
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

| Consideration | Notes |
|---------------|-------|
| **Same-owner fee exemption** | When Doron runs both the platform and a business, shouldn't pay Stripe fees to himself. Could use internal ledger credits or skip Stripe transfer for platform-owned businesses. Revisit when relevant. |
| **Instant payouts** | Stripe offers instant payouts (for a fee). Could be a Partner perk. |
| **Refund handling** | User wants refund on unused punches. Need refund policy and Stripe refund integration. |
| **Expired punch handling** | When punches expire (12 months), does LocalLane keep the money? Legal/policy decision needed. |
| **Multi-currency** | Not needed for Portland launch. Would need if expanding internationally. |
| **Tax reporting** | Stripe Connect handles 1099 forms for connected accounts earning over $600/year. |
| **Dispute handling** | If a user disputes a Punch Pass charge, LocalLane handles the dispute (we're merchant of record). |

---

*This document is the source of truth for Stripe Connect integration. Update as decisions are made during implementation.*
