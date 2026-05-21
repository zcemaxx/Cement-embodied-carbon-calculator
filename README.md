# Embodied Carbon Calculator for Ordinary Portland Cement (OPC)

A thesis submitted in the fulfilment of the degree of Master of Philosophy in the Faculty of Science and Engineering
<br>

## Background
Building and construction industry is a major contributor to global GHG emissions. Decarbonisation efforts have focused on operational carbon, while embodied carbon has received less attention. To balance the trade-off between carbon savings and costs, this study develops a parametric model for quantifying cradle-to-gate embodied carbon emission factors, initially applied to cement as one of the most widely used and carbon-intensive construction materials.

Existing databases and tools often provide generalised emission factors without fully disclosing their assumptions, system boundaries, data sources, or calculation processes. This study addresses this gap by offering a transparent and replicable methodology for calculating and customising emission factors under defined assumptions and project contexts.

<br>

## Table of Contents
- [Calculation Scope](Calculation_scope)
- [Methodology](#methodology)
  - [A1 – Raw Material Extraction](#a1--raw-material-extraction)
  - [A2 – Transportation](#a2--transportation)
  - [A3 – Manufacturing](#a3--manufacturing)
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

All CO₂ emissions are expressed in kgCO₂ or kgCO₂ per tonne of cement.

For consistency, emissions expressed per unit mass of cement produced are converted to functional unit, such as per 1 tonne of Ordinary Portland Cement (OPC).

<br>

## Methodology

Emissions are calculated using the following general formula:

```
Total EC = ∑ (Activity data × Emission factor)
```

Each life cycle stage breaks down emission sources as follows.

### A1 – Raw Material Extraction

A1 covers two sub-stages: quarrying and crushing & screening.

#### A1.1 – Quarrying

Clinker composition is first determined from its five major mineral components:

| Component | Formula |
|---|---|
| Tricalcium silicate | 3CaO·SiO₂ |
| Dicalcium silicate | 2CaO·SiO₂ |
| Tricalcium aluminate | 3CaO·Al₂O₃ |
| Tetracalcium aluminoferrite | 4CaO·Al₂O₃·Fe₂O₃ |
| Gypsum | CaSO₄·2H₂O |

CaO content in clinker is derived from molecular weight ratios of each component. Limestone required is then back-calculated using:

```
CaCO₃ required = cement mass × clinker% × CaO in clinker% × 1.785
Limestone required = CaCO₃ required / mineral purity
```

Where **1.785** is the stoichiometric ratio of CaCO₃ to CaO (100/56).

For each raw material (Limestone, Clay, Sand, Iron ore), emissions come from two sources:

**Diesel combustion:**
```
CO₂_diesel = diesel consumed (L/tonne) × mass (tonne) × 2.68 (kgCO₂/L)
```

**Blasting (ANFO):**
```
CO₂_blasting = ANFO used (kg/tonne) × mass (tonne) × 0.26 (kgCO₂/kg)
```

Reference values for user guidance:

| Raw Material | Diesel (L/tonne) | ANFO (kg/tonne) |
|---|---|---|
| Limestone | 1.5 – 3.0 | 0.10 – 0.25 |
| Clay | 1.0 – 2.0 | 0.05 – 0.15 |
| Sand | 0.5 – 1.5 | 0.03 – 0.10 |
| Iron ore | 2.0 – 4.0 | 0.15 – 0.30 |

#### A1.2 – Crushing & Screening

Electricity consumed by crushing and screening equipment:

```
CO₂_crushing = Σ [P_i (kW) × t_i (hrs/tonne)] × EF_electricity
```

Reference values for user guidance:

| Equipment | Power (kW) | hrs/tonne |
|---|---|---|
| Primary crusher | 200 – 500 | 0.5 – 1.5 |
| Secondary crusher | 100 – 300 | 0.5 – 1.0 |
| Screening unit | 50 – 150 | 0.3 – 0.8 |

**Total A1:**
```
CO₂_A1 = CO₂_extraction + CO₂_crushing
```

---

### A2 – Transportation

Three transport legs are accounted for:

| Leg | From | To | Mode |
|---|---|---|---|
| 1 | Quarry | Cement plant | Truck / Conveyor / Electric rail / Diesel rail |
| 2 | Fuel supplier | Cement plant | Truck |
| 3 | Gypsum supplier | Cement plant | Truck |

**For truck, electric rail, diesel rail:**
```
CO₂_transport = distance (km) × mass (tonne) × EF_transport (kgCO₂/tonne·km)
```

**For conveyor belt:**
```
CO₂_conveyor = P (kW) × t (hrs) × EF_electricity (kgCO₂/kWh)
```

Transport emission factors:

| Mode | EF (kgCO₂/tonne·km) | Source |
|---|---|---|
| Heavy diesel truck | 0.062 | GLEC Framework |
| Electric rail | 0.022 | GLEC Framework |
| Diesel rail | 0.041 | GLEC Framework |
| Conveyor belt | UK grid (0.233 kgCO₂/kWh) | DESNZ 2023 |

Mass carried per leg:
- Leg 1 → limestone required (tonne, from A1)
- Leg 2 → fuel consumed / 1000 (tonne, from A3)
- Leg 3 → gypsum mass (tonne, derived from clinker %)

**Total A2:**
```
CO₂_A2 = CO₂_leg1 + CO₂_leg2 + CO₂_leg3
```

---

### A3 – Manufacturing

A3 covers three emission sources.

#### A3.1 – Kiln Fuel Combustion

Heat required to raise raw meal to kiln temperature (~1200°C):

```
Q = m × c × ΔT
Fuel consumed = Q / calorific value of fuel
CO₂_combustion = fuel consumed (kg) × EF_fuel (kgCO₂/kg)
```

Where:
- `c` = 0.84 kJ/kg·°C (specific heat capacity of cement raw meal, fixed constant)
- `ΔT` = kiln temperature − ambient temperature (°C)

Available fuel types and emission factors:

| Fuel | Calorific Value (kJ/kg) | EF (kgCO₂/kg) | Source |
|---|---|---|---|
| Coal | 29,307 | 2.42 | IPCC 2006 |
| Natural gas | 55,500 | 2.75 | IPCC 2006 |
| Fuel oil | 41,868 | 3.17 | IPCC 2006 |
| Diesel | 42,700 | 2.68 | IPCC 2006 |
| Petcoke | 32,500 | 3.40 | IPCC 2006 |

#### A3.2 – Electricity Consumption

Electricity consumed by each piece of equipment:

```
CO₂_electricity = Σ [P_i (kW) × t_i (hrs/tonne)] × EF_electricity
```

Where `EF_electricity` = 0.233 kgCO₂/kWh (UK grid, DESNZ 2023).

Equipment covered:

| Equipment | Power (kW) | hrs/tonne |
|---|---|---|
| Raw mill | 1500 – 4000 | 15 – 25 |
| Kiln drive | 800 – 2500 | 20 – 30 |
| Fans & blowers | 500 – 1500 | 20 – 30 |
| Cement mill | 2000 – 5000 | 20 – 35 |
| Conveyors & pumps | 100 – 500 | 20 – 30 |

#### A3.3 – Calcination

CO₂ released from limestone decomposition (CaCO₃ → CaO + CO₂):

```
CO₂_calcination = limestone required (tonne) × CaO in clinker (%) × (44/56) × 1000
```

Where **44/56** is the molecular weight ratio of CO₂ to CaO (fixed chemistry constant).  
Limestone required and CaO content are carried forward from A1 — no separate user input needed.

**Total A3:**
```
CO₂_A3 = CO₂_combustion + CO₂_electricity + CO₂_calcination
```

---

## Assumptions

| Assumption | Value | Justification |
|---|---|---|
| Specific heat capacity of raw meal | 0.84 kJ/kg·°C | Standard literature value for cement raw meal |
| UK grid emission factor | 0.233 kgCO₂/kWh | DESNZ 2023 |
| ANFO emission factor | 0.26 kgCO₂/kg | Sapko et al. (2002) |
| CaCO₃/CaO stoichiometric ratio | 1.785 (100/56) | Basic chemistry |
| CO₂/CaO molecular weight ratio | 44/56 | Basic chemistry |
| Gypsum proportion in clinker | 5% | Standard OPC composition assumption |
| Functional unit | 1 tonne OPC | Cradle-to-gate boundary |

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
git clone https://github.com/yourusername/your-repo-name.git
```

2. Install dependencies:
```
pip install pandas
```

3. Run the calculator:
```
python your_script_name.py
```

4. Follow the prompts to enter your inputs. Reference tables are displayed at each stage to guide your inputs.
