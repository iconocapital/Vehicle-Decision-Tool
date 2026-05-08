# README — Updating the Embedded Data Layer

**Tool:** `2026-05-08-icono-vehicle-decision-tool.html`
**Methodology:** see `2026-05-08-icono-vehicle-tool-methodology.md`
**Last refreshed:** 2026-05-08

This document is the operator's guide for keeping the Vehicle Acquisition Path Planner current. It is not for clients.

---

## 1. Where the data lives

All market-derived data sits in a single `const DATA = { ... }` JavaScript object inside the HTML file's `<script>` block. Open the HTML in any text editor, search for `EMBEDDED DATA LAYER` (the comment banner just above the object), and you will see the entire data layer in one place.

Two strings to update on every refresh:

| Location | What to change |
|---|---|
| `<title>` and `dataStamp` `<div>` (top of page) | The visible "Data refreshed YYYY-MM-DD" stamp |
| `DATA.asOf` | The internal date string that drives the footer text |

The footer auto-renders from `DATA.asOf`, so you only need to update the human-visible stamp and the `asOf` field for the date to flow through everywhere.

---

## 2. Refresh checklist by cadence

### Weekly (every Monday morning)

These move fast enough that a stale week makes the tool wrong.

- [ ] **Rochester gas prices** (`DATA.tco.fuel.rochesterRegular`, `rochesterPremium`)
  - Source: AAA Gas Prices · https://gasprices.aaa.com/?state=NY (look at Rochester metro)
  - Cross-check: GasBuddy Rochester
- [ ] **Best-rate lender APRs** (`DATA.loanRates.monroeBest`)
  - PenFed: https://www.penfed.org/auto-loans (Car Buying Service rate page)
  - Navy Federal: https://www.navyfederal.org/loans-cards/auto-loans/auto-loan-rates.html
  - ESL FCU Rochester: https://www.esl.org/rates/loan-rates (auto loans section, by vehicle age)
  - Reliant CU, Summit FCU rate pages

### Monthly (1st business day of each month)

OEM lease and finance programs reset on the 1st. Refresh before any client meeting in a new month.

- [ ] **OEM captive offers** (`DATA.loanRates.captiveOffers`)
  - Each manufacturer's "Current Offers" page (toyota.com/local-specials, honda.com/specials, hyundaiusa.com/offers, kia.com/offers, etc.)
  - Cross-check: CarsDirect, Edmunds incentive trackers
  - Note expiration dates explicitly — the artifact does not auto-expire offers
- [ ] **Lease money factors and residuals** (`DATA.lease.avgMFforSegment`, `avgResidualPct`, model-specific entries)
  - Source: Edmunds Forums "Ask the Lease Expert" thread (current month)
  - Cross-check: Leasehackr forums, RealCarTips
- [ ] **EV lease cash** (`DATA.lease.evLeaseCash`)
  - Source: same OEM offers pages, plus CarsDirect EV deals roundup (monthly)
  - These are now OEM marketing dollars (not federal §45W, which terminated 2025-09-30) and are the most volatile single category
- [ ] **Manheim Used Vehicle Value Index** (`DATA.used.manheimIndex`, `manheimMoM`)
  - Source: Cox Automotive · https://www.coxautoinc.com/insights/manheim-used-vehicle-value-index/
  - Released mid-month for prior month
- [ ] **Used segment ATPs** (`DATA.used.avgUsed3yr`, `avgUsed1yr`, `newUsedGap`)
  - Source: Edmunds Used Vehicle Report (quarterly), Cox Automotive monthly press releases
- [ ] **Inventory days' supply** (referenced in methodology only; not in DATA object yet)
  - Source: Cox Automotive monthly inventory report
  - If you add this to the data layer, expose it as `DATA.market.daysSupply`

### Quarterly

- [ ] **National auto loan APR averages** (`DATA.loanRates.nationalNew`, `nationalUsed`)
  - Source: Experian "State of the Automotive Finance Market" report
  - Released ~10 weeks after quarter close (Q4 2025 was released March 5, 2026)
- [ ] **RG&E electricity rates** (`DATA.ev.rgeKwh`)
  - Source: rge.com (residential PTC supply rate; verify it has not reset)
  - PTC currently $0.10750/kWh through 2026-07-31 — refresh August 2026
- [ ] **Insurance averages** (`DATA.tco.insAnnual`)
  - Source: Bankrate, Experian, MoneyGeek Rochester full-coverage averages

### Annually

- [ ] **Maintenance averages** (`DATA.tco.maintAnnual`)
  - Source: RepairPal annual averages by brand
- [ ] **CPO program terms** (`DATA.used.cpoBest`, expand if needed)
  - Source: each manufacturer's CPO landing page; U.S. News annual best-CPO ranking
- [ ] **NY-specific fees** (`DATA.tco.titlePlate`, `regSedan2yr`, `regSuv2yr`, `monroeUseTax`, `inspectionAnnual`, `docFeeCapNY`, sales tax)
  - Source: NY DMV fee schedule, NYS Pub 838
- [ ] **Dealer directory** (`DATA.dealers`)
  - Source: each brand's dealer locator; Google Maps verification
- [ ] **Drive Clean rebate parameters** (`DATA.ev.nyDriveClean`)
  - Source: nyserda.ny.gov/All-Programs/Drive-Clean-Rebate-For-Electric-Cars-Program

### On-policy-change

