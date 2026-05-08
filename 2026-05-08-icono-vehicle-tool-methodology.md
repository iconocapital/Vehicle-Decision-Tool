# Vehicle Acquisition Path Planner — Methodology Document

**Version:** 1.0
**Tool:** `2026-05-08-icono-vehicle-decision-tool.html`
**Data layer dated:** May 8, 2026
**Geography of primary calibration:** Monroe County, NY (Rochester metro)
**Audience:** Iconoclastic Capital Management compliance review · Client-facing planning use
**Author:** Iconoclastic Capital Management

---

## 1. Purpose and scope

This client-facing planning tool walks a household through a vehicle acquisition decision and produces two comparable, side-by-side acquisition paths drawn from six possible types: **Finance New, Finance CPO, Finance Used, Lease New, Cash Purchase, and Finance + Invest the Delta**. The tool is positioned as educational planning, not investment, tax, legal, or personalized financial advice.

The differentiator versus a generic "lease vs. buy" calculator is the **opportunity-cost overlay**: every path that involves committing cash or interest payments is paired against the future value of that capital deployed at the client's stated nominal expected return. This is what Icono actually does for clients — frame transportation as a capital-allocation question.

---

## 2. Data layer

The tool ships with an embedded JavaScript data object (`DATA`) that is the single source of truth for all market-derived figures. The complete source attribution lives in the underlying research file (`compass_artifact_wf-72487930-7ab5-4bbe-9ed4-9becfb7e4365_text_markdown.md`, May 8, 2026). The data layer covers:

- National auto loan APR averages by VantageScore tier (Experian Q4 2025, released March 5, 2026)
- Best-rate lender APRs verified live May 7–8, 2026 (PenFed, Navy Federal, ESL FCU Rochester)
- Active OEM captive promotional financing (Toyota TFS, Honda AHFC, Hyundai/Kia, GM Financial, Stellantis, Ford Credit, Subaru, Tesla, Volvo) — all expiring early June or July 2026
- May 2026 lease money factors, residual percentages, acquisition/disposition fees by brand
- OEM-funded EV lease cash (Honda Prologue ~$16,550, Hyundai Ioniq 5 ~$17,250, Kia EV9 ~$12,850, etc. — all OEM marketing dollars now that the federal §45W lease loophole has terminated)
- Manheim Used Vehicle Value Index (April 2026: 211.9, MoM −1.6%, YoY +1.8%)
- Average used vehicle ATPs by age (1-yr, 3-yr, 5-yr) and the widest new-to-1-yr-old gap since 2015 (~$10,222)
- 5-year depreciation curves by powertrain segment (industry avg 41.8%, EVs 57.2%, trucks 34.2%, hybrids 35.4%)
- CPO program comparison (Hyundai/Kia/Genesis 10-yr/100K reinstated; Lexus L/Certified 2-yr unlimited B2B; Toyota Gold 7-yr/100K)
- EV federal credit status post-OBBBA (§30D, §25E, §45W terminated 2025-09-30; §30C charger credit active until 2026-06-30, restricted to eligible census tracts; new auto-loan-interest-deduction active 2025–2028 up to $10,000/yr, US-assembly required, lease-ineligible)
- NY State Drive Clean Rebate (point-of-sale, no income cap, $30M added April 21, 2026)
- Monroe County 8.0% combined sales tax, 2-year registration cycle, $21 inspection, $175 NY doc fee cap
- Rochester gas (regular $4.36, premium $5.18 as of May 2, 2026) and RG&E all-in $0.17/kWh
- Insurance averages for Rochester (full coverage $1,728–$2,220/yr)
- Maintenance averages by brand (RepairPal annual: Honda $428, Toyota $441, BMW $968, etc.)
- Major franchise dealer directory for Monroe County

---

## 3. Calculation methodology

### 3.1 Loan amortization

Standard amortizing-loan monthly payment (P&I):

