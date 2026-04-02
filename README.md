# PropEx · Fractional Real Estate Exchange
**ISBR Business School · Entrepreneurship Project · April 2026**
Naman Bhandari · Aryan · Trishanu

---

> This document is a revised working model. It answers four structural questions left open in the original draft — initial valuation, how trading works, revenue model, go-to-market strategy — and rewrites the risk section with concrete resolutions.

---

## OVERVIEW

PropEx is a regulated platform where investors buy and sell fractional ownership stakes in individual Indian residential properties — not a pooled fund, not a basket, not a trust. A specific house or apartment, chosen by the investor, priced transparently, with a simple mechanism to buy and sell portions of it online.

The mechanism rests on three pillars: a legal architecture using Special Purpose Vehicles (SPVs) drawn from company law, a pricing model anchored to independent valuation, and a peer-to-peer transaction layer that lets owners sell their stake to willing buyers without requiring a stock exchange.

---

## THE PROBLEM

Indian residential real estate is simultaneously the country's most valued asset class and its most inaccessible one. A Bengaluru professional earning ₹10 lakh annually faces a median home price of ₹80–120 lakh — 8 to 12 times their gross income. The down payment alone exceeds three years of savings.

| Barrier | Structural Impact |
|---|---|
| Affordability | Median home prices are 8–12× average annual income in Tier 1 cities. A 20% down payment locks out the majority of salaried professionals indefinitely. |
| Illiquidity | Real estate is binary — you own 100% or nothing. No mechanism exists to sell 20% of your house to fund an emergency or an opportunity. |
| Exclusion | Without existing collateral, banks won't lend. Without a loan, no purchase. Without a purchase, no collateral. The cycle is self-reinforcing. |
| Opacity | Pricing is negotiated privately. No standardised yield data, no comparable transaction database, no real-time price signal. |

---

## HOW INITIAL LISTING VALUE IS DECIDED

This was not answered clearly in the original document. Here is the full framework.

### Step 1 — Independent Valuation (Primary Anchor)

Before a property is listed on PropEx, a **SEBI-registered independent property valuer** is appointed by the platform — not by the owner. The valuer produces a **Total Asset Value (TAV)** using three standard methods:

- **Sales Comparison Approach**: Recent comparable transactions within a 1km radius, adjusted for floor, age, condition, and amenities.
- **Income Approach**: Gross rental yield × standard capitalisation rate for that micro-market. For example, Koramangala yields ~3–3.5% gross; the income approach anchors TAV to rental earning capacity.
- **Cost Approach** (secondary validation only): Land value + depreciated replacement cost of the structure.

The final TAV is a weighted blend of the first two methods. The cost approach is used only as a sanity check.

### Step 2 — Unit Count and IPO Price

Once TAV is established:

```
Number of units = TAV ÷ Target unit price (₹500–₹1,000 range)

Example:
  TAV = ₹80,00,000
  Target unit price = ₹1,000
  Units issued = 8,000 units

  A buyer of 100 units owns 100/8,000 = 1.25% of the property.
```

The IPO (initial subscription) price is fixed at this derived value. It does not move during the subscription window. Every investor who subscribes during the IPO gets the same price.

### Step 3 — Post-Listing Price Behaviour

After full subscription, trading opens. From this point, the **price is not set by the platform**. It is determined by what sellers ask and what buyers are willing to pay. However, two guardrails prevent speculative disconnection:

- **Quarterly NAV Reset**: Every 90 days, the property is independently revalued. The NAV per unit is published. If the market price has diverged significantly from NAV, a correction band is applied — trades cannot occur more than 20% above or below the latest NAV.
- **Forced Reset on Major Events**: If a significant event occurs (structural damage, tenant dispute, area rezoning), an emergency revaluation is triggered.

This means the listing price is always grounded in reality, not in sentiment.

---

## HOW TRADING WORKS — A SIMPLER MODEL

### The Original Problem with the "Live Exchange" Idea

