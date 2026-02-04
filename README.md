# Cooperative Pricing Model 

This notebook implements a **data-driven pricing recommendation system** for car-sharing cooperatives.  
It predicts (1) **monthly demand** (hours) at user level, (2) **monthly supply/capacity** (hours) at vehicle level, and then computes a **cost-plus recommended price** that covers costs + margin. It also supports **mixed-category cooperatives** and can **simulate discount scenarios**.

>  **Data privacy note**  
> The original datasets used in this project are private. This repository/notebook therefore does **not** include:
> - the raw trip-level database,
> - the cooperative metadata tables,
> - the leasing / cost tables,
> - the final merged dataset,
> - real paths or real cost values.  
>
> To run this notebook end-to-end, you must provide **your own merged CSV** with the required schema (see below) and fill the `COSTS` dictionary.

---

## 1) What this notebook does

### A) Data preprocessing
- Filters to **only completed rides**: `status == "FINISHED"`.
- Cleans vehicle category names and rebuilds a **pure** `cost_cat`.
- Drops unused columns and removes rows missing critical values.

### B) Model training (two supervised tasks)
1. **Model 1 — Demand model (User → Hours)**  
   Predicts **total monthly usage hours per user** using **RandomForestRegressor**.
2. **Model 2 — Supply model (Car → Hours)**  
   Predicts **total monthly available/usage hours per vehicle** using **XGBRegressor (XGBoost)**.

Both models use **GroupShuffleSplit**:
- User model groups by `id_hh` (prevents leakage across the same user).
- Car model groups by `kenteken` (prevents leakage across the same vehicle).

Metrics reported:
- **MAE**
- **R²**
- **RMSE** (computed as `sqrt(mean_squared_error)`)

### C) Pricing recommendation
Given:
- predicted demand: `H` (hours), `K` (km)
- predicted supply: `hours_per_car`, then cars needed
- costs (fixed + variable)
- membership fees
- margin

It computes a **recommended base price**:
- `price_km = price_hour / 10` (business rule)

### D) Mixed category recommendation
Supports cooperatives where users want different vehicle categories:
- Example: `{"B1": 0.8, "B2": 0.2}`
- Predicts demand and cars per category
- Outputs a single global price (hour/km rule preserved)

### E) Discount simulation (optional)
Discounts are treated as **hardcoded rules**.
The notebook:
- randomly selects **exactly 2 discount rules**
- applies them to the **base price**
- computes an **effective price** after discounts
- enforces a floor:
  - effective price must not be lower than `(base price) * (1 - margin_floor)`

Outputs:
- `recommended_price_*` = base price **before** discounts  
- `effective_price_*` = implied price **after** discounts

---