```
PMT = P × (r × (1+r)^n) / ((1+r)^n − 1)
```

where `P` is principal, `r` is monthly rate (APR ÷ 12), and `n` is term in months. When APR = 0, PMT = P/n.

**Total interest** is `PMT × n − P`.

**Remaining balance after k payments** uses the closed-form:

```
B(k) = P × ((1+r)^n − (1+r)^k) / ((1+r)^n − 1)
```

This is needed when the client's hold period is shorter than the loan term — the tool subtracts terminal residual value from any remaining balance to compute equity at end of hold.

### 3.2 Lease monthly payment

The industry "money factor" formula:

```
Depreciation portion = (Cap Cost − Residual) / Term
Finance portion      = (Cap Cost + Residual) × Money Factor
Pre-tax monthly      = Depreciation + Finance
After-tax monthly    = Pre-tax × (1 + sales tax)
```

where:
- **Cap Cost** = MSRP minus any cap-cost reduction (down payment) and any OEM lease cash
- **Residual** = MSRP × residual percentage (e.g., 60%)
- **Money Factor** ≈ APR-equivalent ÷ 2400 (so MF 0.00064 ≈ 1.54% APR equivalent)

NY sales tax (8.0% in Monroe County) applies monthly to lease payments, not as a lump sum. Acquisition and disposition fees are added per OEM schedule (Toyota $750 acq / $350 disp; Honda $595 / $0; BMW $925 / $350; etc.).

The tool uses segment-average money factors and residuals as a fallback. When a specific model in the data layer matches, it uses that model's verified figures. Excess-mileage penalties (typically $0.20/mi mainstream, $0.25–0.30 luxury) are computed if the client's stated annual mileage exceeds 12,000.

### 3.3 Total cost of ownership

Annual ownership cost = Insurance + Maintenance + Fuel/Electricity + Inspection.

- **Insurance** uses Rochester full-coverage averages adjusted by class multiplier (luxury sedan 1.40x, luxury SUV 1.55x, EV 1.30x, minivan 0.95x). Used vehicles get a 15% discount to the class average; CPO gets 5% discount.
- **Maintenance** uses RepairPal annual averages by brand, scaled: new vehicles at 1.0x, CPO at 1.10x, used (3-yr+) at 1.35x to reflect out-of-warranty repairs, leases at 0.5x because the vehicle stays inside warranty.
- **Fuel** = (annual miles ÷ MPG) × $4.36/gal Rochester regular. Premium-required vehicles use $5.18/gal.
- **EV electricity** = (annual miles ÷ 3.5 mi/kWh) × $0.17/kWh RG&E all-in residential. This produces the ~$0.049/mi figure cited in the research; saving ~$1,500/yr at 12,000 miles vs. a 25-mpg ICE.
- **Inspection** = $21/yr Monroe County.

Total cost over hold = upfront drive-off + (monthly all-in × 12 × hold years) − terminal equity (residual or trade-in value at end of period). Cost-per-mile = net cost ÷ (annual miles × hold).

### 3.4 Depreciation / terminal value

Used as the basis for equity at end of hold and for the cash-purchase opportunity-cost comparison.

5-year industry averages (iSeeCars 2026 study):

| Segment | 5-yr depreciation |
|---|---|
| Trucks | 34.2% |
| Hybrids | 35.4% |
| Industry average | 41.8% |
| Midsize SUV | ~47% |
| EVs | 57.2% |

The tool applies an annualized geometric decay: `value(t) = MSRP × (1 − dep5yr/5)^t`. CPO units depreciate at ~70% of the new rate to reflect the steeper first-year drop having already occurred. Used 3-yr-old vehicles depreciate at 15%/yr from purchase as a simple approximation.

**Caveat:** the iSeeCars EV depreciation figures do not adjust for the now-expired $7,500 federal credit, which means used EV cost-of-ownership for original buyers was historically overstated. This is flagged in the research file but not yet adjusted in the model — a known refinement opportunity.