The original document described a live order book with real-time bid/ask prices, like Zerodha or NSE. This creates a real problem: **it mimics a stock exchange without having stock-exchange-level liquidity.** With 50 properties and even 50,000 users, most properties will have thin order books, wide spreads, and frustrated investors who can't exit. It also invites speculation divorced from property fundamentals.

### The Better Model: Peer-to-Peer Stake Marketplace (Not a Trading Platform)

PropEx should not try to be SEBI. Think of it less like NSE and more like a **classified board for property stakes** — but with legal enforcement, verified pricing, and instant settlement.

Here is how it works:

**Selling a stake:**
A unit holder who wants to exit lists their units for sale. They set an asking price (within the ±20% NAV band). The listing is visible to all registered PropEx users. It stays live until a buyer accepts or the seller withdraws it.

**Buying a stake:**
A registered investor browses available listings — searchable by locality, property type, yield, price per unit. They find a listing they want, review the House Profile Document, and submit a purchase request. If the seller accepts (or the listing was open-offer), the transaction executes.

**Settlement:**
The share transfer is recorded in the SPV's digital shareholder register (maintained by PropEx as Registrar). The buyer's bank account is debited, the seller's is credited. PropEx charges a transaction fee. No land registry involvement. Settlement: T+1 business day.

**No order book. No market orders. No live price ticker.**

This is intentionally simpler. It matches how property actually works — people think carefully before buying or selling, they don't day-trade houses. The platform makes this process fast, legal, and transparent, but it does not pretend residential property is a stock.

### What Stays the Same

- IPO subscription process (initial sale of units when a property first lists)
- NAV-based pricing anchor
- Monthly rental income distribution
- Concentration limits per investor
- Minimum ₹500 entry

### What Changes

- No live order book or real-time price feed
- No market-making reserve (not needed without live trading)
- Listings are posted with asking prices and stay up until filled
- Price negotiation happens within the NAV band
- Trading hours are irrelevant — listings are always visible (transactions settle on business days)

---

## REVENUE MODEL — HOW PROPEX MAKES MONEY

### How SEBI Makes Money (Internal Reference)

SEBI's revenue comes from: annual fees from registered intermediaries (brokers, depositories, mutual funds), transaction charges on trades processed through SEBI-regulated exchanges, fines and penalties for regulatory violations, and application/registration fees from new entities seeking licences. SEBI does not charge investors directly. It charges the infrastructure players and intermediaries who serve investors.

### Applying This Logic to PropEx

PropEx sits between SEBI (regulator) and SEBI (exchange). It is a licensed intermediary and marketplace operator. Its revenue model charges the entities using the platform — property owners, and transacting investors — not a subscription fee that would discourage participation.

| Revenue Stream | How It Works | Estimated Rate |
|---|---|---|
| **Listing Fee** | One-time fee charged to the property owner when a property is onboarded. Covers due diligence cost (valuer, title search, inspection), SPV formation, and listing. | ₹75,000–₹1,50,000 per property depending on asset value |
| **Transaction Fee (Buyer)** | Charged on every secondary market stake purchase. Deducted at settlement. | 1.5–2% of transaction value |
| **Transaction Fee (Seller)** | Charged on every secondary market sale. Deducted from proceeds. | 1–1.5% of transaction value |
| **Rental Management Fee** | PropEx (or its certified partner) manages tenant placement, rent collection, maintenance coordination. Fee deducted before distributing rental income to unit holders. | 8–10% of monthly gross rent per property |
| **Annual Registrar Fee** | PropEx maintains the SPV's shareholder register as Registrar. Annual fee per SPV for maintaining and updating the register, handling KYC re-verification, and issuing dividend distribution records. | ₹12,000–₹25,000 per SPV per year |
| **Revaluation Fee** | Quarterly independent valuations are mandatory. PropEx coordinates and charges a pass-through fee (cost of valuer + platform margin). | ₹8,000–₹15,000 per property per quarter |

### Revenue at Scale — Illustrative Numbers

