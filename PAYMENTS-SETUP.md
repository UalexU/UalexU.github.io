# Payments — what I need from you to go live

The checkout in `All Ways Social Website v4.dc.html` now offers eight methods.
Right now it runs in **demo mode**: `PAYMENTS.demoMode = true` in the page's
`<script data-dc-script>` block, so nothing hits the network and nothing is
charged. Everything below is what turns that into real money in your account.

---

## 1. Pick a primary processor — this is the one decision that matters

Almost everything else follows from it. My recommendation is **Stripe**,
because a single Stripe account covers Card, Apple Pay, Google Pay, Cash App
Pay, Klarna and ACH — seven of the eight methods — with one integration and one
payout schedule. PayPal/Venmo always require a second account regardless of who
you pick, because PayPal owns Venmo.

| | Stripe (recommended) | Square | PayPal Commerce |
|---|---|---|---|
| Card | 2.9% + 30¢ | 2.9% + 30¢ | 3.49% + 49¢ |
| Apple / Google Pay | included | included | included |
| Cash App Pay | 2.9% + 30¢ | included | — |
| Klarna / BNPL | 5.99% + 30¢ | Afterpay 6% + 30¢ | Pay Later, included |
| ACH bank transfer | **0.8%, capped at $5** | not offered | — |
| PayPal / Venmo | needs PayPal too | needs PayPal too | native |

That ACH row is worth reading twice. On the **$1,750** Brand Launch deposit,
card costs you about **$51**; ACH costs **$5**. Over a year of deposits that is
real money, which is why I put bank transfer in the list rather than hiding it.

---

## 2. What I need from you, in order

### A. Business & bank details (needed before anything else)

Every processor runs KYC before releasing funds. Have these ready:

- [ ] **Legal business name and structure** — sole proprietor, LLC, etc.
- [ ] **EIN**, or your SSN if you're a sole proprietor without an EIN
- [ ] **Business address** and phone
- [ ] **Business bank account** — routing + account number for payouts
- [ ] **Government photo ID** for you as the beneficial owner
- [ ] **The statement descriptor** — the ≤22 characters customers see on their
      card statement. Suggest `ALLWAYSSOCIAL`. Get this right or you'll eat
      chargebacks from people who don't recognise the charge.

*You enter these directly into the processor's dashboard. Don't send them to me
— I don't need them and shouldn't have them.*

### B. Keys and IDs (send me these — they're safe to share)

Once the account is approved, from the Stripe Dashboard → Developers → API keys:

- [ ] **Publishable key** — starts `pk_live_…`. Public by design; goes in the page.
- [ ] **Secret key** — starts `sk_live_…`. **Do not send this to me or paste it
      into chat.** Put it straight into your server's environment variables.

If you want PayPal and Venmo, from developer.paypal.com → Apps & Credentials:

- [ ] **PayPal client ID** (live) — public, safe to send
- [ ] **PayPal secret** — server-side only, same rule as above

For Apple Pay you'll also need:

- [ ] An **Apple Pay merchant ID** (e.g. `merchant.com.allwayssocial`)
- [ ] The **domain verification file** hosted at
      `/.well-known/apple-developer-merchantid-domain-association` — Apple
      won't show the button on an unverified domain

For Google Pay:

- [ ] A **merchant ID** from the Google Pay & Wallet Console

### C. Decisions I need from you

- [ ] **Which methods do you actually want?** Right now all eight are on. Trim
      the `PAYMENTS.enabled` list to hide any of them — it's a one-line change.
      My honest read: keep Card, Apple Pay, Google Pay and ACH for certain.
      PayPal/Venmo are worth it if your clients skew younger or creator-side.
      Klarna on a $750 deposit costs you ~$75 in fees — only worth it if it
      genuinely converts people who'd otherwise walk.
- [ ] **Deposit amounts** — the page currently assumes 50% for four services and
      a flat $250 booking fee for Events. Confirm or correct.
- [ ] **Refund and cancellation policy.** Needed for real, not optional: Klarna
      and card networks both require published terms, and it's your first line
      of defence in a chargeback dispute. What's your window — 48 hours? 7 days?
      Is the deposit non-refundable after work starts?
- [ ] **Sales tax.** Marketing services are taxable in some states. Worth asking
      your accountant before the first invoice, not after.
- [ ] **Where should the receipt and calendar invite come from?** The success
      screen promises both. Right now nothing sends them. Options: Stripe's
      built-in receipts (free, plain), or a tool like Resend/Postmark for
      branded ones. Also need the calendar the booking should land in.

### D. What still needs building (not part of this design change)

The page is the front half. A real charge needs a small backend — the amount
must be decided server-side from the service id, or someone can edit the price
in their browser before paying.

- [ ] Somewhere to host the endpoint at `PAYMENTS.createIntentUrl`
      (`/api/create-payment-intent`) — a Vercel or Netlify function is plenty
- [ ] A webhook handler for `payment_intent.succeeded` — this is what actually
      confirms a booking. Never trust the browser's word that a payment worked
- [ ] Somewhere to record bookings — a database, or start with a Google Sheet /
      Airtable and grow into one

---

## 3. What's already done in the page

- Eight methods in one editorial list, styled in the site's own palette —
  sage rules, letterspaced caps, Allura for the amount, accent pink marking the
  selected row. The only brand colour is the 30px dot, so checkout still reads
  as All Ways Social, not as a payment vendor's page.
- Bodies switch per method: card form, one-tap wallet button, Klarna's four-
  instalment schedule (computed live from the deposit), and ACH with instant
  bank connect plus a manual fallback.
- Amounts derive from the selected service, so changing a deposit in `SERVICES`
  updates the modal, the instalment plan and every button label at once.
- A "processing" state between submit and confirmation, so the handoff to a
  provider has somewhere to live.
- An accepted-methods strip under the Reserve form.

## 4. One note on the wallet buttons

Apple and Google both require their own rendered button with their own mark and
wording — you can't ship a custom-styled "Pay with Apple Pay". The sage buttons
in the page are correct as a *design*, but at integration time those two get
swapped for the provider-rendered button. Everything around them stays.
