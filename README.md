# PCB Hardware Engineering Utilities Suite

A modular collection of interactive, browser-based tools for modern hardware and PCB designers. Built entirely as single-file HTML dashboards with zero external dependencies—just open directly in any modern browser to use immediately.

![License](https://img.shields.io/badge/license-MIT-blue)
![No Dependencies](https://img.shields.io/badge/dependencies-none-brightgreen)
![HTML/CSS/JS](https://img.shields.io/badge/built%20with-HTML%2FCSS%2FJS-orange)

---

## Suite Tools Overview

### 1. PCB Current Capacity Calculator (`PCB_Current_Calculator.html`)
Determines the current-carrying capacity or temperature rise of PCB traces, standard through-hole vias, and microvias. It features real-time calculation breakdowns calibrated directly against professional analysis packages.
* **Trace Current Sub-tab:** Incorporates width-based cross-sectional area adjustments.
* **Via Current Sub-tab:** Translates hole geometries to equivalent barrel cross-sections to model current capabilities alongside parasitic series resistance, voltage drop, and power dissipation.
* **Microvia Current Sub-tab:** Modifies base via math via aspect/height constraints for microscale geometries.

### 2. PCB Impedance Calculator (`impedance_calculator.html`)
Evaluates the characteristic single-ended and differential impedance of signal routing geometries according to IPC standards.
* Supports **Microstrip** (surface traces) and **Stripline** (embedded internal layers) layout geometries.
* Toggles interactively between single-ended layouts and edge-coupled differential signaling modes.
* Supports fast cross-unit scaling (mil vs. mm) alongside native copper thickness presets (0.5 oz, 1.0 oz, 2.0 oz).

### 3. MOSFET Thermal & Soldermask Dashboard (`soldermask_temp_variation_simulation.html`)
Simulates the interactive thermal behavior of surface-mount power switches (MOSFETs) placed alongside heat-dissipating PCB vias.
* Evaluates how the application of protective soldermask finishes over board vias alters local convection and radiation properties.
* Generates live graphical charts mapping power dissipation vs. final junction operating temperature ($T_j$).
* Allows designers to dial in customizable parameters such as ambient temperatures ($T_{amb}$), power load steps ($P_d$), packages ($R_{\theta JC}$), and target design margin guidelines.

---

## Detailed Technical Formulas & Foundations

### 1. PCB Current Capacity Tool

#### Trace Current
Based on the IPC-2152 curve-fit of Figure 5-2, rearranged to solve for current:

$$i = i' - C$$

$$i' = \left(\frac{A_c}{117.6 \times \Delta T^{-0.913} + 1.15}\right)^{\frac{1}{x}}$$

$$x = \left(0.84 \times \Delta T^{-0.108}\right) + 1.159$$

Where $A_c$ is the corrected cross-sectional area in $\text{mil}^2$, and $C$ is an empirical current offset factor calibrated dynamically based on trace width:

| Trace Width Range | Area Correction ($\Delta A$) | Current Offset ($C$) |
| :--- | :--- | :--- |
| Exact Integers ($1.0, 2.0, 3.0\dots\text{ mm}$) | $0.005\text{ mm}^2$ | $0.33\text{ A}$ |
| Exact Half-Values ($0.5, 1.5, 2.5\dots\text{ mm}$) | $0.005\text{ mm}^2$ | $0.15\text{ A}$ |
| All intermediate widths | $0.003\text{ mm}^2$ | $0.24\text{ A}$ |

$$A_c(\text{mm}^2) = A(\text{mm}^2) - \Delta A$$
$$A_c(\text{mil}^2) = A_c(\text{mm}^2) \times 1550$$

*The corrections minimize divergence from Saturn PCB Toolkit V8.44 (IPC-2152 with modifiers mode) between widths of 0.1mm and 10mm.*

#### Via Current (Through-Hole)
Employs standard IPC-2152 math evaluated on the continuous annular cylinder cross-section:

$$i = \left[\frac{A}{117.6 \times \Delta T^{-0.913} + 1.15}\right]^{\frac{1}{x}} - 0.1$$

$$A = \pi \times (D + T) \times T$$

Where $D$ = finished drill diameter and $T$ = plated copper barrel thickness.
* **Resistance ($R$):** $\rho \times \frac{L}{A}$ where $\rho \approx 1.48 \times 10^{-6}\ \Omega\cdot\text{cm}$ (Electroplated Cu).
* **Voltage Drop ($V$):** $I \times R$
* **Power Dissipation ($P$):** $I^2 \times R$

#### Microvia Current
Applies aspect-dependent scaling constraints directly onto the primary base calculation model:

$$I = I_{\text{base}} \times x_{\text{corr}}$$

| Condition | $x_{\text{corr}}$ |
| :--- | :--- |
| Drill ($D$), Height ($H$), and $10 \times T$ all $\le 0.25\text{ mm}$ | $0.40$ |
| Any of Drill ($D$), Height ($H$), or $10 \times T > 0.25\text{ mm}$ | $0.48$ |

* Supports **hollow** configurations (barrel cylinder: $A = \pi(D+T)T$) and **copper-filled** configurations (full solid cylinder: $A = \pi(\frac{D}{2}+T)^2$).

---

### 2. PCB Impedance Tool

Implements standardized IPC-2141A industry approximation formulas to calculate signal line characteristics.

#### Microstrip (Surface Trace Single-Ended)
Used when signal conductors are exposed on an outer layer over a single solid reference ground plane:

$$Z_0 = \frac{87}{\sqrt{\varepsilon_r + 1.41}} \times \ln\left(\frac{5.98 \times H}{0.8 \times W + T}\right)\ \Omega$$

#### Stripline (Internal Embedded Single-Ended)
Used when signal conductors are fully embedded symmetrically between dual parallel reference planes:

$$Z_0 = \frac{60}{\sqrt{\varepsilon_r}} \times \ln\left(\frac{1.9 \times H}{0.8 \times W + T}\right)\ \Omega$$

Where:
* $W$ = Conductor width
* $T$ = Conductor thickness (copper weight)
* $H$ = Dielectric core height/spacing to reference plane
* $\varepsilon_r$ = Relative permittivity of the dielectric substrate

#### Differential Pair Mode Modification
When edge-coupled differential routing is enabled, the tool adjusts $Z_0$ based on trace gap ($S$) separation to solve for differential impedance ($Z_{\text{diff}}$):

$$Z_{\text{diff\_microstrip}} = 2 \times Z_0 \times \left(1 - 0.48 \times e^{-0.96 \times \frac{S}{H}}\right)\ \Omega$$

$$Z_{\text{diff\_stripline}} = 2 \times Z_0 \times \left(1 - 0.34 \times e^{-2.9 \times \frac{S}{H}}\right)\ \Omega$$

---

### 3. MOSFET Thermal & Soldermask Dashboard

This engine models total junction temperature step profiles for power MOSFET switches on a multi-layer board structure using standard localized nodal thermal meshes:

$$T_j = T_{\text{amb}} + P_d \times \left(R_{\theta JC} + R_{\theta CS} + R_{\theta S\_amb\_eff}\right)$$

#### Soldermask Thermal Effect
The tool calculates variations in performance by evaluating how bare copper vias compare to soldermask-covered or plugged vias. Applying a soldermask alters the surface emissivity and convective boundary layers:

$$R_{\theta S\_amb\_eff} = R_{\theta \text{board\_base}} \times \kappa_{\text{mask}}$$

Where $\kappa_{\text{mask}}$ is an experimental scale factor that shifts effective board radiation and convective cooling based on mask coverage and composition. This allows designers to simulate thermal variations and identify hotspots caused by tented vs. exposed via designs.

---

## Background & Historical Insights

### Why not just use IPC-2152 directly?
IPC-2152 provides engineering charts, not standardized simple equations. The baseline mathematical curve-fit used here was derived by [Lazar Rozenblat at SMPS.us](https://www.smps.us/pcb-calculator.html) using empirical data from IPC-2152 Figure 5-2. This framework was rearranged to solve natively for trace current limits (where the original solves for baseline area), adding localized structural calibration parameters ($\Delta A, C$) to map outputs accurately against standard professional tools.

### The Brooks & Adam Insights
Research by Douglas Brooks and Dr. Johannes Adam highlights that a via's operational core temperature is extensively governed by its attached trace geometry, rather than standalone internal $I^2R$ power generation (which typically accounts for $<3\text{ mW}$ of localized power loss). Therefore:
1. Always size and route standard traces appropriately first.
2. Ensure vias do not introduce cross-sectional bottlenecks.
3. Traditional design handbooks tend to over-specify via cluster counts.

---

## Validation Data

Calculations are evaluated and verified against industry baselines, including **Saturn PCB Toolkit (V8.44)** (IPC-2152 with modifiers mode) and standard empirical thermal tests:

| Case Conditions Evaluated | Suite Output | Baseline Tool | % Deviation |
| :--- | :--- | :--- | :--- |
| Trace: $2\text{ mm}$, $2\text{ oz}$, $\Delta T = 25^\circ\text{C}$ | $6.42\text{ A}$ | $6.56\text{ A}$ | ~2% |
| Trace: $1\text{ mm}$, $1.5\text{ oz}$, $\Delta T = 25^\circ\text{C}$ | $3.41\text{ A}$ | $3.52\text{ A}$ | ~3% |
| Via: $D = 0.3\text{ mm}$, $25\ \mu\text{m}$, $\Delta T = 10^\circ\text{C}$ | $1.58\text{ A}$ | $1.90\text{ A}$ | ~17%* |
| Microvia: $D = 0.1\text{ mm}$, $H = 0.1\text{ mm}$ | $0.356\text{ A}$ | $0.382\text{ A}$ | ~7% |
| Single-Ended Microstrip: $W=6\text{ mil}, H=4\text{ mil}, \varepsilon_r=4.2$ | $50.3\ \Omega$ | $50.9\ \Omega$ | ~1.2% |

*\*Via current variance is intended: professional suites like Saturn implement custom multi-layer modifiers (such as nearby inner-layer power plane sinks and multi-layer board thicknesses) which augment current ratings. This tool calculates the conservative baseline standard.*

---

## Limitations

* **No Multi-layer Environmental Plane Factor Corrections:** Does not calculate environmental modifications for nearby continuous copper plane sinks or variable substrate choices directly within the trace or via calculations.
* **Nominal Geometry Assumptions:** Traces are evaluated assuming perfect rectangular dimensions. Physical board manufacturing etch factors (sloped conductor sidewalls) are omitted.
* **Mathematical Limits of IPC-2141A Approximations:** IPC-2141A closed-form formulas provide standard estimates for traditional designs, but thin dielectrics or extreme aspect ratios benefit from numerical field solvers (BEM/FEM).
* **Calibrated Component Range:** Best performance is achieved within standard boundaries:
  * Conductor Widths: $0.1 \text{ to } 10\text{ mm}$
  * Base Foil Thicknesses: $0.5 \text{ to } 3.0\text{ oz}$
  * Target Core Temperature Rises: $5^\circ\text{C} \text{ to } 100^\circ\text{C}$

---

## Core References & Literature

* **IPC-2152 (2009)** — Standard for Determining Current Carrying Capacity in Printed Board Design
* **IPC-2141A (2004)** — Design Guide for High-Speed Controlled Impedance Circuit Boards
* **IPC-6012** — Qualification and Performance Specification for Rigid Printed Boards
* **IPC-4761** — Design Guide for Protection of Printed Board Via Structures
* **Brooks & Adam (2021)** — *PCB Trace and Via Currents and Temperatures: The Complete Analysis*
* **SMPS.us Mathematical Modeling** — PCB Trace Width Interpolation Curve Models

---

## Repository File Structure

```text
pcb-hardware-engineering-suite/
├── PCB_Current_Calculator.html                # Trace, via, and microvia capacity calculator
├── impedance_calculator.html                  # Controlled impedance layout geometry calculator
├── soldermask_temp_variation_simulation.html  # MOSFET dashboard and via mask thermal modeler
├── README.md                                  # Comprehensive documentation (this file)
└── LICENSE                                    # MIT License