```
Phase 1: 50 properties, assume 4 stake transactions per property per month

  Listing fees:        50 × ₹1,00,000          = ₹50,00,000 (one-time)
  Transaction fees:    50 × 4 × ₹50,000 × 3%   = ₹30,00,000 / month
  Rental mgmt fees:    50 × ₹80,000 rent × 9%  = ₹3,60,000 / month
  Registrar fees:      50 × ₹18,000 / year      = ₹75,000 / month
  Revaluation:         50 × ₹10,000 / quarter   = ₹1,67,000 / month

  Total monthly (Phase 1, steady state): ~₹35–38 lakh/month
```

This is conservative. Transaction fees scale with the value of stakes traded, not just volume.

---

## WHO WE ARE ADVERTISING TO, AND HOW WE REACH THEM

### Primary Audience: The Stuck Aspirant

**Profile:** Salaried professionals aged 26–38 in Bengaluru (Phase 1). Household income ₹8–25 lakh per year. Already investing in mutual funds or stocks via Zerodha/Groww. Knows real estate is a good asset class but has accepted they cannot afford it yet. Renting. Moderately financially literate. Uses Instagram, YouTube, and Reddit.

**Why they care:** They want real estate exposure without a ₹20 lakh down payment. They already understand SIPs and fractional investing conceptually. The message that lands: *"You've been investing in Infosys. Now invest in the flat next door."*

### Secondary Audience: The Small Property Owner

**Profile:** A homeowner or landlord who owns a second property and wants to unlock partial liquidity — for a business, a child's education, a personal need — without selling the whole asset. Currently has no mechanism to do this. Banks won't lend against second property easily without income proof.

**Why they care:** PropEx lets them sell 30% of their second flat to investors, keep 70%, still collect their share of rent, and get the cash they need — without losing the asset entirely.

### Tertiary Audience: The NRI Investor

**Profile:** Indian citizens living in the US, UK, UAE, Singapore. Wants Indian real estate exposure but cannot manage a property remotely. Income is in foreign currency, so ₹500/unit is trivially small. Trusts regulated platforms. Familiar with fractional investing concepts from Western fintech.

**Why they care:** PropEx gives them managed, transparent residential exposure in Indian cities they know — without the operational nightmare of owning property remotely.

---

### How We Reach Them

**Channel 1 — Content on YouTube and Instagram (Primary)**

Not ads. Actual content. A series of short videos (3–7 minutes) answering questions real people have: *"Can I actually own a flat in Koramangala for ₹500?"*, *"What happens to my money if the tenant leaves?",* *"How is this different from a REIT?"*

The goal is to build trust before the platform launches. A waitlist of 5,000 people who understand exactly what PropEx is — and have chosen to sign up — is worth more than 50,000 casual impressions.

**Channel 2 — Personal Finance Communities**

Reddit (r/IndiaInvestments, r/personalfinanceindia), Finshots, Zerodha's Varsity community, WhatsApp investor groups. These are high-intent audiences. A well-written post that genuinely answers "how does fractional real estate work in India" will circulate organically.

**Channel 3 — Partnership with Fintech Platforms**

Integration or co-marketing with platforms where the target audience already invests — Zerodha, Groww, Smallcase. Even a simple "PropEx is now available on Smallcase" announcement reaches millions of people already comfortable with fractional, digital investing.

**Channel 4 — NRI Outreach via Partner Brokers Abroad**

In Phase 2, partnerships with Indian diaspora investment advisors in Dubai, Singapore, and the US. NRIs actively seek managed Indian real estate exposure. A regulated, SEBI-compliant product addresses a real gap.

**Channel 5 — Developer and Builder Relationships (Supply Side)**

Property developers who list on PropEx benefit from faster capital and guaranteed subscription before construction is complete. Existing developer LOIs are a proof point — lean into this publicly. A developer endorsement signals legitimacy to both investors and regulators.

---

## RISKS — RESOLVED, NOT JUST LISTED