### 3.5 Opportunity cost overlay (the Icono lens)

The mechanic that distinguishes this tool. Two variants:

**Cash Purchase:** the tool displays the future value of the cash committed if it had instead been invested at the client's stated nominal expected return (default 6%, slider 2–12%). FV uses annual compounding: `FV = PV × (1 + r)^t`. The figure is shown as a negative — what the client foregoes by deploying the cash into a depreciating asset.

**Finance + Invest the Delta:** explicitly calculated as a paired strategy.

1. Path A (pay cash) requires `Total Cash` out of pocket today.
2. Path B (this path) finances at the best available APR (favoring 0% captive offers when the recommended vehicle qualifies — Toyota bZ family, Hyundai Ioniq 5, Kia EV6/EV9, Tesla Model 3 Standard, etc.) with a minimum 10% down, leaving `Invested Now = Total Cash − Min Down` to be invested at the client's expected return.
3. The tool computes:
   - FV of the lump sum invested today
   - Less the FV of an annuity stream representing the monthly payments forgone (since each monthly payment is dollars that *could* have stayed invested)
   - Net investment gain over the hold period
4. **Arbitrage** = Net investment gain − Total interest paid on the loan. Positive arbitrage means the cash works harder invested than it does paying off the vehicle.

The math is shown in the "How was this calculated?" expandable section on each result card, and the arbitrage figure is surfaced in a dedicated dashed-border callout.

This calculation assumes nominal pre-tax compounding. Real-world after-tax outcomes depend on account type (taxable brokerage vs. retirement account vs. HSA, etc.) and the client's marginal tax rate — the disclaimer flags this and directs the client to their advisor. It also assumes the client could and would actually hold the cash invested for the full hold period rather than spend it, which is itself a behavioral assumption Icono should test in conversation.

### 3.6 EV-specific stack

The tool applies, in order:

1. **Federal §30D / §25E / §45W: $0.** Terminated 9/30/2025 under OBBBA P.L. 119-21. Not available to May 2026 buyers.
2. **NY Drive Clean rebate** (point-of-sale at participating NY dealer, applied to pre-rebate price for sales tax purposes per NY rule):
   - $2,000 if range ≥200 mi AND base MSRP ≤ $42,000 (e.g., Chevy Equinox EV 1LT, Hyundai Ioniq 5 SE Std, Tesla Model 3 RWD if base ≤ $42K)
   - $1,000 if range 40–199 mi AND MSRP ≤ $42,000
   - $500 if MSRP > $42,000 OR range < 40 mi
   - 36-month minimum ownership/lease, no income cap, refilled with $30M April 2026
