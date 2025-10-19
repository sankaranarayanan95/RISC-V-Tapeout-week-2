# Day 3 — CMOS Inverter Analysis (Enhanced Lab Report)

📅 Date: 2025-10-19  
👩‍💻 Author: sankaranarayanan95  
⚙️ PDK: SkyWater Sky130  
🏷️ Devices: PMOS / NMOS (W/L sizing shown below)

---

## Contents

- Overview
- Files & directory
- Quick-run commands
- Netlist snippets & measurement directives (.measure)
- Transient Analysis — results, graphs & interpretation
- Voltage Transfer Characteristic (VTC) — results, graphs & interpretation
- Comparison tables and quick insights
- Advanced visualizations (Python) — how to export and plot
- What to try next
- Appendix: Example netlist snippets & recommended .measure statements

---

## Overview

This document expands your Day 3 lab: we combine dynamic (transient) and static (VTC/DC sweep) analyses for a CMOS inverter built with Sky130 transistors. It includes:

- Clear commands to re-run the simulations
- Recommended .measure statements to extract rise/fall times, delays, switching threshold and noise margins automatically
- Embedded screenshots from your ngspice runs
- Tables summarizing transistor sizes and (placeholder) measurements
- Python plotting recipe to create publication-quality graphs (gain curve, dVout/dVin, annotated thresholds)
- Next experiments and variations to explore

---

## Files & Directory

Location (from your notes):

```
cd sky130CircuitDesignWorkshop/design/
```

Key SPICE netlists used:
- `day3_inv_tran_Wp084_Wn036.spice` — transient switching with time-domain input
- `day3_inv_vtc_Wp084_Wn036.spice` — DC sweep for VTC

Screenshot (open netlist in vim to inspect connections and sources):

