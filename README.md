# British Airways Lounge Eligibility Model
### Heathrow Terminal 3 — Demand Forecasting Tool

---

## Project Overview

Lounge access is a core part of the premium travel experience at British Airways. As BA plans for future operations at Heathrow Terminal 3, the Airport Planning team needs a way to forecast how many passengers will be eligible to use each lounge tier on a typical operating day — without relying on specific flight numbers or aircraft types that may change.

This project builds a **reusable lounge eligibility lookup table** that estimates the percentage of passengers qualifying for each lounge tier, grouped by flight category. The model is designed to be forward-looking: any future schedule can be run against it using only two attributes that are always known at planning time.

This was completed as part of the **British Airways Data Science Virtual Experience Programme** on Forage.

---

## The Problem

British Airways operates three lounge tiers at Terminal 3:

| Tier | Lounge | Eligible Passengers |
|------|--------|-------------------|
| 1 | Concorde Room *(hypothetical at T3)* | First Class, BA Premier cardholders, BA Gold Guest List |
| 2 | First Lounge | BA Gold members |
| 3 | Club Lounge | BA Silver cardholders, Club World (business class) |

The challenge: future schedules are unpredictable. A model tied to specific routes or aircraft becomes obsolete every season. The goal was to find grouping variables that are **stable, always known, and genuinely predictive of lounge demand**.

---

## Approach

### Grouping Strategy

After testing several grouping approaches — including destination region, aircraft type, and route category — the final model uses two dimensions:

**1. Route length** (`SHORT` vs `LONG` haul)
The strongest structural driver of lounge eligibility. Long-haul aircraft carry dedicated First Class cabins and a higher absolute count of business seats. Short-haul aircraft (predominantly A319/A320 family) carry fewer premium seats but serve a disproportionately high share of BA loyalty cardholders — frequent business travellers on intra-European routes.

**2. Time of day** (Morning / Lunchtime / Afternoon / Evening)
Captures behavioural passenger mix. Early morning short-haul flights attract frequent business travellers with accumulated loyalty status. Evening and leisure-hour flights skew toward holiday passengers with lower lounge eligibility rates.

This produces **8 groups** that cover every possible flight combination.

### Why Region Was Excluded

Regional destination (Europe / North America / Asia / Middle East) was explicitly tested and rejected. Within long-haul flights, the Tier 2 eligibility rate varied by only **0.1 percentage points** across all three regions (2.85%–2.95%). Adding region would double the long-haul table complexity from 4 to 12 rows while explaining virtually none of the within-group variance.

The reason: BA's Executive Club loyalty status is earned through flight frequency, not destination. A Gold cardholder flying to Dubai and one flying to JFK are equally eligible — the route is irrelevant to their lounge access. Region would be a meaningful variable if we had destination-level booking class fill data, or if this were a different airline where specific corporate routes dominate.

### Tier 1 Note

There is currently no Concorde Room at Terminal 3. Tier 1 estimates in this model reflect the volume of passengers who *would qualify* based on their travel class and loyalty status. This is intentionally modelled as a **demand signal** to inform whether a future Tier 1 facility could be justified — not as a description of existing infrastructure.

---

## The Lookup Table

Derived from 10,000 flights in the BA Summer 2025 schedule dataset. Percentages represent the average ratio of eligible passengers to total seats within each group.

| Group | Tier 1 % | Tier 2 % | Tier 3 % |
|-------|----------|----------|----------|
| Short-Haul – Morning (06:00–11:59) | 0.3% | 4.4% | 16.7% |
| Short-Haul – Lunchtime (12:00–13:59) | 0.4% | 4.5% | 17.3% |
| Short-Haul – Afternoon (14:00–17:59) | 0.3% | 4.3% | 16.6% |
| Short-Haul – Evening (18:00–23:59) | 0.3% | 4.5% | 17.0% |
| Long-Haul – Morning (06:00–11:59) | 0.2% | 3.0% | 11.3% |
| Long-Haul – Lunchtime (12:00–13:59) | 0.2% | 2.9% | 11.1% |
| Long-Haul – Afternoon (14:00–17:59) | 0.2% | 2.9% | 11.2% |
| Long-Haul – Evening (18:00–23:59) | 0.2% | 2.9% | 11.0% |

**How to use this table:** For any future flight, identify its haul type and departure time window, look up the corresponding row, and multiply each tier percentage by the expected passenger count to get an estimated eligible headcount per lounge.

---

## Key Findings

- Short-haul flights show **~50% higher lounge eligibility rates** than long-haul across all tiers, despite carrying fewer absolute passengers — driven by the concentration of frequent business travellers on European routes
- Time of day produces only modest variation within each haul category (< 0.5pp across tiers), suggesting that route length is the dominant structural driver
- Destination region contributes negligible additional predictive power in BA's network, where loyalty status is frequency-based rather than route-based
- Tier 3 (Club Lounge) represents by far the largest demand volume — roughly 4–6× Tier 2 headcount per flight — making it the primary capacity planning concern

---

## Files

```
ba-lounge-project/
│
├── British_Airways_Summer_Schedule_Dataset.xlsx   # Raw schedule data (10,000 flights)
├── Lounge_Eligibility_Lookup_Completed.xlsx       # Final model + written justification
└── README.md                                      # This file
```

**Lounge_Eligibility_Lookup_Completed.xlsx** contains two sheets:
- `Lounge Eligibility Lookup Table` — the 8-group model with tier percentages and example destinations
- `Justification` — written responses to the four methodology questions

---

## Tools Used

- **Python (pandas)** — data loading, group-by aggregation, variance analysis, percentage calculations across 10,000 rows
- **Microsoft Excel** — final lookup table formatting, template completion, PivotTable validation
- **openpyxl** — programmatic Excel file generation with BA brand formatting

---

## About This Project

This project was completed as **Task 1** of the British Airways Data Science Virtual Experience on [Forage](https://www.theforage.com). The task brief asked for a lounge eligibility lookup table and written justification of the modeling approach — with emphasis on scalability to unknown future schedules.

The modelling decisions here prioritise **interpretability and robustness** over precision. A planner with no data background should be able to pick up this table and apply it to a new schedule in under five minutes.
