# PCB Current Capacity Calculator

A browser-based calculator for determining the current-carrying capacity of PCB traces, through-hole vias, and microvias. Built as a single HTML file with zero dependencies — just open and use.

![License](https://img.shields.io/badge/license-MIT-blue)
![No Dependencies](https://img.shields.io/badge/dependencies-none-brightgreen)
![HTML](https://img.shields.io/badge/built%20with-HTML%2FCSS%2FJS-orange)

---

## Features

- **Trace Current Calculator** — Width-based correction factors calibrated against Saturn PCB Toolkit
- **Via Current Calculator** — Barrel area model with resistance, voltage drop, and power loss
- **Microvia Current Calculator** — Height-dependent correction factor for hollow and copper-filled microvias
- Real-time calculation with step-by-step breakdown
- All formulas displayed inline for transparency
- Fully responsive — works on desktop and mobile
- Single file, no build step, no server needed

---

## Formulas

### Trace Current

Based on the IPC-2152 / SMPS.us curve-fit of Figure 5-2, rearranged to solve for current:

```
i = i' − C

i' = (Ac / (117.6 × ΔT^(−0.913) + 1.15))^(1/x)

x = (0.84 × ΔT^(−0.108)) + 1.159
```

Where `Ac` is the corrected cross-sectional area in mil², and `C` is a current offset. Both depend on trace width:

| Trace Width (mm) | Area Correction (ΔA) | Current Offset (C) |
|---|---|---|
| Integer (1.0, 2.0, 3.0 ...) | 0.005 mm² | 0.33 A |
| Half (0.5, 1.5, 2.5 ...) | 0.005 mm² | 0.15 A |
| In between (all others) | 0.003 mm² | 0.24 A |

```
Ac (mm²) = A (mm²) − ΔA
Ac (mil²) = Ac (mm²) × 1550
```

The correction factors were empirically calibrated against Saturn PCB Toolkit V8.44 (IPC-2152 with modifiers mode) across trace widths from 0.1 to 10 mm.

### Via Current (Through-Hole)

Uses the same IPC-2152 base formula with the via barrel cross-section area:

```
i = [A / (117.6 × ΔT^(−0.913) + 1.15)]^(1/x) − 0.1

A = π × (D + T) × T
```

Where `D` = drill diameter, `T` = plating thickness.

Also computes:
- **Resistance**: `R = ρ × L / A` (ρ = 1.48×10⁻⁶ Ω·cm for electroplated Cu)
- **Voltage drop**: `V = I × R`
- **Power loss**: `P = I² × R`

### Microvia Current

Applies a height-dependent correction factor to the base via current:

```
I = I_base × x
```

| Condition | x |
|---|---|
| D, H, and 10×T all ≤ 0.25 mm | 0.40 |
| Any of D, H, or 10×T > 0.25 mm | 0.48 |

Supports both **hollow** (barrel only: `A = π(D+T)T`) and **copper-filled** (full cross-section: `A = π(D/2+T)²`) microvias.

---

## Background

### Why not just use IPC-2152 directly?

IPC-2152 provides charts, not equations. The base formula used here was derived by [Lazar Rozenblat at SMPS.us](https://www.smps.us/pcb-calculator.html) by curve-fitting IPC-2152 Figure 5-2 data. We rearranged it to solve for current (the original solves for area), then added empirical correction factors to match Saturn PCB Toolkit's output.

### The Brooks & Adam Insight

Research by Douglas Brooks and Dr. Johannes Adam showed that a via's temperature is determined by the connected traces, not by the via's own I²R heating. The via's self-heating is typically < 3 mW — negligible. This means:

- Size the traces first
- The via only needs to not be a thermal bottleneck
- Fewer and smaller vias are needed than traditional methods suggest

### Why correction factors?

The raw IPC-2152 curve-fit gives a baseline that doesn't match Saturn PCB Toolkit's output (which applies board thickness, copper weight, plane proximity, and other IPC-2152 modifiers). The width-dependent ΔA and C corrections close this gap empirically, giving results within ~2% of Saturn for typical designs.

---

## Validation

Results have been compared against Saturn PCB Toolkit V8.44 (IPC-2152 with modifiers mode):

| Case | This Calculator | Saturn PCB | Difference |
|---|---|---|---|
| Trace: 2mm, 2oz, ΔT=25°C | 6.42 A | 6.56 A | ~2% |
| Trace: 1mm, 1.5oz, ΔT=25°C | 3.41 A | 3.52 A | ~3% |
| Via: D=0.3mm, 25µm, ΔT=10°C | 1.58 A | 1.90 A | ~17%* |
| Microvia: D=0.1mm, H=0.1mm | 0.356 A | 0.382 A | ~7% |

*Via current difference is expected — Saturn applies additional IPC-2152 modifiers (board thickness, plane proximity) that increase the current rating. This calculator provides the conservative baseline.

---

## Limitations

- **No IPC-2152 environmental correction factors** — The calculator does not apply board thickness, plane proximity, or material corrections. Results represent the baseline (conservative) estimate. For the full corrected value, use [Saturn PCB Toolkit](https://saturnpcb.com/saturn-pcb-toolkit/) (free) or multiply using the virtual temperature method described in IPC-2152.
- **No etch factor** — The trace area uses the nominal rectangular cross-section. Saturn applies an etch factor (typically 2:1) that slightly reduces the effective area.
- **Empirical corrections are calibrated for a specific range** — Width: 0.1–10 mm, copper: 0.5–3 oz, ΔT: 5–100°C. Results outside this range may be less accurate.
- **Via formula does not include height** — Per Brooks & Adam, the via temperature is set by the traces, not the via itself. For microvias, the separate height correction is provided.
- **No parasitic impedance** — This version calculates current capacity only. Parasitic capacitance (C = 1.41εrTD1/(D2−D1)), inductance (L = 5.08h[ln(4h/d)+1]), and characteristic impedance (Z₀ = 60/√εr × ln(D_antipad/D_barrel)) are covered in the companion white paper but not in this calculator version.

---

## References

- **IPC-2152** (2009) — Standard for Determining Current Carrying Capacity in Printed Board Design
- **IPC-6012** — Qualification and Performance Specification for Rigid PCBs
- **IPC-4761** — Design Guide for Protection of Printed Board Via Structures
- **SMPS.us** — [PCB Trace Width Calculator and Equations](https://www.smps.us/pcb-calculator.html)
- **Brooks & Adam** — *PCB Trace and Via Currents and Temperatures* (2021)
- **Saturn PCB Toolkit** — [Free download](https://saturnpcb.com/saturn-pcb-toolkit/) (V8.44 used for validation)
- **NinjaCalc** — [IPC-2152 Track Current Calculator](https://ninjacalc.mbedded.ninja/calculators/electronics/pcb-design/track-current-ipc2152)

---

## File Structure

```
pcb-current-calculator/
├── index.html          # The calculator (single file, self-contained)
├── README.md           # This file
└── LICENSE             # MIT License
```

---

## Contributing

Contributions are welcome. Areas that would be particularly valuable:

- Adding IPC-2152 correction factors (board thickness, plane proximity, copper weight)
- Adding a via parasitics tab (capacitance, inductance, impedance)
- Adding differential pair via impedance calculation
- Validating against additional data points from Saturn or physical measurements
- Adding export/print functionality for calculation results

To contribute:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/add-parasitic-tab`)
3. Commit your changes (`git commit -m 'Add via parasitic calculations'`)
4. Push to the branch (`git push origin feature/add-parasitic-tab`)
5. Open a Pull Request

---

## License


---

## Acknowledgments

- **Lazar Rozenblat** (SMPS.us) for the IPC-2152 curve-fit interpolation formula
- **Jack Olson** (Caterpillar) for the original IPC-2152 coefficient extraction
- **Douglas Brooks & Dr. Johannes Adam** for the via thermal research
- **Matt Dirks** (Saturn PCB Design) for Saturn PCB Toolkit used in validation