![Netlist screenshot 1](https://github.com/user-attachments/assets/ed182767-4a54-4039-8cb6-6d80f9f32a35)

Other session screenshots:
- Transient plot screenshots:
  - ![Transient plot 1](https://github.com/user-attachments/assets/974114f3-6b3f-497f-9aa2-318d8a05191d)
  - ![Transient plot 2](https://github.com/user-attachments/assets/0826d46d-4e8d-4813-960e-b96c266a710a)
  - ![Transient plot full](https://github.com/user-attachments/assets/885b96c8-f3eb-4a95-b84f-21c15d7d6a59)
- VTC / DC sweep screenshots:
  - ![VTC plot 1](https://github.com/user-attachments/assets/d58e51ef-bc3c-420d-ac5d-8f00ce99ee97)
  - ![VTC plot 2](https://github.com/user-attachments/assets/fad418c5-17ab-4a95-b4cb-a0dbca4e3e4c)
  - ![VTC plot full](https://github.com/user-attachments/assets/7628f29c-6316-4261-9c74-47e0ccf54da5)

---

## Transistor sizing & Simulation Parameters

| Device | W (µm) | L (µm) | Notes |
|---:|---:|---:|---|
| PMOS (Wp) | 0.84 | 0.18 (assumed) | from filename: Wp084 |
| NMOS (Wn) | 0.36 | 0.18 (assumed) | from filename: Wn036 |
| Vdd | 1.8 V (typical for Sky130 experiments) | — | confirm in netlist |
| Input waveform | pulse (0→Vdd) | rise/fall/time set in netlist | check `.tran` source |

(If L is different in your netlist, update the table accordingly.)

---

## Quick-run commands

Open directory, run ngspice:

```
cd sky130CircuitDesignWorkshop/design/
ngspice day3_inv_tran_Wp084_Wn036.spice
# inside ngspice:
plot v(out) v(in)
# or to export:
wrdata tran_out.csv v(out) v(in)
```

For VTC:

```
ngspice day3_inv_vtc_Wp084_Wn036.spice
# inside ngspice:
plot v(out) v(in)
# export:
wrdata vtc_out.csv v(in) v(out)
```

Tip: Use `.control` blocks (or the `ngspice` CLI) to automate running & writing CSV for use with Python.

---

## Add these .measure statements to your netlists

Include the lines below in your transient netlist (adjust thresholds if Vdd differs):

```
* Example .measure directives (place before .end)
.meas tran trise TRIG v(out) VAL=0.1*Vdd TD=0 RISE=1 TARG v(out) VAL=0.9*Vdd
.meas tran tfall TRIG v(out) VAL=0.9*Vdd TD=0 FALL=1 TARG v(out) VAL=0.1*Vdd
.meas tran tpLH TRIG v(in) VAL=0.5*Vdd TD=0 RISE=1 TARG v(out) VAL=0.5*Vdd
.meas tran tpHL TRIG v(in) VAL=0.5*Vdd TD=0 FALL=1 TARG v(out) VAL=0.5*Vdd
.meas tran tprop_avg param='(tpLH+tpHL)/2'
.meas tran Ipeak_max MAX I(Vdd_supply) ; name Vdd supply current source appropriately
```

For VTC / DC sweep:

```
.meas dc VIL min V(in) when V(out)=0.9*Vdd
.meas dc VIH max V(in) when V(out)=0.1*Vdd
.meas dc Vth param='V(in) when d(Vout)/d(Vin) is maximum' ; use script or post-process for dV/dVin peak
```

Note: ngspice cannot directly differentiate DC sweep inside `.measure` easily. Export V(in)/V(out) and compute dVout/dVin in Python to find precise switching point (where slope is steepest).

---

## Transient Analysis — Results & Graphs

Instructions:
1. Add `.measure` lines shown above to the transient netlist.
2. Run `ngspice day3_inv_tran_Wp084_Wn036.spice`.
3. Capture automatic `.meas` outputs (ngspice prints measured values to console) or export waveforms to CSV for further analysis.

Example placeholder results table (replace placeholders after running simulation):

| Metric | Measured value | Units | Notes |
|---:|---:|---:|---|
| Rise time (10→90%) trise | 45 | ps | measured on v(out) with .measure |
| Fall time (90→10%) tfall | 38 | ps | faster due to smaller NMOS width? verify |
| Propagation delay tPLH (input→output 50%) | 60 | ps | tpLH |
| Propagation delay tPHL (input→output 50%) | 72 | ps | tpHL |
| Average propagation delay | 66 | ps | (tpLH+tpHL)/2 |

(These are example placeholders — run .measure to get your actual numbers and replace the table.)

Embedded screenshots of your transient plots:

![Transient plot 1](https://github.com/user-attachments/assets/974114f3-6b3f-497f-9aa2-318d8a05191d)
![Transient plot 2](https://github.com/user-attachments/assets/0826d46d-4e8d-4813-960e-b96c266a710a)

Interpretation notes:
- Rise/fall times depend on load capacitance and transistor strengths (W/L).
- If PMOS is wider (Wp larger), expect slower rise due to mobility but sizing compensates.
- Check current draw spikes during switching (short-circuit current).

---

## VTC — Results, Gain and Noise Margins

Run the VTC DC sweep netlist and export V(in) and V(out) to CSV for post-processing.

Key plots to produce:
- Vout vs Vin (VTC)
- dVout/dVin vs Vin (gain curve) — maximum of |dVout/dVin| indicates switching threshold (VM)
- Zoom near transition to annotate VIN at VOUT = VDD/2

Sample VTC screenshots (from your session):

![VTC plot 1](https://github.com/user-attachments/assets/d58e51ef-bc3c-420d-ac5d-8f00ce99ee97)
![VTC detail](https://github.com/user-attachments/assets/fad418c5-17ab-4a95-b4cb-a0dbca4e3e4c)

Suggested VTC metrics table (fill after running/extraction):

| Metric | Value | Units | Notes |
|---:|---:|---:|---|
| Switching point VM (where Vout=Vin or max gain) | 0.9 | V | compute from CSV where dVout/dVin is max or Vout=Vdd/2 |
| Noise margin (NMH) | 0.45 | V | Voh - Vih |
| Noise margin (NML) | 0.47 | V | Vil - Vol |
| Static gain (|dVout/dVin|max) | 18 | unitless | slope magnitude at transition |

How to compute VM and gains:
- Export V(in), V(out) as CSV, then compute derivative dVout/dVin numerically using central differences in Python (script below).
- The VIN at which |dVout/dVin| is maximum is the switching point (VM). Vout(VM) is typically Vdd/2 for symmetric devices.

---

## Advanced visualization: Python plotting recipe

Save `vtc_out.csv` and `tran_out.csv` from ngspice (use `wrdata` or `export` commands in ngspice). Use the following Python snippet to create publication-quality plots and compute metrics:

```python
# example_plot.py
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

# Load VTC CSV: assume columns 'time', 'vin', 'vout' or 'vin','vout'
vtc = pd.read_csv('vtc_out.csv')
vin = vtc['vin'].to_numpy()
vout = vtc['vout'].to_numpy()
Vdd = vin.max()

# compute derivative dVout/dVin
dV = np.gradient(vout, vin)
imax = np.argmax(np.abs(dV))
VM = vin[imax]
gain_max = dV[imax]

# Plot VTC
plt.figure(figsize=(6,5))
plt.plot(vin, vout, lw=2, label='Vout vs Vin')
plt.axvline(VM, color='red', ls='--', label=f'VM = {VM:.3f} V')
plt.xlabel('Vin (V)')
plt.ylabel('Vout (V)')
plt.title('Voltage Transfer Characteristic')
plt.grid(True)
plt.legend()
plt.savefig('vtc_plot.png', dpi=150)

# Plot gain
plt.figure(figsize=(6,4))
plt.plot(vin, dV, lw=2)
plt.axvline(VM, color='red', ls='--', label=f'|gain|max @ {VM:.3f} V')
plt.xlabel('Vin (V)')
plt.ylabel('dVout/dVin')
plt.title('DC Gain (dVout/dVin)')
plt.grid(True)
plt.legend()
plt.savefig('vtc_gain.png', dpi=150)

# Transient plotting example (from tran_out.csv)
tran = pd.read_csv('tran_out.csv')
plt.figure(figsize=(8,4))
plt.plot(tran['time'], tran['v(in)'], label='Vin')
plt.plot(tran['time'], tran['v(out)'], label='Vout')
plt.xlabel('Time (s)')
plt.ylabel('Voltage (V)')
plt.title('Transient Response')
plt.legend()
plt.grid(True)
plt.savefig('tran_plot.png', dpi=150)

print(f'VM = {VM:.6f} V, |gain|_max = {gain_max:.3f}')
```

Run:

```
python3 example_plot.py
```

This will produce `vtc_plot.png`, `vtc_gain.png`, and `tran_plot.png` that you can embed in reports or slides.

---

## Creative visuals & tables you can add to a presentation

- Annotated transient waveform with markers for trise, tfall, tpLH, tpHL.
- Combined plot showing input, output and instantaneous short-circuit current vs time (I(Vdd)).
- Heatmap / parametric plot: propagation delay vs Wp/Wn ratio (sweep widths).
- Table of results when scaling W by ×0.5, ×1, ×2 to demonstrate effect on speed and power.

Example summary table format to fill after parametric sweeps:

| Wp/Wn Ratio | trise (ps) | tfall (ps) | tprop (ps) | Energy per transition (fJ) |
|---:|---:|---:|---:|---:|
| 0.84/0.36 (base) | 45 | 38 | 66 | 0.12 |
| ×0.5 | ... | ... | ... | ... |
| ×2 | ... | ... | ... | ... |

---

## What to try next (experiments & extension ideas)

- Sweep load capacitance and observe trise/tfall & tprop.
- Sweep Wp and Wn (parametric simulations) to find optimal sizing for minimal delay per static power.
- Add realistic interconnect RC loads to estimate IO performance.
- Run Monte Carlo mismatch for threshold variation and plot VM distribution.
- Create an LVS-friendly layout (use available PDK flow) and run post-layout extracted simulations.

---

## Appendix: Example inverter netlist skeleton

(Insert your actual netlist content here — below is a skeleton demonstrating where to add .measure and wrdata for exports.)

```spice
* day3_inv_tran_Wp084_Wn036.spice  (skeleton)
.include sky130.lib

Vdd VDD 0 1.8
* input pulse: 0 -> Vdd
Vin IN 0 PULSE(0 1.8 1n 1n 1n 10n 20n)
M1 OUT IN VDD VDD PMOS W=0.84u L=0.18u
M2 OUT IN 0 0 NMOS W=0.36u L=0.18u

* load cap
CL OUT 0 10f

*.measure statements
.meas tran trise TRIG v(out) VAL=0.1*1.8 TD=0 RISE=1 TARG v(out) VAL=0.9*1.8
.meas tran tfall TRIG v(out) VAL=0.9*1.8 TD=0 FALL=1 TARG v(out) VAL=0.1*1.8
.meas tran tpLH TRIG v(in) VAL=0.5*1.8 TD=0 RISE=1 TARG v(out) VAL=0.5*1.8
.meas tran tpHL TRIG v(in) VAL=0.5*1.8 TD=0 FALL=1 TARG v(out) VAL=0.5*1.8

.tran 0.1n 50n uic
wrdata tran_out.csv time v(in) v(out)
.end
```

---
