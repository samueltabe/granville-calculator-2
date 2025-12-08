# Granville Energy Mini-Grid ROI & Bankability Calculator - Calculation Explanation

This document explains the complete calculation methodology used in the calculator.

---

## Overview

The calculator performs a 20-year cashflow analysis for a PV + battery mini-grid project. It calculates:
1. **CAPEX** (Capital Expenditure) - total project cost
2. **Revenue** - energy sales income
3. **Operating Costs** - O&M, insurance, grid purchases
4. **Financing** - debt service, equity returns, DSRA management
5. **Key Metrics** - DSCR, IRR, Payback periods

---

## Step 1: Input Collection & Tariff Calculation

### System Size
- **PV Size** (kWp): Solar panel capacity
- **PCS Size** (kW): Power conversion system capacity
- **Battery Size** (kWh): Battery storage capacity

### Tariff Determination

The calculator handles three tariff modes:

1. **Fixed Prepaid**: Uses the fixed tariff value directly
   ```
   customerTariffBase = fixedTariff
   ```

2. **TOU (Time-of-Use)**: Weighted average based on usage shares
   ```
   normalizedOffPeak = offPeakShare / (offPeak + standard + peak)
   normalizedStandard = standardShare / (offPeak + standard + peak)
   normalizedPeak = peakShare / (offPeak + standard + peak)
   
   customerTariffBase = (offPeakTariff × normalizedOffPeak) + 
                        (standardTariff × normalizedStandard) + 
                        (peakTariff × normalizedPeak)
   ```

3. **Postpaid/Levy**: Uses the blended tariff field as-is

---

## Step 2: CAPEX Calculation

### 2.1 Hardware CAPEX
```
hardwareCapex = pvCostTotal + pcsBatCostTotal
```

### 2.2 Balance of System (BOS) & Installation
```
bosCapex = hardwareCapex × (bosPct / 100)
```
Example: If BOS is 60% and hardware is R1,000,000, then BOS = R600,000

### 2.3 Development Costs
```
devCost = hardwareCapex × (devPct / 100)
```

### 2.4 Base CAPEX (Before Interest During Construction)
```
baseCapexBeforeIDC = hardwareCapex + bosCapex + devCost
```

### 2.5 Interest During Construction (IDC)

IDC is calculated only on the **debt portion** of the project during the construction period:

```
debtCapexApprox = baseCapexBeforeIDC × (debtShare / 100)
idc = debtCapexApprox × interestRate × (constructionMonths / 12)
```

**Example:**
- Base CAPEX: R10,000,000
- Debt share: 80% → Debt portion: R8,000,000
- Interest rate: 12% per year
- Construction period: 6 months

```
IDC = R8,000,000 × 0.12 × (6/12) = R480,000
```

### 2.6 Total CAPEX
```
totalCapex = baseCapexBeforeIDC + idc
```

---

## Step 3: Financing Structure

### 3.1 Debt & Equity Split
```
debtAmount = totalCapex × (debtShare / 100)
equityAmount = totalCapex - debtAmount
```

### 3.2 Annual Debt Service (Annuity Formula)

The calculator uses the standard annuity formula to calculate **level annual debt service**:

```
annualDebtService = debtAmount × [r × (1+r)ⁿ] / [(1+r)ⁿ - 1]

Where:
  r = interestRate (as decimal, e.g., 0.12 for 12%)
  n = loanTenor (years)
```

**Formula Explanation:**
- This is the loan amortization formula that calculates equal annual payments
- Each payment covers both interest and principal repayment
- The total payment amount remains constant each year

**Example:**
- Debt: R8,000,000
- Interest: 12% per year
- Tenor: 10 years

```
r = 0.12
n = 10
numerator = 0.12 × (1.12)¹⁰ = 0.12 × 3.106 = 0.373
denominator = (1.12)¹⁰ - 1 = 3.106 - 1 = 2.106
annualDebtService = R8,000,000 × (0.373 / 2.106) = R1,417,000
```

### 3.3 DSRA (Debt Service Reserve Account)

DSRA is a reserve fund to ensure debt service can be paid even if revenue is temporarily low:

```
dsraTarget = annualDebtService × (dsraMonths / 12)
```

**Example:**
- Annual debt service: R1,417,000
- DSRA months: 3

```
dsraTarget = R1,417,000 × (3/12) = R354,250
```

This reserve must be **funded upfront** as part of equity (Year 0).