3. **§30C residential charger credit** (30% of installation up to $1,000, sunsets 6/30/2026, restricted to low-income (≥20% poverty) or non-urban (≥10% blocks outside urban area) census tracts. Most of suburban Monroe County including Pittsford, Brighton, Penfield, Webster, Greece is "urban" and likely ineligible. The tool flags this in the disclaimer and methodology rather than applying it automatically.
4. **New auto loan interest deduction** (above-the-line, 2025–2028, up to $10,000/year, US-assembly required, *not* lease-eligible). Income phase-outs: full to $100K single / $200K joint MAGI; eliminated at $150K single / $250K joint. The tool estimates the present-value tax benefit using a 22% bracket assumption for MAGI between $100K and $200K, 24% above that, and 12% below; the methodology document discloses this as an approximation.

### 3.7 Path selection logic

The tool selects two paths from six possible types using these rules, evaluated in order:

**Path 1 (always an ownership path):**
- If `condition === 'used'` OR (`condition === 'open'` AND hold ≥ 6 yrs AND not cash-rich) → Finance Used
- Else if `condition === 'cpo'` OR credit is challenged (sub-660) → Finance CPO
- Else → Finance New

**Path 2 (varies):**
- If cash-rich (cash ≥ 85% of new MSRP) AND expected return ≥ 5% AND not buying used → **Finance + Invest the Delta** (Icono signature path)
- Else if hold < 3 yrs AND mileage < 15K AND not buying used → Lease New
- Else if condition is open or CPO → Finance CPO (gap-pricing argument)
- Else if hold ≥ 6 yrs AND condition is new → Finance Used (hold-it-out case)
- Else if mileage < 15K AND comfortable with mileage caps (q4 ≥ 4) → Lease New
- Else if challenged credit → Finance Used
- Else if cash-rich → Finance + Invest the Delta
- Else → Lease New

**Hard rules enforced regardless of preferences:**
- High-mileage drivers (≥15K/yr) never see lease as primary; if a lease appears, the excess-mileage penalty is shown explicitly
- Holds <3 yrs always make lease eligible
- Subprime credit always biases away from new finance toward used

### 3.8 Fit Score (0–100)

Each path starts at a base of 60 and is adjusted up or down by:

- **Budget alignment:** +12 if monthly all-in ≤ stated budget; −4 if 1–10% over; −14 if >10% over
- **Lease-specific:** +10 if hold ≤ 3.5 yrs; −16 if hold ≥ 5; −18 if mileage ≥ 15K; +6 if "latest tech" preference ≥ 4
- **CPO-specific:** +6 if used-comfort ≥ 3; +4 if cash on hand < $8K
- **Used-specific:** +12 if used-comfort ≥ 4; −16 if used-comfort ≤ 2; +6 if hold ≥ 5
- **Finance New:** +6 if "latest tech" ≥ 4; +4 if interest deduction ≥ $500; +4 if hold ≥ 6
- **Cash:** +8 if cashflow-flexibility preference ≤ 2 (meaning the client values lowest long-term cost); −6 if cashflow ≥ 4
- **Invest-the-delta:** +14 if expected return clears APR by ≥3 pts; +8 if a captive 0% offer matches; +6 if arbitrage > 0; −8 if arbitrage < 0
- **EV match:** +6 if powertrain preference is BEV; +4 if vehicle qualifies for full $2,000 NY Drive Clean tier

Score is clamped to [15, 98]. The plain-English rationale shown to the client uses the top three contributors.

---

## 4. Display and transparency

Every figure on the result screens has a corresponding "How was this calculated?" expander showing:

- The actual numbers used (MSRP, tax, fees, principal, APR, term, MF, residual, etc.)
- The formula applied
- The line-item breakdown of monthly costs
- Caveats specific to that path (e.g., excess-mileage penalty if applicable, EV credit termination note for any EV lease, US-assembly check for the new auto loan interest deduction)

The Chart.js TCO line chart shows cumulative cost over the hold period, allowing the client to see when each path crosses the other.

---

## 5. Refresh cadence

| Data Category | Volatility | Cadence | Owner |
|---|---|---|---|
| National auto loan APR averages | 🔴 High | Quarterly (when Experian releases) | Research |
| Best-rate lender APRs (PenFed, Navy, ESL) | 🔴 High | Weekly | Research |
| OEM captive offers | 🔴 High | Monthly (1st of month) | Research |
| Lease MFs and residuals | 🔴 High | Monthly | Research |
| Lease cash / EV OEM cash | 🔴 High | Monthly (subject to OEM whim) | Research |
| Manheim Used Vehicle Value Index | 🟡 Medium | Mid-month + month-end | Research |
| Used segment ATPs | 🟡 Medium | Monthly | Research |
| Inventory days' supply | 🟡 Medium | Monthly | Research |
| Rochester gas prices | 🔴 High | Weekly | Research |
| Electricity rates (RG&E) | 🟡 Medium | Quarterly (PTC reset cycle) | Research |
| Insurance averages | 🟢 Low | Quarterly | Research |
| Maintenance averages | 🟢 Low | Annually | Research |
| CPO program terms | 🟢 Low | Annually | Research |
| Federal EV tax law | ⚪ Rare | On policy change only | Compliance |
| NY Drive Clean rebate parameters | ⚪ Rare | Quarterly verify | Compliance |
| NY sales tax / fees / lemon law / doc fee cap | ⚪ Rare | Annually | Compliance |
| Dealer directory | ⚪ Rare | Annually | Research |

The README explains the file-level mechanics for each refresh.

---

## 6. Known limitations and disclosures

These are the items that should be reviewed alongside this document during compliance review.

1. **Used EV depreciation curves** are based on iSeeCars 2026 data that does not net out the now-expired $7,500 federal credit, modestly overstating new-EV cost-of-ownership for the original buyer cohort. Will be refined when post-OBBBA cohort data is published.
2. **Auto loan interest deduction** estimates use bracket approximations (12/22/24%) tied to income brackets — actual deduction depends on the client's full tax picture. Always direct the client to their CPA.
3. **§30C charger credit** is not auto-applied because eligibility is census-tract-specific; suburban Monroe County is mostly excluded. Disclosed in the disclaimer; available as an explicit conversation topic with the advisor.
4. **Live listings** are deep-link searches, not scraped inventory. Cars.com / CarGurus / Autotrader / CarMax / Carvana actively block scraping and have no free public APIs; honest deep links are the fiduciary-compliant approach. Inventory thinness or pricing anomalies are noted in the research file but are not yet surfaced in the tool — a future enhancement.
5. **Investment expected return** is a single nominal pre-tax number. The tool intentionally does not model taxes on investment gains, account-type tax wrappers, or sequence-of-returns risk — this is conversation territory for the advisor.
6. **Insurance estimates** use Rochester averages with class multipliers, not driver-specific quotes. Real premiums depend on driving record, household composition, prior claims, and credit-based insurance score. The figure is a reasonable mid-range placeholder for planning, not a quote.
7. **Lease money factors and residuals** in the data layer are May 2026 snapshots. Programs reset roughly the 1st of each month — the data layer must be refreshed before the tool is re-run for a new client meeting in a new month.
8. **NY-specific:** the model assumes Monroe County tax (8.0%) and Rochester pricing. Out-of-state or other-NY-county clients should override the data layer or be marked as outside the tool's calibration zone.
9. **Federal EV credit termination** is a structural change. Any client who *contracted* for an EV with binding payment on or before 2025-09-30 may still qualify under the grandfather rule even if delivery is after — that case is rare in May 2026 and is not modeled.
10. **The recommendation is two paths, not "the answer."** The tool's job is to surface a clean comparison; the advisor's job is to weigh non-financial factors (family logistics, garage capacity, charging access, employer benefits, business-use treatment) that the tool cannot see.

---

## 7. Disclosure language used in the tool

The disclaimer block on the results screen reads (paraphrased here, exact text in the HTML file):

> This is an educational planning tool from Iconoclastic Capital Management and does not constitute personalized investment, tax, legal, or financial advice. Rates, residuals, money factors, incentive amounts, and inventory shift constantly — we date-stamp our data layer and refresh on the cadence shown in the methodology document. Federal EV tax credits (§30D, §25E, §45W) terminated September 30, 2025 under the One Big Beautiful Bill Act and are not available to May 2026 buyers; the §30C residential charger credit sunsets June 30, 2026 and is restricted to eligible census tracts. The opportunity-cost overlay assumes your stated nominal expected return compounds annually before tax; actual after-tax outcomes depend on account type and tax bracket. Results should be reviewed with your Icono advisor before acting.

---

## 8. Change log

| Date | Version | Author | Notes |
|---|---|---|---|
| 2026-05-08 | 1.0 | Icono | Initial release. Data layer current as of May 8, 2026. |

---

*Right here with you.*
*Iconoclastic Capital Management · Fee-only fiduciary RIA*
