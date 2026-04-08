# Mortgage vs. Rent: NPV Comparison

**Boris Garbuzov, April 2026**

## 1. Motivation

The motivation for this project was my personal need. A realtor tried to
convince me to buy a condo. YouTube finance channels warned against it.
Neither side did the actual math.

You either rent the place you live in, or you own it. If you own it, you can
live there yourself — in which case the avoided rent is your implicit income —
or you can rent it out and live elsewhere. Either way, the calculation is the
same: what does ownership return, net of all costs, versus what does renting
cost, net of the opportunity cost of the down payment.

Both options are negative in absolute terms — housing is a cost. The honest
question is not "is buying good" but "which option loses less money in present
value terms, under your specific forecast of the inputs."

Toronto condo prices have been falling for three years
([data](https://toronto.listing.ca/condo-price-history.htm)). That matters,
but it is not decisive on its own. A negative appreciation rate can be offset
by a low mortgage rate, a low maintenance fee, a long holding horizon, or a
high equivalent rent. There is no shortcut — you have to set all twelve
parameters and compute.

This notebook does that. You set your numbers, it computes six outputs and
plots each one as a function of any parameter you want to vary. If you do not
care about the plots as a sensitivity analysis, a single call gives you the
bottom line.

## 2. The Model

### 2.1 Twelve Input Parameters

- **`r_m_yearly = 0.04`** — annual mortgage rate. Published by lenders online,
  no estimation needed. Around 4% was the prevailing rate at the time of
  writing. Reset it to whatever you are quoted today — probably around 3.5%.

- **`r_d_yearly = 0.0375`** — annual discount rate, and simultaneously the
  opportunity cost of the down payment. At the time of writing, a GIC slightly
  below 3.75% was available in Canada. In the ownership scenario this rate
  discounts future cash flows to present value; in the rental scenario it
  represents what the down payment would earn sitting in a deposit instead of
  going toward a purchase. This rate should sit below the mortgage rate: the
  bank bears the credit risk on the mortgage and earns a spread over the
  risk-free deposit rate. That spread is the gap between `r_m` and `r_d`.
  Economically, the discount rate has two components: the appreciation of the
  consumption basket, and the pure time preference of a consumer who assigns
  some probability to not being alive next year.

- **`r_a_yearly = 0.03`** — annual real estate appreciation rate. The parameter
  requiring the most judgment. Real estate is one component of the consumption
  basket; its appreciation can diverge from the general discount rate depending
  on local supply, demographics, and credit conditions. The Toronto historical
  estimate was widely quoted at around 5% annualized through 2022. The
  following three years were negative. As a working forecast: roughly +1% on
  average for a 5-year horizon, +3% for a 10-year horizon. The notebook sweeps
  from −3% to +9% so you can observe exactly where the decision flips.

- **`downpayment_percentage = 0.20`** — fraction of the purchase price paid
  upfront. The remainder becomes the mortgage principal. The down payment is
  the capital tied up in the property — its foregone interest is the
  opportunity cost that makes renting more attractive than it first appears.

- **`agency_percentage = 0.06`** — realtor commission plus legal fees at sale,
  as a fraction of the future selling price. A fixed transactional drag paid
  once at the end of the horizon. Easy to overlook. On a $500K property it is
  $30K out the door.

- **`capital_gain_tax_percentage = 0.10`** — tax on the nominal price
  appreciation at sale. Set between 0 and 0.20 depending on your residential
  status, jurisdiction, and ownership structure.

- **`T_years = 30`** — mortgage amortization term. Determines the monthly
  payment. We assume the mortgage rate is constant for the full term — no
  modelling of 5-year resets. A simplification, but it keeps the sensitivity
  analysis interpretable.

- **`H_years = 10`** — project horizon: the year you expect to sell.
  Underappreciated parameter. Transactional costs — agency fees, capital gains
  tax — are fixed charges paid at sale regardless of how long you held the
  property. A short horizon means those costs are amortized over fewer years,
  making ownership less attractive. The model makes this visible.

- **`initial_house_price = 400_000`** — purchase price in dollars. The
  notebook sweeps this from $0 to $1M. Caution on interpretation: across that
  range the equivalent rent is held fixed at $1,800/month regardless of price.
  A more realistic range for the Toronto condo market is $300K–$500K.

- **`condo_fee_and_maintenance = 650`** — monthly ownership costs above the
  mortgage payment. In Toronto, a typical breakdown is $600 condo fee and $50
  maintenance reserve.

- **`utilities = 200`** — monthly hydro, gas, internet. This appears
  identically in both scenarios and therefore cancels in the comparison — it
  does not affect `net_purchase_benefit`. It is included so each scenario shows
  its true total monthly outflow.

- **`rent_payment_monthly = 1_800`** — the monthly rent paid for an equivalent
  property, or equivalently, the rent that could be collected by renting it
  out. The most important variable on the rental side.

### 2.2 Six Outputs

- **`Future house price`** — `initial_house_price × (1 + r_a_yearly)^H_years`.
  The projected selling price before any deductions.

- **`Net selling revenue`** — what remains after the sale: future house price
  minus realtor commission and legal fees (`agency_percentage × future price`),
  minus capital gains tax (`capital_gain_tax_percentage × nominal gain`), minus
  the remaining mortgage balance at month `H_years × 12`. This is the terminal
  cash inflow to the owner.

- **`Mortgage NPV`** — present value of the full ownership position. Outflows:
  monthly mortgage payments plus condo fee, maintenance, and utilities,
  discounted at `r_d`. Inflow: `net_selling_revenue` discounted back from month
  `H_years × 12`. These two components are summed to give the ownership NPV.

- **`Rent NPV`** — present value of the rental position. Outflow: monthly rent
  plus utilities, discounted at `r_d`. Inflow: the down payment itself, which
  is not spent and earns `r_d` — it enters as a positive term offsetting the
  rental outflows.

- **`Mortgage payment monthly`** — the fixed annuity payment on the mortgage
  principal, computed from the standard amortization formula.

- **`Net purchase benefit`** — `Mortgage NPV − Rent NPV`. The decision
  variable. Positive means owning wins; negative means renting wins.

## 3. Code and Usage

Implemented as a Python notebook. Dependencies: `numpy`, `pandas`,
`matplotlib`.

Single calculation at default parameters:

```python
calculate()
```

Sensitivity sweep — vary one parameter, hold the rest at defaults:

```python
run(r_a_yearly=np.arange(-0.03, 0.1, 0.01))
```

Pass exactly one parameter as a NumPy array. The `run()` function iterates
over it, calls `calculate()` at each point, and produces a 2×3 plot grid —
one panel per output variable — plus a formatted pandas table. Additional
parameters can be fixed away from their defaults:

```python
run(T_years=np.arange(5, 50, 5))
```

The notebook covers all twelve parameters as sweep variables: 12 sections, 6
plots each.

## 4. My Case: Summary and Conclusions

Running `calculate()` at default parameters — a $400K Toronto condo, 20% down,
4% mortgage, 3.75% discount, 3% appreciation, $1,800 equivalent rent, 10-year
horizon:

| Output | Value |
|---|---|
| Future house price | $537,567 |
| Net selling revenue | $239,447 |
| Mortgage NPV | –$86,262 |
| Rent NPV | –$123,007 |
| Mortgage payment monthly | $1,528 |
| Net purchase benefit | **+$36,745** |

Both NPVs are negative — housing is a cost in either case. Ownership wins at
the 10-year horizon because $1,800/month rent, discounted over ten years, costs
more in present value than the net ownership burden. The break-even monthly
rent is approximately **$1,600** — a $200 drop from current Toronto levels
would make the two options equivalent.

At a **5-year horizon** (`H_years = 5`) the result reverses: the transactional
costs at sale are too large to recover in five years, and renting wins.

My personal conclusion was to continue renting. The 10-year ownership advantage
depends on holding the property for the full decade. I was not confident I
could maintain continuous Toronto employment for that long — a job change or
relocation would force an early sale, erasing the benefit and likely producing
a loss once transactional costs are paid. The model confirmed the intuition
quantitatively.

## 5. Scope and Limitations

Mortgage insurance is not modelled — we assume the down payment is large enough
to avoid it (20% or more).

The model captures the main financial flows. It does not capture factors that
are real but hard to quantify: the ability to customize an owned property; the
time cost of property search and eventual sale; the loss of geographic
mobility; the risk of rental income disruption if you relocate and must sublet.
These are outside this analysis.

The code adapts to other property types. For a detached house, set
`condo_fee_and_maintenance` to zero and add property tax. The structure stays
the same.