- [ ] **Federal EV credit status** (`DATA.ev.fedNewCredit`, `fedUsedCredit`, `fedLeaseCredit`, `fedChargerCredit30C`, `autoLoanInterestDeduction`)
  - All currently aligned with OBBBA P.L. 119-21 as signed 2025-07-04
  - Watch for: Treasury / IRS guidance updates, any subsequent legislation
  - The §30C residential charger credit sunsets 2026-06-30 — flip `active` to `false` and update the disclaimer in the HTML around July 2026
  - The new auto loan interest deduction sunsets after tax year 2028 — same drill in 2029

---

## 3. How to make a change safely

The data layer is plain JSON inside JavaScript. There is no build step. To change a value:

1. Open `2026-05-08-icono-vehicle-decision-tool.html` in a text editor (VS Code, Sublime, Notepad++).
2. Locate the field via search (e.g., search `rochesterRegular` to find the gas price).
3. Edit the value, save the file.
4. Rename the file with the new date prefix: `YYYY-MM-DD-icono-vehicle-decision-tool.html`.
5. Update the visible stamp and `DATA.asOf` to match.
6. Open the file in a browser and walk through one full client flow to verify nothing broke.

**Do not change** the structure of the `DATA` object (key names, nesting). The recommendation engine reads specific keys; renaming will silently break path generation. If you need a new field, add it alongside existing fields rather than reorganizing.

**Do not commit client-specific data** (income brackets, ZIPs, names) into the file. The tool is delivered identically to every client; the per-client state lives in browser memory only.

---

## 4. Verification walkthrough (run after every refresh)

A 90-second sanity check that catches most refresh bugs.

1. Open the HTML in Chrome.
2. Click `Begin →`.
3. Group A: pick **SUV / Crossover**, **Hybrid**, **New**, leave models blank, check **AWD**, choose **12k–15k mi**, **5–8 yrs** hold.
4. Group B: ZIP **14618**, income **$100k–$150k**, **Excellent (760+)**, $20,000 cash, $800/mo budget, **None** for current vehicle, $0 trade, expected return **6%**, NY, joint, **None** for business use.
5. Group C: leave all sliders at 3.
6. Click **Show my two paths**.

**Expected output:**
- Vehicle: Toyota Grand Highlander XLE AWD (since 3-row was not picked, this should be the RAV4 Hybrid path; if you checked 3-row it'll be Grand Highlander)
- Path 1: Finance New, ~$700–$900/mo all-in, drive-off ~$8,000–$10,000
- Path 2: Lease New OR Cash + Invest depending on the cashRich heuristic
- Both fit scores between 50 and 90
- TCO chart renders with two lines crossing somewhere around year 3–4
- Listings grid shows 6 deep-link cards (Cars.com, CarGurus, Autotrader, CarMax, Carvana, TrueCar)
- Dealer list shows 3+ Toyota dealers (Hoselton, Bob Johnson, West Herr) plus CarMax

If any of those go wrong, the most likely culprit is a malformed JSON value (trailing comma, missing quote, NaN). Open Chrome DevTools (F12), check the Console tab — JS errors will point you to the line number.

---

## 5. Adding a new vehicle model

The tool uses model heuristics in `pickVehicleModel(seg)`. To add a specific model that should be recommended in certain conditions, edit that function. The pattern is:

```js
if(state.body === 'truck' && state.features.has('tow')){
  return {
    name: "Ford F-150 Lariat 4x4",
    msrpNew: 65000,
    msrp1yr: 56000,
    msrp3yr: 44000,
    brand: "ford",
    usAssembly: true       // affects auto loan interest deduction eligibility
  };
}
```

For EVs you can additionally set:
- `driveCleanTier: 2000` if the model qualifies for the full NY rebate (range ≥200 mi AND MSRP ≤ $42K)
- `driveCleanTier: 1000` for the middle tier
- omit for the default fallback ($500 if MSRP > $42K)

The brand string must match a key in `DATA.tco.maintAnnual` and `DATA.dealers` for the maintenance estimate and dealer list to populate correctly. Add new brand keys to those tables as needed.

---

## 6. Adding a new acquisition path

The path system is modular. Each path is built by a function that returns an object with the standard shape (`type`, `vehicle`, `description`, `upfront`, `monthly`, `apr`, `term`, `principal`, `totalInt`, `annualOwnCost`, `hold`, `valueAtEnd`, `equity`, `remainingPrincipal`, `totalOutOfPocket`, `netCost`, `costPerMile`, `math`).

To add a path (example: a "Subscription / Care-by-Volvo" type service):

1. Write a `buildSubscription(vehicle, seg, ctx)` function alongside the existing builders, returning the standard shape.
2. Add a key to `PATH_BUILDERS`: `subscription: buildSubscription`.
3. Update `selectPaths()` with the trigger logic for when this path should appear.
4. Optionally update `fitScore()` to add path-type-specific scoring rules.

The renderer is generic; new paths render automatically.

---

## 7. Files in the deliverable

| File | Purpose | Audience |
|---|---|---|
| `2026-05-08-icono-vehicle-decision-tool.html` | The tool | Clients |
| `2026-05-08-icono-vehicle-tool-methodology.md` | Calculation methodology, sources, limitations, disclosures | Compliance, advisors |
| `2026-05-08-icono-vehicle-tool-readme.md` | This file | Operators (whoever maintains the tool) |

---

## 8. Maintenance ownership

Recommended:
- **Weekly refresh:** delegate to whoever runs Icono's market data process; 15 minutes
- **Monthly refresh:** primary maintainer; 45 minutes (incentive resets are the bulk of it)
- **Quarterly refresh:** primary maintainer + compliance review; 90 minutes
- **Annual refresh:** full compliance + advisor review of methodology document

The tool's value to clients depends on data being fresh. A stale data layer with the May 2026 stamp and June lease MFs is worse than no tool at all.

---

*Right here with you.*
*Iconoclastic Capital Management*
