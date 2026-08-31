# Embodied Carbon Calculator for Ordinary Portland Cement (OPC)

A thesis submitted in the fulfilment of the degree of Master of Philosophy in the Faculty of Science and Engineering, University of Manchester.
<br>

## Background

Building and construction industry is a major contributor to global GHG emissions. Decarbonisation efforts have focused on operational carbon, while embodied carbon has received less attention. To balance the trade-off between carbon savings and costs, this study develops a parametric model for quantifying cradle-to-gate embodied carbon, initially applied to cement as one of the most widely used and carbon-intensive construction materials.

Existing databases and tools often provide generalised emission factors without fully disclosing their assumptions, system boundaries, data sources, or calculation processes. This study addresses this gap by offering a transparent and replicable methodology for calculating and customising emission factors under defined assumptions and project contexts.

Flexibility is central to the design methodology. The model allows emission factors to be customized based on project-specific parameters. Where measured values are unavailable, pressing Enter automatically substitutes a literature-referenced default value, ensuring that even a partial dataset can generate a complete and traceable result.

<br>

## Table of Contents
- [Calculation Scope](#calculation-scope)
- [Methodology](#methodology)
  - [A1 – Raw Material Supply](#a1--raw-material-supply)
  - [A2 – Transportation](#a2--transportation)
  - [A3 – Manufacturing](#a3--manufacturing)
- [Reference Result](#reference-result)
- [Parameters](#parameters)
- [Assumptions](#assumptions)
- [References](#references)
- [How to Run](#how-to-run)

<br>

## Calculation Scope
### System Boundary

This calculator covers cradle-to-gate embodied carbon, which includes:

| Stage | Description | Included |
|---|---|---|
| A1 | Raw material extraction | ✅ |
| A2 | Transportation | ✅ |
| A3 | Manufacturing | ✅ |
| A4 | Transport to site | ❌ |
| A5 | Installation | ❌ |

### Functional Unit

All results are expressed in **kgCO₂e per tonne of finished cement**.

For consistency, emissions expressed per unit mass of cement produced are converted to functional unit, such as per 1 tonne of Ordinary Portland Cement (OPC).

### Cement Types

| Type | Clinker | Gypsum | SCM | SCM supply factor |
|---|---|---|---|---|
| CEM I | 95% | 5% | – | – |
| CEM II/B-S | 70% | 5% | 25% GGBS | 79 kgCO₂e/t |
| CEM III/A | 45% | 5% | 50% GGBS | 79 kgCO₂e/t |

Selecting a type pre-loads its composition; all three fractions remain editable and are validated to sum to 100%.

<br>

## Methodology

Emissions are calculated using the following general formula:

```
Total EC = ∑ (Activity data × Emission factor)
```

Each life cycle stage breaks down emission sources as follows:  

![Figure 1](https://github.com/zcemaxx/Cement-embodied-carbon-calculator/blob/main/Figures/Full%20cement%20manufacturing%20process.jpg)


### A1 – Raw Material Extraction

A1 covers quarrying (A1.1), crushing and screening (A1.2), and purchased materials (A1.3). Emission sources include diesel combustion and ANFO blasting in quarrying, and electricity consumption in crushing & screening.

#### Clinker chemistry and the material balance

The raw material demand is derived from clinker chemistry rather than assumed. CaO mass fraction is computed from molar masses for each of the four Bogue phases:

Cao fraction example - tricalcium silicate (3CaO·SiO₂)
```
CaO weight in each component is derived from molecular weight ratios.
In tricalcium silicate (3CaO·SiO₂, molecular weight = 228):
CaO weight = (56 × 3) / 228 = 0.74
```

| Phase | Formula | Default proportion | CaO fraction |
|---|---|---|---|
| C3S – alite | 3CaO·SiO₂ | 52.6% | 0.7368 |
| C2S – belite | 2CaO·SiO₂ | 26.3% | 0.6512 |
| C3A – aluminate | 3CaO·Al₂O₃ | 10.6% | 0.6226 |
| C4AF – ferrite | 4CaO·Al₂O₃·Fe₂O₃ | 10.5% | 0.4616 |
| Gypsum | CaSO₄·2H₂O | 0 | 0.00 |

Gypsum does not appear in this table. It is interground after the kiln and is not a clinker phase, so it is accounted separately as 5% of the finished cement. Including it here would understate the clinker phases and, through them, the calcination CO₂.

To estimate the raw materials required for producing 1 tonne cement, for instance, we calculate total CaO content as the weighted sum of CaO across all components:
```
CaO in clinker (%) = Σ [proportion (%) × CaO weight]
```
With the default composition this gives 67.33%.
Limestone demand is then back-calculated through the CaCO₃/CaO stoichiometric ratio (100/56 = 1.785) and the quarried mineral purity:
```
CaCO₃ required   = cement mass × clinker% × CaO in clinker% × 1.785
Limestone required = CaCO₃ required / mineral purity
```
For CEM I, it requires 1.142 t of CaCO₃, and at 95% purity 1.202 t of quarried limestone is supplied per tonne of cement.

Non-carbonate raw materials are entered per tonne of clinker and scaled to the cement basis:

| Raw material | Default (t per t clinker) | Per tonne CEM I |
|---|---|---|
| Limestone | derived from chemistry | 1.202 t |
| Clay | 0.20 | 0.190 t |
| Sand | 0.03 | 0.028 t |
| Iron ore | 0.02 | 0.019 t |
| **Total quarried rock** | | **1.439 t** |

Two masses are carried forward. CaCO₃ (1.142 t) drives calcination, because only carbonate decomposes. Quarried rock (1.439 t) drives digging, crushing and hauling, because all of it is handled.

#### A1.1 – Quarrying

Emissions arise from mobile plant diesel and from blasting, both applied to the total quarried rock:

```
CO₂_diesel   = quarried rock (t) × diesel (L/t) × 2.68 (kgCO₂/L)
CO₂_blasting = quarried rock (t) × ANFO (kg/t) × 0.26 (kgCO₂/kg)
```
where 2.68 kgCO₂/L and 0.26 kgCO₂/kg are emission factors of diesel and ANFO.

Reference values of diesel and ANFO consumed to quarry per tonne of raw materials, and with default OPC Type 1 composition:

| Raw Material | Mass / tonne OPC (kg) | Diesel (L/tonne) | ANFO (kg/tonne) |
|---|---|---|---|
| Limestone | 1210 | 2 | 0.18 |
| Clay | 200 | 1.5 | 0.10 |
| Sand | 50 | 1 | 0.06 |
| Iron ore | 30 | 3 | 0.22 |

#### A1.2 – Crushing & Screening

Crushing energy is derived from equipment throughput and motor rating, so the specific energy follows from the plant's own machines:

```
Specific energy (kWh/t) = Σ [P_i (kW) / throughput_i (t/h)]
CO₂_crushing = quarried rock (t) × specific energy × EF_electricity (kgCO₂/kWh)
```
where EF_electricity = 0.233 kgCO₂/kWh is applied in the UK.

The number of crusher stages is entered by the user, then throughput and power for each. Defaults values of equipment:

| Equipment | Power (kW) | tonne/hr |
|---|---|---|
| Primary (jaw) crusher | 220 | 600 |
| Secondary (impact) crusher | 150 | 400 |
| Screening unit | 50 – 150 | 5 |

#### A1.3 – Purchased Materials

Gypsum and any SCM are bought in rather than quarried by the plant, so their upstream production is included at 15 kgCO₂e/t and 79 kgCO₂e/t respectively.

#### Total A1
Total A1 emissions are the sum of extraction and crushing & screening, in formula of:
```
CO₂_A1 = CO₂_extraction + CO₂_crushing
```

<br>

### A2 – Transportation

A2 covers four legs. Each takes its own mode and one-way distance.

| Leg | From | To | Default mode | Default distance |
|---|---|---|---|---|
| 1 | Quarry | Cement plant | Truck | 50 km |
| 2 | Gypsum supplier | Cement plant | Truck | 200 km |
| 3 | SCM supplier | Cement plant | Electric rail | 300 km |
| 4 | Fuel supplier | Cement plant | Diesel rail | 300 km |

Emissions are calculated with general formula, for truck, electric rail, diesel rail:
```
CO₂_transport = distance (km) × mass (tonne) × EF_transport (kgCO₂/tonne·km)
```

For conveyor belt:
```
CO₂_conveyor = P (kW) × t (hrs) × EF_electricity (kgCO₂/kWh)
```

Transport emission factors referred in calculation are listed below:

| Mode | EF (kgCO₂/tonne·km) 
|---|---|
| Heavy diesel truck | 0.062 
| Electric rail | 0.022 
| Diesel rail | 0.041 
| Conveyor belt | UK grid (0.233 kgCO₂/kWh) 

These are GLEC well-to-wheel factors, which already allow for typical empty running. Return legs are therefore not counted separately, doing so would double-count.

Masses are carried forward automatically — quarried rock from A1, gypsum and SCM from the cement composition, and kiln fuel from A3.1.

<br>

### A3 – Manufacturing

A3 covers three emission sources: kiln fuel combustion (A3.1), electricity consumption (A3.2), and calcination (A3.3).

#### A3.1 – Kiln Fuel Combustion

Heat required to raise raw meal to kiln temperature (~1200°C) is calculated with:

```
Q = m × c × ΔT
```
Where:
- `m` = mass of raw meal (kg)
- `c` = 0.84 kJ/kg·°C (specific heat capacity of cement raw meal, fixed constant)
- `ΔT` = kiln temperature − ambient temperature (°C)

The default heat required is 3,000 MJ per tonne of clinker, within the 3.0–3.4 GJ/t range typical of modern dry-process kilns with preheater and precalciner.

The fuel consumed and CO₂ produced from the heat generated:
```
Fuel consumed (kg) = Q / calorific value of fuel (KJ/kg)
CO₂_combustion = fuel consumed (kg) × EF_fuel (kgCO₂/kg)
```

Fuel is entered as a mix, given as percentage shares of kiln heat that are normalised to 100%. This allows co-firing scenarios rather than forcing a single fuel. Calorific values and emission factors are listed below:

| Fuel | Net Calorific Value (MJ/kg) | EF (kgCO₂/kg) |
|---|---|---|
| Coal | 25.8 | 2.42 |
| Petcoke | 32.5 | 3.40 |
| Natural gas | 48.0 | 2.75 |
| Fuel oil | 40.4 | 3.17 |
| Diesel | 43.0 | 3.17 |

Both columns are on a net calorific value basis. Mixing a gross-CV emission factor with a net CV overstates fuel demand by roughly 5–10% and is a common error.

#### A3.2 – Electricity Consumption

Electricity consumed by each piece of equipment is calculated separately and then summed up:

```
CO₂_electricity = Σ [P_i (kW) × t_i (hrs/tonne)] × EF_electricity
```

Reference values of equipment user inputs:

| Equipment | Default (kWh per tonne cement) |
|---|---|
| Raw mill | 25 |
| Kiln drive and preheater fans | 25 |
| Clinker cooler fans | 5 |
| Cement (finish) mill | 40 |
| Conveyors, pumps and packing | 8 |
| **Total** | **103** |

This total sits in the 90–120 kWh/t range reported for modern plants. The finish mill is the single largest electrical load and must not be omitted.

`EF_electricity` defaults to 0.233 kgCO₂e/kWh (UK grid, DESNZ 2023) and is editable, so non-UK or contracted-supply scenarios can be modelled.

#### A3.3 – Calcination

CO₂ released from limestone decomposition (CaCO₃ → CaO + CO₂):

```
CO₂_calcination = CaCO₃ required (tonne) × (44/100) × 1000
```

The mass entering this equation is pure CaCO₃, not the as-quarried limestone. Limestone at 95% purity contains 5% non-carbonate material that does not calcine; using the quarried mass would overstate this term by the reciprocal of the purity.

<br>

## Reference Result

All defaults, CEM I, 1 tonne of cement:

| Line item	| kgCO₂e |
| --- | --- |
| A1.1 quarrying – mobile plant diesel	| 3.86 |
| A1.1 quarrying – blasting (ANFO)	| 0.13 |
| A1.2 crushing and screening	| 0.25 |
| A1.3 gypsum supply	| 0.75 |
| A2 raw materials quarry → plant	| 4.46 |
| A2 gypsum supplier → plant	| 0.62 |
| A2 fuel supplier → plant	| 1.17 |
| A3.1 kiln combustion – coal (70%)	| 187.13 |
| A3.1 kiln combustion – natural gas (30%)	| 48.98 |
| A3.2 electricity (5 equipment groups)	| 24.01 |
| A3.3 calcination of CaCO₃	| 501.98 |

| Stage | kgCO₂e/t | Share |
|---|---|---|
| A1 | 4.99 | 0.6% |
| A2 | 6.25 | 0.8% |
| A3 | 762.09 | 98.5% |
| **Total A1–A3** | **773.33** | **100%** |

<br>

## Parameters

All 28 parameters are prompted for at run time. `list_params()` prints the full set with current values.

| Group | Parameters |
|---|---|
| Cement composition | cement type, clinker %, gypsum %, SCM %, SCM supply factor |
| Clinker phases | C3S, C2S, C3A, C4AF proportions |
| Raw materials | limestone purity, clay / sand / iron ore per t clinker |
| A1.1 quarrying | diesel L/t, diesel EF, explosive kg/t, explosive EF, gypsum EF |
| A1.2 crushing | number of stages, throughput and motor rating per stage |
| A2 transport | mode and distance for each of four legs |
| A3.1 kiln | specific heat consumption, fuel mix shares |
| A3.2 electricity | five equipment groups, kWh per tonne cement |
| Grid | electricity emission factor |

Inputs are validated as they are entered: values are range-checked, non-numeric entries re-prompt, and the three percentage sets are normalised to 100%. `validate()` re-checks the complete set before any calculation runs, so an inconsistent parameter set raises a named error rather than producing a plausible-looking wrong number.

<br>

## Assumptions

| Assumption | Value | Justification |
|---|---|---|
| Kiln specific heat consumption | 3,000 MJ/t clinker | Typical modern dry kiln with preheater/precalciner |
| UK grid emission factor | 0.233 kgCO₂/kWh | DESNZ 2023 |
| ANFO emission factor | 0.26 kgCO₂/kg | Sapko et al. (2002) |
| Diesel emission factor | 2.68 kgCO₂/L | DESNZ 2023 |
| CaCO₃/CaO stoichiometric ratio | 1.785 (100/56) | Basic chemistry |
| CO₂/CaCO₃ molecular weight ratio | 0.44 (44/100) | Basic chemistry |
| Gypsum proportion in clinker | 5% of finished cement| Standard OPC composition assumption; added post-kiln, not a clinker phase |
| Functional unit | 1 tonne OPC | Cradle-to-gate boundary |
| CaO source attribution | 100% carbonate | IPCC Tier 2 |
| Transport factors | GLEC well-to-wheel, incl. empty running	| Return legs not counted separately|

---

## References

- IPCC (2006). *2006 IPCC Guidelines for National Greenhouse Gas Inventories, Volume 3: Industrial Processes and Product Use.* IPCC.
- DESNZ (2023). *UK Government GHG Conversion Factors for Company Reporting.* Department for Energy Security and Net Zero.
- GLEC Framework (2019). *Global Logistics Emissions Council Framework for Logistics Emissions Accounting and Reporting.* Smart Freight Centre.
- Sapko, M.J. et al. (2002). *Experimental and Theoretical Chemical Equilibrium Explosive Output.* Proceedings of the 28th Annual Conference on Explosives and Blasting Technique.
- EN 15804:2012+A2:2019. *Sustainability of Construction Works — Environmental Product Declarations.*

---

## How to Run

1. Clone the repository:
```
git clone https://github.com/zcemaxx/Cement-embodied-carbon-calculator.git
```

2. Open Cement_Embodied.ipynb in Jupyter or VS Code, and run the cells in order.

3. Run the calculator:
```
inp = main()
```

4. Answer the prompts. Press Enter, or type na, to accept the default shown in brackets. Every question shows its default and its permitted range.