---

## Step 4: Year 0 Initial Investment

At project start (Year 0), the equity investor must provide:

```
initialEquityInvestment = equityAmount + dsraTarget

Where:
  equityAmount = totalCapex - debtAmount
  dsraTarget = reserve fund amount
```

This is recorded as a **negative cashflow** (outflow) in Year 0.

---

## Step 5: Annual Cashflow Calculation (Years 1-20)

For each year from 1 to project horizon:

### 5.1 Energy Generation with Degradation

Battery degradation reduces available energy each year:

```
energyY = baseEnergyYear1 × (1 - batteryDegradation) ^ (year - 1)

Where:
  baseEnergyYear1 = pvSize × yieldPerKwp × usableFraction
```

**Example:**
- PV Size: 500 kWp
- Yield: 1,750 kWh/kWp/year
- Usable fraction: 0.9 (90%)
- Battery degradation: 2% per year

```
Year 1: energy = 500 × 1,750 × 0.9 = 787,500 kWh
Year 2: energy = 787,500 × (1 - 0.02)¹ = 771,750 kWh
Year 3: energy = 787,500 × (1 - 0.02)² = 756,315 kWh
```

### 5.2 Tariff Escalation

Tariff increases annually:

```
tariffY = baseTariffYear1 × (1 + tariffEscalation) ^ (year - 1)
```

**Example:**
- Base tariff: R2.80/kWh
- Escalation: 7% per year

```
Year 1: R2.80 × (1.07)⁰ = R2.80
Year 2: R2.80 × (1.07)¹ = R3.00
Year 3: R2.80 × (1.07)² = R3.21
```

### 5.3 Revenue

```
revenueY = energyY × tariffY
```

### 5.4 Grid Energy Costs

If the project purchases grid energy to supplement:

```
effectiveGridShare = gridShare% × gridAvailabilityFactor
gridEnergyY = energyY × effectiveGridShare
gridCostY = gridEnergyY × gridTariff
```

**Example:**
- Energy sold: 787,500 kWh
- Grid share: 10% of energy sold
- Grid availability: 90%
- Grid tariff: R1.80/kWh

```
effectiveGridShare = 0.10 × 0.90 = 0.09 (9%)
gridEnergy = 787,500 × 0.09 = 70,875 kWh
gridCost = 70,875 × R1.80 = R127,575
```

### 5.5 Operating & Maintenance (O&M) Costs

O&M escalates annually:

```
omY = totalCapex × (omPct / 100) × (1 + omEscalation) ^ (year - 1)
```

**Example:**
- Total CAPEX: R10,000,000
- O&M: 1.5% of CAPEX
- O&M escalation: 6% per year

```
Year 1: R10,000,000 × 0.015 × (1.06)⁰ = R150,000
Year 2: R10,000,000 × 0.015 × (1.06)¹ = R159,000
```

### 5.6 Insurance Costs

Insurance also escalates:

```
insuranceY = totalCapex × (insurancePct / 100) × (1 + omEscalation) ^ (year - 1)
```

**Note:** Insurance uses the same escalation rate as O&M.

### 5.7 Battery Replacement

If battery replacement occurs in a specific year:

```
battReplaceY = battReplaceCost × (1 + battReplaceLabour%)
```

This is only applied in the specified replacement year (e.g., Year 12).

### 5.8 Net Operating Cashflow (Before Financing)

```
netOperatingY = revenueY - gridCostY - omY - insuranceY - battReplaceY
```

This is the cashflow **before** debt service, DSRA, and distributions.

---

## Step 6: DSRA Management

The DSRA (Debt Service Reserve Account) must be maintained at target level while the loan is outstanding:

### 6.1 DSRA Target by Year

```
if (year <= loanTenor) {
  targetDsraY = dsraTarget
} else {
  targetDsraY = 0  // No DSRA needed after loan repaid
}
```

### 6.2 DSRA Top-Up or Release

```
prevDsra = DSRA balance from previous year

if (targetDsraY > prevDsra) {
  // Need to top up
  topUpY = targetDsraY - prevDsra
  dsraEndY = targetDsraY
} else if (targetDsraY < prevDsra) {
  // Can release excess (usually when loan ends)
  release = prevDsra - targetDsraY
  // Release is added back to net operating cashflow
  netOperatingY += release
  dsraEndY = targetDsraY
}
```

