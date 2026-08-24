# Airline Performance Analytics — Power BI Project

An end-to-end Power BI report analyzing three years (2023–2025) of operations, revenue, customer
loyalty, fleet health, and satisfaction data for a synthetic US carrier modeled on a Delta-style
hub-and-spoke network. Built as a portfolio project to demonstrate star-schema data modeling, DAX
measure design, and multi-page report/UX design in Power BI.

> **Data note:** the underlying dataset is synthetically generated for portfolio/demonstration
> purposes — no real passengers, employees, or flights are represented. Seasonality, delay patterns,
> and cost structures are modeled to resemble realistic airline behavior, but figures should not be
> read as real-world data. See **Data & Known Limitations** below.

## Contents
- [Overview](#overview)
- [Tools Used](#tools-used)
- [Data Model](#data-model)
- [Report Pages](#report-pages)
- [Key Measures](#key-measures)
- [Key Insights](#key-insights)
- [Recommendations](#recommendations)
- [Data & Known Limitations](#data--known-limitations)
- [Setup](#setup)

## Overview

The report is a 6-page, single-file Power BI (`.pbix`) analytics application covering:

| Area | Business question it answers |
|---|---|
| Overview | How is the airline performing right now, across ops, revenue, and satisfaction? |
| Flight Operations | Where and why are delays and cancellations happening? |
| Revenue | What's driving revenue — fare mix, channel, route, season? |
| Customer & Loyalty | Who are the customers, how loyal are they, who's most valuable? |
| Fleet & Maintenance | Which aircraft are costing the most, and is the fleet aging into risk? |
| Customer Satisfaction | What's driving NPS up or down, and where are complaints concentrated? |

Every page shares a consistent navigation bar, filter bar (Year / Hub / Route / Fare Class / City /
Loyalty Tier), and a custom report theme, so the six pages read as one coherent application rather
than six separate reports.

## Tools Used

- **Power BI Desktop** — data modeling, DAX, report/UX design
- **Power Query** — data import and type transformation
- **DAX** — ~50 custom measures across operations, revenue, loyalty, fleet, and satisfaction analysis
- **Custom Power BI theme (JSON)** — consistent brand palette and typography applied report-wide
- **Star schema data modeling** — dimensional design with fact/dimension separation and role-playing
  dimension handling (origin/destination airports)

## Data Model

15 tables in a star schema, imported via CSV:

**Dimensions:** `Date`, `Airport`, `Aircraft`, `Route`, `Customer`, `FareClass`, `BookingChannel`,
`DelayReason`, `CancellationReason`

**Facts:** `Flights` (flight-level operational grain), `Bookings` (ticket-level revenue grain),
`CustomerSatisfaction` (survey response grain), `Maintenance` (maintenance event grain), `FuelPrices`
(monthly grain)

**Supporting:** `MeasuresTable` — a dedicated, unhidden-column table holding all DAX measures,
separated from the data tables for a clean Model view

Filtering flows through a single consistent spine (`Date` / `Route` / `Airport` → `Flights` →
`Bookings` / `CustomerSatisfaction` → `Customer` / `FareClass` / `BookingChannel`), avoiding the
multi-path ambiguity that a naive design (with direct `Date`-to-every-fact-table relationships) would
introduce.

## Report Pages

**Overview** — KPI summary (Total Revenue, Active Customers, On-Time Performance %, Cancellation
Rate %), monthly OTP trend, revenue by fare class, flight volume by state/hub map.

**Flight Operations** — cancellation/delay trend by month, cancellations by reason (ribbon chart),
on-time performance matrix by origin/destination, delay minutes by reason, diversion rate.

**Revenue** — total/ancillary/bag fee revenue, RASM, revenue by hub and by route type, revenue by
booking channel, YoY revenue trend, fare class revenue mix by month.

**Customer and Loyalty** — loyalty points issued trend, active customers, repeat customer rate,
customers by loyalty tier, ticket-count-vs-spend scatter (RFM-style view), top customers table.

**Fleet and Maintenance** — maintenance cost by aircraft model, fleet age vs. maintenance cost vs.
downtime (scatter), downtime by maintenance type, aircraft-level detail table.

**Customer Satisfaction** — NPS score and trend, promoter/passive/detractor split, satisfaction
sub-scores (on-time, crew, comfort, beverage) side by side, arrival delay vs. satisfaction by route,
complaint category breakdown.

## Key Measures

A representative sample from the ~50 measures in `MeasuresTable`:

| Category | Measures |
|---|---|
| Operations | Completed Flights, Cancellation Rate %, On-Time Performance %, AVG Arrival Delays (min), Diversion % |
| Revenue | Total Revenue, Total Ticket Revenue, Total Ancillary Revenue, Total Bag Fee Revenue, Revenue Per ASM (RASM), Revenue YoY % |
| Loyalty | Active Customers (in period), Total Customers, Repeat Customer Rate %, Total Loyalty Points Issued, AVG Ticket per Customer |
| Fleet | Total Maintenance Cost, Maintenance Cost per Aircraft, Fleet Age, Total Downtime Hours, Flights per Aircraft |
| Satisfaction | NPS Score, Promoters % / Passives % / Detractors %, AVG Customer Satisfaction, Would Recommend %, Total Responses |

## Key Insights

Findings the report is designed to surface from the underlying (synthetic) data patterns:

- **Winter is the highest-risk season for operations.** Cancellation and delay rates rise
  noticeably in December–February, concentrated at congestion-prone hubs (JFK, DTW, MSP, BOS).
  Cancellation reason mix also shifts toward weather in these months.
- **Delay's effect on satisfaction only shows up at the individual-flight level, not the
  route-average level.** Because delay minutes are right-skewed (a few severe outliers pull the
  mean up while most flights stay on time), a route's *average* delay is a weak predictor of
  satisfaction — a *rate*-based metric (e.g., % of flights severely delayed) correlates far better.
  This is a genuine modeling lesson surfaced while building the satisfaction-vs-delay scatter chart.
- **Revenue is concentrated in Main Cabin and Comfort+**, with Basic Economy and premium cabins
  (First/Delta One) each a smaller share — ancillary revenue (bags, upgrades) adds a meaningful
  layer on top of ticket price alone.
- **A small number of aircraft models (older 757s/767s) disproportionately drive maintenance cost
  and downtime**, visible directly in the fleet-age-vs-cost scatter — the kind of pattern that
  motivates fleet renewal planning.
- **Complaint volume concentrates in a handful of categories** (delay/cancellation and customer
  service lead), rather than being evenly spread — worth prioritizing operationally over more minor
  categories like cleanliness.

## Recommendations

1. **Shift winter resourcing** (crew reserves, de-icing capacity, rebooking staff) toward the hubs
   with the highest weather-driven cancellation rates, ahead of the December–February peak.
2. **Track and report delay *rate*, not just average delay minutes**, in operational scorecards —
   the average is systematically misleading for anything correlated with the tail of the
   distribution (satisfaction, complaints, rebooking cost).
3. **Investigate route/fare-class combinations with high load factor but low RASM** as candidates
   for fare optimization — the Revenue page's route table is built to surface exactly this.
4. **Prioritize maintenance/retirement planning for the oldest high-cost aircraft models** identified
   in the fleet scatter, rather than spreading maintenance budget evenly across the fleet.
5. **Target the top 1–2 complaint categories directly** with operational fixes (e.g., baggage
   handling process, delay communication) rather than broad customer-experience initiatives, since
   complaint volume is concentrated rather than evenly distributed.

## Data & Known Limitations

- **`Bookings` is a ~7.5% sample of sold seats per flight**, not a complete record — generated this
  way to keep the file performant while remaining statistically representative. Absolute revenue
  totals are scaled up (`× 1/0.075`) in the relevant measures; ratios and trends do not need scaling.
  Do not sum `Bookings`-derived revenue and a separately-scaled figure together, or bag fee revenue
  on top of ancillary revenue — bag fee is a subset of ancillary revenue, not additive to it.
- **`CustomerSatisfaction` is a ~9% sample of non-cancelled tickets.**
- **Fuel price is a monthly national average**, not per-route/per-flight actual burn.
- **Crew (captain/first officer) are randomly assigned per flight**, with no real crew
  scheduling/rest-rule logic.
- **All fares and costs are synthetic** — they track distance, season, and class realistically but
  are not calibrated to real published airfare or MRO cost data.
- **Origin/destination airports and captain/first officer are role-playing dimensions** — each
  requires either duplicated dimension tables or `USERELATIONSHIP()` in DAX to activate both
  directions simultaneously; only one relationship per pair is active by default in Power BI.

## Setup

1. Open `airtravel.pbix` in Power BI Desktop (2023.x or later recommended).
2. If prompted to update data source paths, repoint the CSV sources in **Transform data → Data
   source settings** to your local copy of the source files.
3. Refresh the model (**Home → Refresh**) to load current data.
4. All report-level filters (Year, Hub, Route, Fare Class) are synced across the operational pages
   via **View → Sync slicers** — no additional setup needed to navigate between pages.

---

*Built as a Power BI portfolio project. Feedback and suggestions welcome via issues/PRs.*
