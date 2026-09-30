# Understanding Singapore's COE Bidding Results and Vehicle Population

## 1. Context

### 1.1 What is a COE?

In Singapore, anyone who wants to register a vehicle must first own a **Certificate of Entitlement (COE)**. A COE gives the right to own and use a vehicle for **10 years**. 

The Land Transport Authority (LTA) limits how many COEs exist, which caps the total number of vehicles on Singapore's roads. Since February 2018, the [vehicle population growth rate has been set at **0% per annum** for Cat A, B and D, while that of Cat C will remain at 0.25% per annum.
This will remain in place until 31 Jan 2028](https://www.lta.gov.sg/content/dam/ltagov/who_we_are/statistics_and_publications/statistics/pdf/COEQuotaAllocationRV_2026_Onwards.pdf).

COEs are allocated through an **open bidding system**. [Bidding exercises are held twice a month](https://www.motorist.sg/article/5199/coe-open-bidding-schedule-for-2026). Each vehicle category has its own quota, and the price is set by the bids.

### 1.2 Who the System Affects

The Government and policymakers set the supply. The goal is to *keep the vehicle population in check to manage fair use of limited road space*. The concerns would be: what is the growth limit to hold and at what cost to affordability?

Future car owners pay the price. COE premiums make up a large share of a car's price in Singapore. The main concern would be: *when to bid for a car, what car type to buy, and how much to budget for it?* 

The bidding data shows how the quota set by the government meets demand from the market. The population data shows what that policy produces over time: how many vehicles there are, and of what kind.

---

## 2. The Datasets

This project uses two related datasets. Both are published by the Land Transport Authority (LTA) on [data.gov.sg](https://data.gov.sg/).

### 2.1 COE Bidding Results Prices

The `COEBiddingResultsPrices.csv` dataset shows the result of each COE bidding exercise. There are **1,980 rows** where each row represents one category in one bidding exercise, from Jan 2010 to Sep 2026 (with two bidding processes in a month).
| Field | Type | Description | Values / example | Role |
| --- | --- | --- | --- | --- |
| `month` | Text (`YYYY-MM`) | Month of the bidding exercise | `2010-01` to `2026-09` | Time |
| `bidding_no` | Integer | The first or second exercise of the month | `1`, `2` | Time |
| `vehicle_class` | Text | COE category (see [Vehicle Categories](#####vehicle-categories)) | `Category A` to `Category E` | Segment |
| `quota` | Integer | Number of COEs offered in the exercise | 43 to 2,272 | **Supply** (set by policy) |
| `bids_success` | Integer, stored as text* | Number of COEs actually allocated | 39 to 2,246 | Outcome |
| `bids_received` | Integer, stored as text* | Number of valid bids submitted | 65 to 4,545 | **Demand** |
| `premium` | Integer (SGD) | Quota Premium (QP): the lowest successful bid | 852 to 158,004 | **Price** (outcome) |

\* Some values have a thousands separator (e.g. `"1,007"`): 41 rows in `bids_success` and 109 in `bids_received`. These commas need to removed before converting these columns into numbers for analysis. 

##### Vehicle Categories
| Category | Covers |
|---|---|
| **A** | Cars up to 1600cc and 97kW (for electric cars: up to 110kW) |
| **B** | Cars above 1600cc or 97kW (for electric cars: above 110kW) |
| **C** | Goods vehicles and buses |
| **D** | Motorcycles |
| **E** | Open: any vehicle except motorcycles |

##### Understanding relationships between fields

```text
     POLICY                             MARKET
(set by LTA)                   (buyers, mostly via dealers)
      │                                   │
    quota  ─────────►  AUCTION  ◄───── bids_received
                          │
            ┌─────────────┴─────────────┐
        premium                    bids_success
  (clearing price)               (≤ quota; any leftover
                                  rolls into later exercises)
```

- **Supply vs demand:** `quota` is the supply and `bids_received` is the demand. Their ratio shows how competitive an exercise was.
- **Price discovery:** `premium` is the lowest bid that still won. Every successful bidder pays this same price, not the amount they bid (a uniform-price auction).
- **Allocation:** `bids_success` is usually at or just below `quota`. Any gap is left unallocated and rolls into later exercises.
- **Quota looks backward:** the quota is based mainly on how many vehicles were deregistered recently. A COE lasts 10 years, so supply tends to rise and fall in waves that follow purchases made about a decade earlier.
- **Categories are linked:** Category E can be used for vehicles in A, B or C, so its premium tends to track the most expensive of those, usually B.
- **Derived price, not in the dataset:** the Prevailing Quota Premium (PQP) is the moving average of the QP over the last 3 months. Owners pay the PQP to renew a COE, and it can be calculated from this dataset.


### 2.2 Annual Motor Vehicle Population

The `AnnualMotorVehiclePopulationbyVehicleType.csv` dataset shows how many vehicles are on the road for each vehicle type and for the respective years. There are **412 rows** where each row represents the population of one vehicle type for that year, from 2005 to 2024.

| Field | Type | Description | Values / example | Role |
|---|---|---|---|---|
| `year` | Integer | The year of the count (check with LTA whether this is the year-end figure) | `2005` to `2024` | Time |
| `category` | Text | Broad vehicle group (6 values, see [2.2](#22-annual-vehicle-population)) | `Cars and Station-wagons`, `Buses` | Segment |
| `type` | Text | Vehicle type within the group (22 values) | `Private cars`, `Light Goods Vehicles (LGVs)` | Sub-segment |
| `number` | Integer | Number of registered vehicles | 274 to 540,063 | **Stock** (outcom

##### Vehicle Groups and Types
| `category` | `types` |
| --- | --- |
| Cars and Station-wagons | Private cars, Company cars, Tuition cars, Off peak cars, Rental cars *(2005–2012)*, Private Hire (Self-Drive) cars *(2013–2024)*, Private Hire (Chauffeur) cars *(2013–2024)* |
| Taxis | Taxis |
| Motorcycles and Scooters | Motorcycles and Scooters |
| Goods and Other Vehicles | Goods-cum-passenger vehicles (GPVs), Light Goods Vehicles (LGVs), Heavy Goods Vehicles (HGVs), Very Heavy Goods Vehicles (VHGVs) |
| Buses | Omnibuses, School buses (CB), Private buses, Private hire buses, Excursion buses |
| Tax Exempted Vehicles | Cars and Station-wagons, Motorcycles and scooters, Buses, Goods and Other Vehicles |
##### Understanding relationships between fields

- `category` → `type` is a hierarchy. Adding up the types gives the total for each group, and adding up the groups gives the total vehicle population.
- **Changes in how types are recorded:** *Rental cars* stops after 2012. From 2013 it is replaced by two *Private Hire* types. Treat 2012 to 2013 as a break in the series, not a real change in the number of vehicles.
- *Tax Exempted Vehicles* are grouped by exemption status, not by vehicle kind. Keep them separate from the matching vehicle groups so no vehicle is counted twice.


### 2.3 Interaction between Datasets

```text
COE bidding (flow, twice a month)             Vehicle population (stock, yearly)

bids_success  ── new vehicles registered ──►  number (this year)
                                                =  number (last year)
quota  ◄── set mainly from ────────────────     +  new registrations
           deregistrations                      −  deregistrations
```

**Flow and stock:** COEs allocated during a year add vehicles to the population. Vehicles that are deregistered remove them. So the year-on-year change in `number` is roughly new registrations minus deregistrations.
- **Checking the policy against what happened:** the bidding data shows the supply the policy allowed. The population data shows what that supply produced. Comparing them shows whether the growth limit held in practice.
- **Renewals appear in only one dataset:** when an owner renews a COE (by paying the PQP), the vehicle stays in the population, but no bid appears in the bidding data. A gap between the two datasets may therefore partly reflect renewals.
- **Matching segments is approximate.** The two datasets are grouped differently:

| COE category | Approximate population `type`s |
|---|---|
| A, B (and E) | Private, company, tuition, off-peak and private hire cars |
| C (and E) | Goods and Other Vehicles, Buses |
| D | Motorcycles and Scooters |
| Unclear | Taxis and Tax Exempted Vehicles. These may get COEs through other routes, not the open bidding. Check with LTA before matching them to a category. |

- **Different time grains:** the bidding data must be added up to a yearly total before it can be compared with the population. The two datasets only overlap from **2010 to 2024**.
              
---

## 3. Stakeholders

### 3.1 Policymaker

**Goals:** control growth in the vehicle population, allocate limited road space efficiently, and weigh affordability and fairness against congestion and emissions.

**Questions these datasets can help frame:**
- **Supply lever:** how strongly do premiums respond to changes in quota?
- **Stability:** is COE supply predictable, or do the 10-year replacement cycles cause price shocks?
- **Growth target:** has the vehicle population actually stayed flat since the allowed growth for cars and motorcycles was cut to 0%?
- **Mix of vehicles:** how has the population shifted between private cars, company cars, private hire cars and taxis? Does the shift line up with changes in premiums?
- **Category design:** do the category definitions separate the mass market (A) from premium cars (B) as intended? Is Category E being used as a way around those limits?
- **Economic access:** are COEs affordable for businesses (C) and for delivery and gig workers who rely on motorcycles (D)? Are the matching vehicle populations growing or shrinking?
- **Revenue:** how much does the bidding system raise (`bids_success × premium`)?
- **Policy impact:** how did premiums, bidding behaviour and the vehicle population change around major policy changes?

**Limitations:**
- The bidding data is aggregated, so it can't show who won the bids (individuals or dealers).
- It only records the clearing price, not the full spread of bids.
- The population data is yearly, so it hides changes within a year.
- The two datasets group vehicles differently, so they can only be matched approximately.
- They show associations, not proof of cause and effect.

### 3.2 Potential Car Buyer

**Goals:** decide when to buy, what to buy, and how much to budget, over the full cost of owning a car.

**Questions these datasets can help frame:**
- **Timing:** are premiums noticeably different by season or between the first and second exercise?
- **Supply outlook:** quotas are announced ahead of each quota period (February to April, May to July, August to October, November to January). How does a change in announced supply relate to the price?
- **Replacement waves:** a large number of cars registered about 10 years ago could mean more COEs becoming available soon. Does past car population growth help anticipate future supply?
- **Choice of car:** how much can be saved by choosing a car that qualifies for Category A instead of B, including electric cars?
- **Other ways to own a car:** how common are off-peak cars, and is there a trend towards using private hire or rental cars instead of owning?
- **Risk:** how much can the premium move between exercises, and how big a buffer should a buyer budget?
- **Renewal:** at the end of 10 years, should an owner renew at the PQP or replace the car?

**Limitations:**
- Most buyers get a COE through a dealer package, so what they actually pay may not follow the QP exactly.
- The premium is only one part of a car's cost. Other costs (the car's own price, taxes and fees) are not in these datasets.
- The population data counts vehicles, not new registrations or deregistrations. It can only hint at future supply, not predict it exactly.

---

# 4. Data Exploration

The profiling, cleaning and exploration steps are in [`code/data-cleaning-and-exploration.ipynb`](code/data-cleaning-and-exploration.ipynb). It reads both files from `data/`, charts the distribution of each numeric field, and saves cleaned copies as new `_clean.csv` files. The original files are not changed.

## 4.1 What Was Checked

For each dataset, the checks cover the following:
- **Formatting:** whether there are missing values or nulls, empty strings, leading or trailing spaces, and numbers stored as text
- **Unit:** whether each row is unique based on understanding of the data, so this can identify any duplicated records
- **Time coverage:** since both datasets are time series, to review the date range and identify any gaps in it
- **Data profile:** for numeric fields, understand the data profile using min, max, percentiles, skew, zeros and negatives
- **Reasonableness check:** `bids_success` should never be more than `quota` or `bids_received`
- **Suspicious values:** unusually large jumps between one period and the next as these may indicate data quality issue

## 4.2 Key Facts About the Data

**COE Bidding Results**

| Check | COE Bidding Results | Annual Motor Vehicle Population |
| --- | --- | --- |
| Unit | One row per `month` × `bidding_no` × `vehicle_class`, with no duplicates. Every exercise has all 5 categories. | One row per `year` × `category` × `type`, with no duplicates. |
| Formatting | No missing values/nulls, blanks or stray spaces. **The only problem is the thousands separators in `bids_success` and `bids_received`.** | No missing values/nulls, blanks or stray spaces. All values in `number` are whole numbers.<br><br>*Motorcycles and Scooters* (a category) and *Motorcycles and scooters* (a type under Tax Exempted Vehicles) differ only in capitalisation. Don't join on `type` alone. |
| Time coverage | 198 months, not 201: **April to June 2020 are missing** because bidding was suspended during the COVID-19 circuit breaker. | 2005 to 2024 with no gaps. There are 20 types per year up to 2012 and 21 from 2013, because of the change from *Rental cars* to the two *Private Hire* types. |
| Reasonableness check | `bids_success` never exceeds `quota` or `bids_received`. About 81% of exercises leave a few COEs unallocated (median 4). |  |
| Distributions | Quota and bids are right-skewed. Premium has two groups: Category D (motorcycles, median about $6k) sits far below the car categories (median about $49k to $70k). | Heavily right-skewed (median about 9.5k, max 540k), because *Private cars* is much larger than every other type. |
| Round numbers | About 19% of premiums end in `000` or `001`. This is expected, because bidders choose round numbers. |  |



## 4.3 Issues Found

| Severity | Dataset | Issue |
| --- | --- | --- |
| 🔴 High | COE | **2010-01, exercise 2, Category D:** `premium` = 20,090. It matches Category -C's premium in the same exercise. Category D was 889 the exercise before and 852 the exercise after. |
| 🔴 High | COE | **2010-02, exercise 1, Category B:** `quota` = 1,154. It matches Category A's quota in the same exercise. Category B's quota is about 690 in the exercises around it. The wrong value makes this the only undersubscribed exercise in the data and the one with the largest unallocated gap (464). |
| 🟠 Medium | COE | No bidding exercises from April to June 2020. |
| 🟡 Low | Population | In 2013, *Goods-cum-passenger vehicles* dropped 23.6% and tax-exempted *Motorcycles and scooters* dropped 23.9%. These happen in the same year the car types were redefined, so they are likely reclassifications, not real changes in the number of vehicles. |

Both high-severity values look like they were copied from a neighbouring row. They should be checked against LTA's published results.

## 4.4 Cleaning Done

| Step | How |
|---|---|
| Removed thousands separators | `pd.read_csv(..., thousands=',')` |
| Parsed `month` as a date | `pd.to_datetime(coe['month'])` |
| Allowed missing values in the COE count and price columns | `quota`, `bids_success`, `bids_received` and `premium` converted to nullable integers (`Int64`) |
| Set the 2 suspicious values to missing | Category D `premium` (2010-01, exercise 2) and Category B `quota` (2010-02, exercise 1) set to `pd.NA`. They were not replaced with an estimate. |

No rows were removed. The cleaned data is saved as:

| File | Rows | What changed from the original |
|---|---|---|
| `data/COEBiddingResultsPrices_clean.csv` | 1,980 | No thousands separators. The 2 suspicious values are empty cells. `month` stays as `YYYY-MM` text. |
| `data/AnnualMotorVehiclePopulationbyVehicleType_clean.csv` | 412 | No value changes. Saved so both datasets can be loaded from the cleaned set. |

To load the cleaned COE data with the empty cells kept as missing integers:

```python
coe = pd.read_csv('data/COEBiddingResultsPrices_clean.csv',
                  dtype={'quota': 'Int64', 'bids_success': 'Int64', 'bids_received': 'Int64', 'premium': 'Int64'})
coe['date'] = pd.to_datetime(coe['month'])
```

## 4.5 Insights from Exploration

**COE bidding, 2010 to 2026**
- **Supply and price move in opposite directions over a roughly 10-year cycle.** The yearly quota rose from about 41,000 (2013) to 112,000 (2017), then fell back to about 43,000 (2022). Premiums were lowest around 2018 to 2019 (about $30,000 to $40,000 for Categories A and B) and have since risen to records above $130,000.
- **COE revenue reached a record of about $6.5 billion in 2025**, driven by the high premiums.

**Vehicle population, 2005 to 2024**
- **The total grew fast, then stopped.** It rose from 755,000 (2005) to about 970,000 (2012), stayed almost flat until 2020 as the allowed growth rate was cut, and passed 1 million in 2024.
- **By category:** *Tax Exempted Vehicles* grew fastest (+90%) and cars grew 50%. Motorcycles (+6%) and goods vehicles (+12%) barely changed. **Taxis fell by more than half** from their 2014 peak (28,700 to 13,100).
- **A shift towards ride-hailing.** From 2014 to 2017, *Private Hire (Chauffeur)* cars rose by about 46,000 while private cars fell by about 38,000, even though the COE quota was high. By 2024, private hire cars (both types) made up about 14% of all cars. The data shows the timing, not the cause: some private hire cars are converted private cars.
- **Private cars** peaked at 540,063 in 2013 and have stayed between about 515,000 and 532,000 since 2019.
- **Off-peak cars fell by 85%** from their 2010 peak (50,040 to 7,692).
- **The 2013 reclassification is continuous:** *Rental cars* (14,862 in 2012) became *Private Hire (Self-Drive)* (15,782 in 2013), so the two can be joined into one series.
- **Company cars jumped 26% in 2020** (24,610 to 30,966), much more than in any other year. This may be another reclassification and is worth checking with LTA.

---

# 5. Analysis

The analysis is in [`code/analysis.ipynb`](code/analysis.ipynb). It focuses on the **potential car buyer** (Section 3.2), with one finding for the policymaker. Each question is answered first with a simple method that is easy to explain (grouped comparisons and percentiles), then with regression or a statistical test where it adds accuracy. All results describe the past and are ranges to plan with, not forecasts.

**Preparation:** the two suspicious values are filled from neighbouring exercises for time-series work only, and flagged. Supply and price are compared by LTA quota period. The renewal price (PQP) is estimated as the average premium of the latest 6 exercises. Changes that would span the April to June 2020 gap are left out.

| # | Question | Method | Key finding |
|---|---|---|---|
| 2 | How much can premiums move before I buy? | Change from each exercise to the same exercise 1, 3, 6 and 12 months later: median, 90th and 95th percentiles | Premiums rose over 3 to 6 months in about 60% of past cases (65% to 75% since February 2018). The typical change is small (Category B median +5% over 6 months), but the 90th percentile is large: **+22% within 3 months and +37% within 6 months** for B. |
| 3 | Does the announced quota predict the premium? | Premium change grouped by quota change; regression of log changes with HAC standard errors and a robust check | **Quota cuts of more than 10% were followed by higher premiums in 78% to 91% of periods. Quota increases did not reliably lower them**, especially for A and E. A 10% quota cut goes with about **+2.5% to +2.8%** for A, B and E and **+8.5%** for motorcycles (D). Quota explains only 7% to 21% of car premium changes. |
| 4 | When to bid? | Exercise 1 vs 2 (Wilcoxon test); month of the quota period and month of the year vs a 12-month trend (F-tests) | **No reliable timing effect for cars.** Motorcycles (D) were about 7% below trend in December and January, but this may not repeat. |
| 5 | How much does Category A save over B? | Gap per exercise, compared across the 97kW rule (February 2014) and the electric car limit (May 2022); regression | A was cheaper in 95% of exercises, by a median of **$9,000 (16%)**. The gap follows the premium cycle, not the rule changes, and is **almost zero now ($1,110 in September 2026)**. |
| 6 | Renew or replace at year 10? | Estimated PQP vs the current premium, split by whether premiums were rising or falling | When premiums had been rising, renewing was cheaper in about **80%** of cases (by about 4%). When falling, a new COE was cheaper in about 75%. The 5-year and 10-year renewals cost the same per year. |
| 7 | Alternatives to owning a car | Vehicle types since 2013 vs the Category A premium | Rental and car-sharing cars nearly **doubled (+97%)** from 2013 to 2024, while private cars stayed flat (−3%). The COE alone now costs about **$1,100 a month** for a car. |

**A note on method:** grouping by calendar quarter instead of LTA's quota periods overstated the quota effect by about 40% for Category A and 70% for Category E, and roughly doubled it for B. Using the right period mattered.

---

# 6. Recommendations

**For a potential buyer of a Category A or B car:**

| # | Recommendation | Confidence |
|---|---|---|
| 1 | **Budget a buffer above today's premium:** about **20% to 25%** if bidding within 3 months, **30% to 40%** within 6 months. This covered 9 out of 10 past cases. | High |
| 2 | **Don't wait for prices to fall without a reason.** Premiums have usually been higher a few months later. | High |
| 3 | **Watch the quota announcement.** If the next period's quota is cut by more than 10%, bid before it takes effect. A quota increase is not a reliable reason to wait. | Medium (only 9 to 19 large cuts per category) |
| 4 | **Don't try to time the exercise or month** for a car. | High |
| 5 | **Check the current A vs B gap** before choosing a car. The long-run saving is about $9,000, but it is almost zero today. | High |
| 6 | **At year 10, renew if premiums have been rising; consider replacing if they have been falling.** The COE difference is usually only a few percent, so the car's condition matters more. | High for the direction; the effect is small |
| 7 | **Compare owning with renting or car-sharing** if you drive occasionally. The COE alone costs about $1,100 a month. | The cost is exact; rental prices are not in these datasets |

**For the policymaker:** cutting the quota raises premiums reliably, but raising it lowers them only a little. A 10% change in supply goes with about a 2.5% to 3% change in car premiums and explains a small share of their movement, so supply changes alone will not bring car premiums down much.

**Limitations:**
- Results describe January 2010 to September 2026 (COE) and 2005 to 2024 (population). They are not forecasts.
- Only the COE premium is covered, not the car's price, taxes, dealer packages or running costs.
- The PQP is estimated from the data and may differ slightly from LTA's published figure.
- The findings are associations, not proof of cause and effect.

---

# 7. Web App Data

A **COE planner web app** for car buyers is planned, with a Next.js front end on Vercel. [`code/web-app-data.ipynb`](code/web-app-data.ipynb) runs the analysis and saves the numbers the app needs to `outputs/coe_planner_data.json`:

| App feature | From |
|---|---|
| "How much should I budget?" with a safety slider (75%, 90%, 95%) | Section 2 |
| Quota signal when the next period's quota is announced | Section 3 |
| Category A vs B saving | Section 5 |
| Renew or replace at year 10 | Section 6 |

Re-run the notebook when new COE results are published, so the app and the analysis use the same numbers.