**Important:** DSRA top-ups are **deducted** from available cash before debt service. DSRA releases are **added back** to cashflow when the loan is repaid.

---

## Step 7: Debt Service & DSCR

### 7.1 Cash Available for Debt Service

```
cashBeforeDebt = netOperatingY - dsraTopUpY
```

### 7.2 Annual Debt Service

```
if (year <= loanTenor) {
  debtServiceY = annualDebtService  // Constant level payment
} else {
  debtServiceY = 0  // Loan fully repaid
}
```

### 7.3 Debt Service Coverage Ratio (DSCR)

DSCR is a key bankability metric:

```
DSCRY = cashBeforeDebt / debtServiceY
```

**Interpretation:**
- **DSCR > 1.3**: Very strong coverage
- **DSCR 1.2 - 1.3**: Bankable for most lenders
- **DSCR 1.1 - 1.2**: Borderline, may need restructuring
- **DSCR 1.0 - 1.1**: High risk
- **DSCR < 1.0**: Cannot service debt

The calculator tracks:
- **Minimum DSCR**: Lowest DSCR across all years (critical metric)
- **Average DSCR**: Average across loan tenure

---

## Step 8: Equity Cashflow Distribution

### 8.1 Cash After Debt Service

```
cashAfterDebt = cashBeforeDebt - debtServiceY
```

### 8.2 Minimum Cash Covenant

Banks often require a minimum cash buffer:

```
if (cashAfterDebt > minCash) {
  distributable = cashAfterDebt - minCash
} else {
  distributable = 0  // All cash must remain in reserve
}
```

### 8.3 Revenue Share to Partner (Landlord/Body Corporate)

Revenue share is calculated on **distributable cash** (after debt and min cash):

```
if (year >= revShareStartYear && distributable > 0) {
  partnerShareY = distributable × (revSharePct / 100)
} else {
  partnerShareY = 0
}
```

**Example:**
- Distributable cash: R500,000
- Revenue share: 10%
- Start year: Year 1

```
partnerShare = R500,000 × 0.10 = R50,000
```

### 8.4 Equity Distribution to Granville

```
equityDistributionY = distributable - partnerShareY
```

This is the cash that flows to the equity investor (Granville) in each year.

---

## Step 9: Key Performance Metrics

### 9.1 Project Payback Period (Pre-Finance)

Calculates when cumulative **operating cashflows** (before financing) recover initial CAPEX:

```
cumulativeCashflow = -totalCapex

For each year 1 to N:
  cumulativeCashflow += netOperatingY
  if (cumulativeCashflow >= 0 && paybackYear == null) {
    paybackYear = currentYear
  }
```

This shows how quickly the project generates cash to cover its initial cost.

### 9.2 Equity Payback Period

Calculates when cumulative **equity cashflows** (after financing, revenue share) turn positive:

```
cumulativeEquity = -(equityAmount + initialDsraFunding)

For each year 1 to N:
  cumulativeEquity += equityDistributionY
  if (cumulativeEquity >= 0 && equityPaybackYear == null) {
    equityPaybackYear = currentYear
  }
```

This shows when the equity investor recovers their initial investment.

### 9.3 Internal Rate of Return (IRR)

IRR is the discount rate that makes the Net Present Value (NPV) of equity cashflows equal to zero.

**IRR Calculation (Newton-Raphson Method):**

The calculator uses an iterative numerical method:

1. Start with an initial guess (default: 12%)
2. Calculate NPV using the guess:
   ```
   NPV = Σ [cashflow[t] / (1 + rate)^t]
   ```
3. Calculate derivative of NPV:
   ```
   dNPV = Σ [-t × cashflow[t] / ((1 + rate)^(t+1))]
   ```
4. Update guess:
   ```
   newRate = oldRate - (NPV / dNPV)
   ```
5. Repeat until NPV ≈ 0 (or maximum iterations reached)

**Formula:**
```
0 = CF₀ + CF₁/(1+IRR)¹ + CF₂/(1+IRR)² + ... + CFₙ/(1+IRR)ⁿ

Where:
  CF₀ = Initial equity investment (negative)
  CF₁...CFₙ = Annual equity distributions (positive)
```

**Example:**
- Year 0: -R2,354,250 (equity + DSRA)
- Year 1: +R300,000
- Year 2: +R350,000
- ...
- IRR ≈ 18.5% (solution where NPV = 0)