The original document listed risks with generic mitigations. Below is a rewrite that either provides a concrete solution or, where no solution exists, specifies how to avoid the scenario entirely.

---

### Risk 1: Regulatory Shift (SEBI SM REIT Framework Changes)

**What could go wrong:** SEBI introduced the SM REIT framework in March 2024. It is new. It could be amended — minimum thresholds raised, SPV structures restricted, or residential assets excluded from coverage.

**Resolution:** Apply for SEBI's regulatory sandbox before full launch. This gives the platform a formal engagement channel with the regulator and advance notice of any amendments. More importantly, design the SPV architecture so it is valid under the Companies Act 2013 **independently** of the SM REIT framework. If SM REIT rules change, the underlying SPV structure does not collapse — it may require a different disclosure regime, but investor holdings remain legally valid. Maintain a dedicated regulatory counsel on retainer (not ad hoc). Build compliance as an internal team, not an outsourced function.

**If no solution:** If SEBI were to prohibit residential fractional platforms entirely (an extreme and currently unlikely scenario), existing investors retain their SPV shares — their legal ownership does not vanish. The platform would need to facilitate a structured wind-down (property sale, proceeds distribution), which is already the exit mechanism for any SPV. Investors are protected even in this scenario.

---

### Risk 2: Secondary Market Illiquidity

**What could go wrong:** If very few people want to buy a specific property's units at any given time, a seller is stuck. This was the original document's highest-rated risk.

**Resolution (updated for peer-to-peer model):** The live order book model has been replaced with a peer-to-peer listing model, which is more appropriate for residential real estate. The illiquidity risk is now managed differently:

- **Listing requires full IPO subscription.** A property does not begin trading until 100% of its units are subscribed. This ensures there is an existing investor base from day one — not an empty market.
- **Structured exit at 12 months.** If a unit holder cannot find a buyer within 12 months of listing their stake, PropEx triggers a buyback at the latest NAV, funded by a Liquidity Reserve (initially ₹50 lakh). This is the hard floor.
- **Concentration in Phase 1.** 50 properties in high-demand Bengaluru micro-markets with 50,000 registered users means approximately 1,000 investors aware of each property. Far better odds of finding a counterparty than a fragmented 500-property launch.
- **The platform can itself list units for sale** from its market-making position if needed, and accept buy requests — acting as a passive liquidity provider, not an active trader.

**If no solution:** Properties in genuinely zero-demand locations should never have been listed. This is why the eligibility criteria include a rental viability assessment and a liquidity score pre-listing. Prevention is the solution — not recovery.

---

### Risk 3: Property Title Fraud

**What could go wrong:** Indian property titles are complex. Undisclosed encumbrances, forged documents, inheritance disputes, or benami ownership could invalidate a listing. One fraudulent property would destroy platform credibility.

**Resolution:**
- Multi-layer due diligence: independent title search (30-year chain), encumbrance certificate from the sub-registrar, NOC checks, mutation record verification, and RERA status check.
- **Title insurance** is purchased per property at listing. If a title defect emerges post-listing, the insurer compensates investors — PropEx does not absorb this risk alone, and neither does the investor.
- Platform absorbs due diligence cost in the listing fee — the owner does not control which agencies are used. The valuer and title search firm are appointed by PropEx, not by the listing party.
- Independent trustee per SPV has a legal obligation to flag title anomalies. They are a second check on platform-level due diligence.

**If no solution:** If a fraud scenario occurs despite all of the above, the title insurance policy is the last line of defence. This is why it is non-negotiable, not optional.

---

### Risk 4: Property Valuation Disputes

**What could go wrong:** Investors believe their property is undervalued or overvalued. Market prices diverge from NAV. Investor trust erodes.

**Resolution:**
- Quarterly independent revaluation by a SEBI-registered valuer (appointed by PropEx, not by the property owner). Methodology and comparable data published alongside the NAV figure.
- The ±20% trading band (updated model) prevents market prices from drifting wildly from NAV. If a seller believes their property is worth more, they can ask up to 20% above NAV — the market will tell them whether buyers agree.
- Any investor who disputes a valuation may formally request a review. A second independent valuer is appointed; if the two figures differ by more than 10%, a third valuer adjudicates. This is standard practice in institutional real estate.
- All valuation reports are published on the platform. Transparency is the dispute prevention mechanism.

