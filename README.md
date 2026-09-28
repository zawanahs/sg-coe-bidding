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

        POLICY                             MARKET
   (set by LTA)                   (buyers, mostly via dealers)
         │                                   │
       quota  ─────────►  AUCTION  ◄───── bids_received
                             │
               ┌─────────────┴─────────────┐
           premium                    bids_success
     (clearing price)               (≤ quota; any leftover
                                     rolls into later exercises)


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
| `category` | `type`s |
|---|---|
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

   COE bidding (flow, twice a month)             Vehicle population (stock, yearly)

   bids_success  ── new vehicles registered ──►  number (this year)
                                                   =  number (last year)
   quota  ◄── set mainly from ────────────────     +  new registrations
              deregistrations                      −  deregistrations


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

## 3. Stakeholder Perspectives

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
- **Supply outlook:** quotas are announced for each quarter ahead of time. How does a change in announced supply relate to the price?
- **Replacement waves:** a large number of cars registered about 10 years ago could mean more COEs becoming available soon. Does past car population growth help anticipate future supply?
- **Choice of car:** how much can be saved by choosing a car that qualifies for Category A instead of B, including electric cars?
- **Other ways to own a car:** how common are off-peak cars, and is there a trend towards using private hire or rental cars instead of owning?
- **Risk:** how much can the premium move between exercises, and how big a buffer should a buyer budget?
- **Renewal:** at the end of 10 years, should an owner renew at the PQP or replace the car?

**Limitations:**
- Most buyers get a COE through a dealer package, so what they actually pay may not follow the QP exactly.
- The premium is only one part of a car's cost. Other costs (the car's own price, taxes and fees) are not in these datasets.
- The population data counts vehicles, not new registrations or deregistrations. It can only hint at future supply, not predict it exactly.