**Interpretation:**
- **IRR > 20%**: Excellent return
- **IRR 18-20%**: Good infrastructure return
- **IRR 15-18%**: Acceptable
- **IRR < 15%**: Low for infrastructure risk

---

## Step 10: Summary Outputs

The calculator displays:

1. **System Size**: PV, PCS, Battery capacities
2. **Total CAPEX**: With breakdown (hardware, BOS, dev, IDC)
3. **Base-Year Tariff & Energy**: Year 1 revenue potential
4. **Year 1 Net Operating Cashflow**: First year operational performance
5. **Debt & Equity Split**: Financing structure
6. **Annual Debt Service**: Loan payment amount
7. **Payback Periods**: Project and equity payback
8. **DSCR**: Minimum and average debt service coverage
9. **Equity IRR**: Return on equity investment
10. **Revenue Share**: Total partner payments over project life

---

## Calculation Flow Summary

```
1. INPUT COLLECTION
   ↓
2. TARIFF CALCULATION (fixed/TOU/postpaid)
   ↓
3. CAPEX BUILDUP
   → Hardware + BOS + Development + IDC = Total CAPEX
   ↓
4. FINANCING STRUCTURE
   → Debt split → Annual debt service → DSRA target
   ↓
5. YEAR 0: Initial equity investment (equity + DSRA)
   ↓
6. FOR EACH YEAR (1-20):
   → Energy (with degradation)
   → Revenue (energy × tariff with escalation)
   → Costs (grid, O&M, insurance, battery replacement)
   → Net Operating Cashflow
   → DSRA management
   → Debt service & DSCR
   → Distributable cash
   → Revenue share to partner
   → Equity distribution to Granville
   ↓
7. METRICS CALCULATION
   → Project payback
   → Equity payback
   → Equity IRR (Newton-Raphson)
   → Min/Avg DSCR
   ↓
8. OUTPUT SUMMARY
```

---

## Key Assumptions & Limitations

1. **Level Debt Service**: Assumes constant annual payments (annuity structure)
2. **Annual Resolution**: Calculations done annually, not monthly
3. **Constant Ratios**: Grid share, O&M, insurance percentages remain constant
4. **Battery Degradation**: Applied to energy output, not battery capacity separately
5. **DSRA Management**: Simplified top-up/release logic
6. **No Tax**: Tax calculations not included
7. **No Inflation Adjustment**: Escalations are separate from inflation

---

## Example Full Calculation

Let's trace through a simplified example:

### Inputs:
- PV: 500 kWp
- Hardware CAPEX: R5,359,018
- BOS: 60% of hardware
- Debt: 80% of total CAPEX
- Interest: 12% per year, 10-year tenor
- Tariff: R2.80/kWh, 7% escalation
- Yield: 1,750 kWh/kWp/year
- Usable fraction: 90%

### Calculations:

**CAPEX:**
```
Hardware: R5,359,018
BOS (60%): R5,359,018 × 0.60 = R3,215,411
Dev (2%): R5,359,018 × 0.02 = R107,180
Base CAPEX: R8,681,609
Debt portion: R8,681,609 × 0.80 = R6,945,287
IDC (6 months @ 12%): R6,945,287 × 0.12 × 0.5 = R416,717
Total CAPEX: R9,098,326
```

**Financing:**
```
Debt: R9,098,326 × 0.80 = R7,278,661
Equity: R9,098,326 × 0.20 = R1,819,665
Annual debt service (annuity): R1,288,423
DSRA (3 months): R1,288,423 × 0.25 = R322,106
Initial equity investment: R1,819,665 + R322,106 = R2,141,771
```

**Year 1 Cashflow:**
```
Energy: 500 × 1,750 × 0.90 = 787,500 kWh
Revenue: 787,500 × R2.80 = R2,205,000
O&M (1.5% CAPEX): R9,098,326 × 0.015 = R136,475
Insurance (0.45% CAPEX): R9,098,326 × 0.0045 = R40,942
Net Operating: R2,205,000 - R136,475 - R40,942 = R2,027,583
Cash before debt: R2,027,583 (assuming no DSRA top-up needed)
Debt service: R1,288,423
DSCR: R2,027,583 / R1,288,423 = 1.57×
Cash after debt: R739,160
Distributable (after R200k min cash): R539,160
Revenue share (10%): R53,916
Equity distribution: R485,244
```

This gives you a complete picture of how each metric is calculated!