---

### Risk 5: Rental Management Quality

**What could go wrong:** The platform's rental management partner fails to find tenants, collects rent irregularly, or mismanages maintenance. Investor returns collapse.

**Resolution:**
- In Phase 1, PropEx manages all properties in-house with its own property management team. No outsourcing until the operational model is proven.
- SLA-backed contracts: if a property sits vacant for more than 45 days after the previous tenant exits, PropEx waives its rental management fee for that period and contributes from the SPV's maintenance reserve to ensure the property is re-advertised actively.
- Monthly performance dashboards published to all unit holders: occupancy status, rent collected, maintenance spend, reserve balance.
- A maintenance reserve is built into each SPV at listing (typically 3–5% of TAV, funded from a portion of rental income over the first 12 months). This covers unexpected repairs without requiring special assessments from investors.

**If no solution:** If a property is genuinely unrentable for an extended period (area demand collapsed, structural issues), the trustee is empowered to recommend a property sale to unit holders. A vote is held; majority decides. Proceeds are distributed proportionally. Exit is always available.

---

### Risk 6: Two-Sided Cold Start

**What could go wrong:** Without investors, developers won't list. Without listings, investors won't sign up. The platform never achieves critical mass.

**Resolution:**
- **Supply side is locked in first.** Developer LOIs are signed before the platform opens to investors. Phase 1 launches only when 50 qualified properties are ready to list. Investors arrive to a full shelf, not an empty one.
- **Demand side is built via waitlist.** 5,000+ waitlist members are cultivated through content and community before launch. These are not casual impressions — these are people who have read about PropEx, understood it, and chosen to sign up. The IPO subscription for Phase 1 properties is offered to waitlist members first (priority access as a retention mechanism).
- **No chicken-and-egg at property level.** The 100% subscription requirement before trading begins means every property enters the market with a complete investor base. There is never a "ghost" listing.

---

### Risk 7 (New): Platform Itself as a Point of Failure

**What could go wrong:** PropEx as a company could face financial difficulty, regulatory action, or operational collapse. What happens to investors' holdings?

**Resolution (this was missing from the original document):**
- Investors do not own units in PropEx. They own shares in individual SPVs incorporated under the Companies Act. These SPVs are legally independent of the platform.
- If PropEx ceases operations, the SPVs continue to exist. The independent trustee per SPV would assume management responsibility and arrange for an alternative registrar.
- This structural separation — investors' assets held in independent legal entities, not on PropEx's balance sheet — is the most important investor protection in the entire architecture. It should be communicated clearly to every investor at onboarding.

---

## SUMMARY

**Three things define this platform — unchanged from the original:**

**Legal Clarity.** The SPV + Companies Act structure is legally sound. SEBI's SM REIT framework provides an explicit regulatory home. This is not a grey area.

**Simplicity Over Speculation.** The peer-to-peer stake marketplace model — not a live order book — matches how residential property actually works. It is simpler, more honest, and more durable. PropEx does not try to make real estate behave like a stock. It makes buying and selling portions of a house easier and online.

**Selective Listing Over Rapid Scaling.** 50 properties in Bengaluru, fully subscribed, with genuine liquidity — is worth more than 500 thin markets across 10 cities. Quality, legal clarity, and depth per property matter more than breadth.

---

*If PropEx executes on legal integrity, genuine liquidity, and disciplined listing — it becomes the infrastructure layer for how Indian residential real estate is owned, transacted, and valued.*

*Not the next fractional ownership platform. The exchange.*

---

**Contact**
Naman Bhandari: 9900515888 · Aryan: 8861170330 · Trishanu: 6305007928
ISBR Business School Entrepreneurship Project — 2026
