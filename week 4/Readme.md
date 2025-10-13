# ⚡ NMOS Drain Current (Id) vs Vds - Complete Guide
## 🎯 Day 1: Basics of NMOS Drain Current & SPICE Simulations

---

<div align="center">

```
╔══════════════════════════════════════════════════════════════╗
║                                                              ║
║    NMOS DRAIN CURRENT CHARACTERISTICS & SPICE ANALYSIS      ║
║                                                              ║
║         SKY130 Circuit Design Workshop - Week 4              ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

</div>

---

## 📚 Table of Contents

```
┌─────────────────────────────────────────────────────────┐
│ 1. 🔍 Why SPICE Simulations?                            │
│ 2. 🔧 Introduction to NMOS                              │
│ 3. ⚡ Strong Inversion & Threshold Voltage              │
│ 4. 📊 Threshold with Positive Substrate                 │
│ 5. 📈 Resistive Region Operation                        │
│ 6. 🎚️ Drift Current Theory                              │
│ 7. 📉 Drain Current Models                              │
│ 8. 🚀 Saturation Region                                 │
│ 9. 🔬 SPICE Simulation Setup                            │
│ 10. 📝 Define Technology Parameters                     │
└─────────────────────────────────────────────────────────┘
```

---

## 🔍 Section 1: Why Do We Need SPICE Simulations?

### 🎯 **Purpose & Importance**

> **SPICE (Simulation Program with Integrated Circuit Emphasis)** is the industry-standard tool for analog circuit simulation.

#### ✨ **Key Benefits:**

![SPICE Simulation Flow](https://www.researchgate.net/publication/224098213/figure/fig1/AS:340823808987136@1458270136021/Flowchart-of-a-SPICE-Simulator-Numbered-blocks-are-calls-to-Matrix-Solver-efficient.png)

```
┌──────────────────────┐
│  CIRCUIT DESIGN      │
│                      │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│  SPICE NETLIST       │
│  (.cir/.sp file)     │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│  SIMULATION ENGINE   │
│  (ngspice/hspice)    │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│  ANALYSIS & PLOTS    │
│  (waveforms/data)    │
└──────────────────────┘
```

| **Aspect** | **What SPICE Provides** | **Why It Matters** |
|------------|-------------------------|-------------------|
| **Prediction** | Pre-fabrication behavior | Saves time & cost |
| **Accuracy** | Device-level physics | Real transistor behavior |
| **Validation** | Design verification | Meets specifications |
| **Analysis** | Delay/Power/Variation | Timing closure |

---

### 💡 **Key Insight Box**

```
╔══════════════════════════════════════════════════════════════╗
║  🎓 LEARNING OBJECTIVE:                                      ║
║                                                              ║
║  SPICE bridges theoretical MOSFET equations with real        ║
║  silicon behavior. It shows what STA (Static Timing         ║
║  Analysis) approximates in timing libraries.                ║
║                                                              ║
║  → Understand transistor delays                             ║
║  → Predict circuit performance                              ║
║  → Analyze process variation impacts                        ║
╚══════════════════════════════════════════════════════════════╝
```

---

## 🔧 Section 2: Introduction to NMOS Transistor

### 📐 **NMOS Structure & Terminals**

![NMOS Transistor Cross Section](https://www.researchgate.net/publication/2985148/figure/fig1/AS:279879644925956@1443739915494/Schematic-cross-section-of-a-NMOS-transistor-a-The-transistor-shown-in-the-schematic.png)

```
                    Gate (G)
                       │
              ┌────────▼────────┐
              │   Gate Oxide    │  ← SiO₂ (insulator)
              │    (thin)       │
   ┌──────────┴─────────────────┴──────────┐
   │                                        │
   │  Source     Channel Region     Drain  │
   │   (S)      (forms when Vgs>Vt)  (D)   │
   │   N⁺  ←─────────────────────→  N⁺    │
   │                                        │
   └────────────────────────────────────────┘
              P-Substrate (Body/Bulk)
                       │
                    Vsub/Vb
```

---

### 🔌 **Terminal Functions**

| Terminal | Symbol | Voltage | Function |
|----------|--------|---------|----------|
| **Gate** | G | V<sub>GS</sub> | Controls channel formation via electric field |
| **Drain** | D | V<sub>DS</sub> | Collector terminal (higher potential) |
| **Source** | S | 0V (ref) | Emitter terminal (lower potential) |
| **Bulk** | B | V<sub>SB</sub> | P-substrate (usually tied to GND/Source) |

---

### ⚙️ **NMOS Operating Principle**

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  WHEN Vgs < Vt:    ❌ NO CHANNEL → Id ≈ 0 (OFF state)     │
│                                                             │
│  WHEN Vgs > Vt:    ✅ CHANNEL FORMS → Id flows (ON state) │
│                                                             │
│  Channel Type:     N-channel (electrons are carriers)      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### 🎯 **Key Characteristics**

> **N-Channel MOSFET**: Requires **positive** gate voltage (relative to source) to turn ON

```
Condition             State        Id Current
─────────────────────────────────────────────
Vgs < Vt              OFF          ≈ 0 A (leakage only)
Vgs = Vt              Threshold    Weak inversion
Vgs > Vt, Vds<Vdsat  LINEAR       ~(Vgs-Vt)·Vds
Vgs > Vt, Vds>Vdsat  SATURATION   ~(Vgs-Vt)²
```

---

## ⚡ Section 3: Strong Inversion & Threshold Voltage

### 🧲 **Channel Formation Process**

![MOSFET Channel Formation](https://miro.medium.com/v2/resize:fit:1400/1*MCqfdDiI8XIH82x1A7gNvQ.png)

```
Step 1: Vgs = 0          Step 2: Vgs < Vt         Step 3: Vgs = Vt
┌──────────┐             ┌──────────┐             ┌──────────┐
│   Gate   │             │   Gate   │             │   Gate   │
├──────────┤             ├──────────┤             ├──────────┤
│  Oxide   │             │  Oxide   │             │  Oxide   │
├──────────┤             ├──────────┤             ├──────────┤
│ P-region │             │ Depletion│             │ Inversion│
│ (holes)  │             │  Region  │             │  Layer   │
└──────────┘             └──────────┘             └──────────┘
   No field          Holes repelled         N-channel forms
```

---

### 📐 **Threshold Voltage Equation**

```
╔═══════════════════════════════════════════════════════════════╗
║                                                               ║
║   Vₜ = Vₜ₀ + γ(√|2φF + Vsb| - √|2φF|)                      ║
║                                                               ║
║   Where:                                                      ║
║   • Vₜ₀ = Threshold voltage at Vsb = 0                      ║
║   • γ   = Body effect coefficient (√V)                       ║
║   • φF  = Fermi potential (≈ 0.3V for typical doping)       ║
║   • Vsb = Source-to-body voltage (body effect)               ║
║                                                               ║
╚═══════════════════════════════════════════════════════════════╝
```

---

### 📊 **Id vs Vgs Characteristic**

![Id vs Vgs Curve](https://i.sstatic.net/frVcb.png)

```
Id (A)
  │
  │                               ╱────────  Strong Inversion
  │                           ╱──
  │                       ╱───     (Quadratic in sat region)
  │                   ╱───
  │               ╱───              Weak Inversion
  │           ╱───                  (Subthreshold)
  │      ╱────
  │  ╱───
  │──────────────┼──────────────────────────> Vgs (V)
  0              Vt
                  │
            Threshold Point
```

---

### 🎚️ **Inversion Regimes**

| Region | Condition | Id Behavior | Channel State |
|--------|-----------|-------------|---------------|
| **Cut-off** | Vgs < Vt | Id ≈ 0 (leakage ~pA-nA) | No channel |
| **Weak Inversion** | Vgs ≈ Vt | Id ∝ exp(Vgs/nVT) | Partial channel |
| **Strong Inversion** | Vgs >> Vt | Id ∝ (Vgs-Vt)² or (Vgs-Vt)Vds | Full channel |

---

### 💎 **Key Takeaway**

```
┌───────────────────────────────────────────────────────────┐
│                                                           │
│  ⚡ STRONG INVERSION is the normal operating region      │
│                                                           │
│     Vgs > Vt  →  N-channel fully formed                  │
│               →  High carrier concentration              │
│               →  Predictable Id-Vds characteristics      │
│                                                           │
└───────────────────────────────────────────────────────────┘
```

---

## 📊 Section 4: Threshold Voltage with Positive Substrate Potential

### 🔬 **Body Effect (Back-Gate Effect)**

> When source-to-body voltage (Vsb) is **non-zero**, the threshold voltage **increases**

---

### 📈 **Vt vs Vsb Relationship**

![Vt vs Vsb Graph](https://www.researchgate.net/publication/275954734/figure/fig5/AS:649632954712100@1531895974322/th-Vs-Vsb-With-NMOS-Body-Biasing-D-CMOS-with-Body-Voltage-Positive-body-voltage-of-tend.png)

```
Vt (V)
  │
  │     ╱────────────
  │    ╱
  │   ╱              Vt increases with Vsb
  │  ╱               (square root relationship)
  │ ╱
  │╱
  ├─────────────────────────────────> Vsb (V)
 Vt0                (Body Effect)
  │
  └─ Vt0 = threshold at Vsb=0
```

---

### 🧮 **Body Effect Formula**

```
╔═══════════════════════════════════════════════════════════════╗
║                                                               ║
║   ΔVt = γ(√|2φF + Vsb| - √|2φF|)                            ║
║                                                               ║
║   Typical Values (SKY130):                                    ║
║   • γ ≈ 0.4 - 0.6 √V                                         ║
║   • φF ≈ 0.3 V                                               ║
║   • For Vsb = 1V → ΔVt ≈ 100-200 mV                         ║
║                                                               ║
╚═══════════════════════════════════════════════════════════════╝
```

---

### 📋 **Body Effect Impact Table**

| Vsb (V) | ΔVt (mV) | Effective Vt | Impact |
|---------|----------|--------------|--------|
| 0.0 | 0 | Vt0 | Standard operation |
| 0.5 | ~70 | Vt0 + 70mV | Stacked transistors |
| 1.0 | ~130 | Vt0 + 130mV | Series NMOS chains |
| 2.0 | ~240 | Vt0 + 240mV | Significant degradation |

---

### 🎯 **Practical Applications**

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│  🔌 NAND Gate Stack:                                   │
│                                                         │
│         Out                                            │
│          │                                             │
│      ┌───┴───┐        ← Top NMOS: Vsb > 0            │
│   A──┤ N1    │           → Higher Vt                  │
│      └───┬───┘           → Slower switching           │
│      ┌───┴───┐        ← Bottom NMOS: Vsb = 0         │
│   B──┤ N2    │           → Normal Vt                  │
│      └───┬───┘                                        │
│          GND                                          │
│                                                        │
│  Impact: Top transistor is weaker due to body effect  │
│                                                        │
└─────────────────────────────────────────────────────────┘
```

---

### ⚠️ **Design Considerations**

```
╔══════════════════════════════════════════════════════════╗
║  📌 CRITICAL INSIGHT:                                    ║
║                                                          ║
║  Body effect degrades drive strength in:                ║
║  • Stacked transistors (NAND/NOR gates)                 ║
║  • Pass transistor logic                                ║
║  • Transmission gates                                   ║
║                                                          ║
║  Solution: Size upper transistors larger to compensate  ║
╚══════════════════════════════════════════════════════════╝
```

---

## 📈 Section 5: Resistive (Linear) Region of Operation

### 🔌 **Linear Region Basics**

> **Condition**: Vgs > Vt **AND** Vds < (Vgs - Vt)

```
Channel Condition:
┌─────────────────────────────────────┐
│  Source ═══════════════════ Drain   │  ← Continuous channel
│          N-channel                  │
│                                     │
└─────────────────────────────────────┘
      Uniform channel depth
   (small Vds doesn't pinch off)
```

---

### 📐 **Drain Current Equation - Linear Region**

```
╔═══════════════════════════════════════════════════════════════╗
║                                                               ║
║   Id = μn·Cox·(W/L)·[(Vgs - Vt)·Vds - Vds²/2]              ║
║                                                               ║
║   For SMALL Vds (Vds << Vgs-Vt):                            ║
║                                                               ║
║   Id ≈ μn·Cox·(W/L)·(Vgs - Vt)·Vds                          ║
║                                                               ║
║   → Behaves like a RESISTOR controlled by Vgs                ║
║                                                               ║
╚═══════════════════════════════════════════════════════════════╝
```

---

### 🔧 **Parameters Explained**

| Parameter | Symbol | Units | Typical Value (SKY130) |
|-----------|--------|-------|------------------------|
| Electron mobility | μn | cm²/V·s | 400-500 |
| Gate oxide capacitance | Cox | F/cm² | 5-10 fF/μm² |
| Width | W | μm | 0.42 - 10 |
| Length | L | μm | 0.15 (min) |
| Threshold voltage | Vt | V | 0.4-0.7 |

---

### 📊 **Id vs Vds Curves (Multiple Vgs)**

![Id vs Vds Family of Curves](https://www.researchgate.net/publication/330268528/figure/fig8/AS:961701608427545@1606298939658/The-family-of-curves-of-ID-VDS-for-different-values-of-VGS-The-solid-lines-correspond-to.png)

```
Id (mA)
  │
10│                                  Vgs = 2.5V ────────────
  │                              ╱────
  │                          ╱────
 8│                      ╱────         Vgs = 2.0V ─────────
  │                  ╱────         ╱────
  │              ╱────         ╱────
 6│          ╱────         ╱────            Vgs = 1.5V ────
  │      ╱────         ╱────            ╱────
  │  ╱────         ╱────            ╱────
 4│────         ╱────            ╱────
  │         ╱────            ╱────
  │     ╱────            ╱────
 2│ ╱────            ╱────
  │────         ╱────
  │        ╱────
 0├─────┬──────┬──────┬──────┬──────┬──────────> Vds (V)
  0    0.5    1.0    1.5    2.0    2.5
       │              │
    Linear        Saturation
    Region         Region
```

---

### 🎚️ **Resistive Behavior**

```
For small Vds:

Rds_on ≈ 1 / [μn·Cox·(W/L)·(Vgs - Vt)]

┌────────────────────────────────────────┐
│                                        │
│  Higher Vgs → Lower Rds_on            │
│  Larger W/L → Lower Rds_on            │
│                                        │
│  This is why we use Vgs=VDD           │
│  for pass transistors!                 │
│                                        │
└────────────────────────────────────────┘
```

---

### ⚡ **Key Applications**

```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  📱 LINEAR REGION APPLICATIONS:                         │
│                                                          │
│  1. Analog Switches                                     │
│     → Low on-resistance                                 │
│                                                          │
│  2. Transmission Gates                                  │
│     → Bidirectional signal passing                      │
│                                                          │
│  3. Source Followers                                    │
│     → Buffer circuits                                   │
│                                                          │
│  4. Resistive Loads                                     │
│     → Variable resistors in analog design               │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## 🎚️ Section 6: Drift Current Theory

### 🌊 **Drift Current Fundamentals**

> Drift current is the flow of charge carriers due to an **electric field**

---

### 📐 **Drift Current Equation**

```
╔═══════════════════════════════════════════════════════════════╗
║                                                               ║
║   Idrift = -n·q·μn·A·(dV/dx)                                ║
║                                                               ║
║   Where:                                                      ║
║   • n  = Electron concentration (carriers/cm³)               ║
║   • q  = Electron charge (1.6×10⁻¹⁹ C)                      ║
║   • μn = Electron mobility (cm²/V·s)                         ║
║   • A  = Cross-sectional area (W × channel depth)            ║
║   • dV/dx = Electric field along channel                     ║
║                                                               ║
╚═══════════════════════════════════════════════════════════════╝
```

---

### ⚡ **Physical Picture**

```
Source (x=0)                             Drain (x=L)
    │                                         │
    │  E-field →→→→→→→→→→→→→→→→→→→→         │
    │                                         │
    ▼  e⁻ ← ← ← ← ← ← ← ← ← ← ← ← ← ←        ▼
    
    • Electric field: E = -dV/dx
    • Electrons drift opposite to E-field
    • Drift velocity: vd = μn·E
    • Current: Id = n·q·vd·A
```

---

### 🔬 **Mobility (μn)**

| Factor | Effect on Mobility | Reason |
|--------|-------------------|--------|
| **Temperature ↑** | μn ↓ | Increased phonon scattering |
| **Doping ↑** | μn ↓ | Increased impurity scattering |
| **E-field ↑** | μn ↓ | Velocity saturation |
| **Surface roughness** | μn ↓ | Interface scattering |

---

### 📊 **Velocity Saturation**

![Velocity Saturation Graph](https://circuitstoday.com/wp-content/uploads/2010/09/electron-velocity-field.jpg)

```
Drift Velocity (cm/s)
  │
  │         ╱───────────────  vsat ≈ 10⁷ cm/s
  │        ╱
  │       ╱   Velocity saturation
  │      ╱    (short channel)
  │     ╱
  │    ╱  Linear (long channel)
  │   ╱   vd = μn·E
  │  ╱
  │ ╱
  ├──────────────────────────────────> E-field (V/cm)
  0                                    Ecrit ≈ 1-2×10⁴
  
  Impact: Current doesn't increase linearly with Vds
```

---

## 📉 Section 7: Drain Current Model for Linear Region

### 🧮 **Deriving the Drain Current**

```
Step-by-step derivation:

1. Start with drift current at position x:
   I(x) = -Qinv(x)·μn·W·(dV/dx)

2. Charge in inversion layer:
   Qinv(x) = -Cox·[Vgs - Vt - V(x)]

3. Substitute:
   Id = μn·Cox·W·[Vgs - Vt - V(x)]·(dV/dx)

4. Integrate from source (x=0, V=0) to drain (x=L, V=Vds):
   Id·∫₀ᴸ dx = μn·Cox·W·∫₀^Vds [Vgs - Vt - V]·dV

5. Result:
   Id = μn·Cox·(W/L)·[(Vgs - Vt)·Vds - Vds²/2]
```

---

### 📐 **Complete Linear Region Equation**

```
╔═══════════════════════════════════════════════════════════════╗
║                                                               ║
║   Id,lin = μn·Cox·(W/L)·[(Vgs - Vt)·Vds - Vds²/2]          ║
║                                                               ║
║   Valid for: Vgs > Vt  AND  Vds < Vdsat = (Vgs - Vt)       ║
║                                                               ║
║   Simplified (small Vds):                                     ║
║   Id,lin ≈ μn·Cox·(W/L)·(Vgs - Vt)·Vds                      ║
║                                                               ║
╚═══════════════════════════════════════════════════════════════╝
```

---

### 🎯 **Design Parameters Impact**

```
Parameter      Change    Id Impact
─────────────────────────────────────────
W (width)       2×       Id × 2  (linear)
L (length)      2×       Id / 2  (inverse)
Vgs             ↑        Id ↑    (linear with Vds)
Vds (small)     ↑        Id ↑    (linear)
Vds (large)     ↑        Id ↑    (quadratic decay)
```

---

## 🚀 Section 8: Saturation Region Operation

### 🔥 **Saturation Region Basics**

> **Condition**: Vgs > Vt **AND** Vds ≥ Vdsat = (Vgs - Vt)

---

### 🎭 **Channel Pinch-Off**

![Channel Pinch-Off Diagram](https://www.allaboutcircuits.com/uploads/articles/TB_MCLD_1.JPG)

```
Linear Region:              Saturation Region:
┌──────────────────┐       ┌──────────────────┐
│ Uniform channel  │       │  Pinched channel │
│═════════════════>│       │══════════╱      │
│                  │       │         ╱ Pinch │
└──────────────────┘       └────────╱────────┘
  Source    Drain            Source  ↑  Drain
                                   Pinch
                                   Point
  
When Vds increases beyond Vdsat, 
the channel pinches off near the drain
```

---

### 📐 **Saturation Current Equation**

```
╔═══════════════════════════════════════════════════════════════╗
║                                                               ║
║   Id,sat = (1/2)·μn·Cox·(W/L)·(Vgs - Vt)²                   ║
║                                                               ║
║   Or with velocity saturation:                                ║
║                                                               ║
║   Id,sat = W·Cox·vsat·(Vgs - Vt)                            ║
║                                                               ║
║   Key property: Id is INDEPENDENT of Vds (ideal case)        ║
║                                                               ║
╚═══════════════════════════════════════════════════════════════╝
```

---

### 📊 **Complete Id-Vds Characteristics**

![Id-Vds Characteristics](https://www.researchgate.net/publication/330268528/figure/fig8/AS:961701608427545@1606298939658/The-family-of-curves-of-ID-VDS-for-different-values-of-VGS-The-solid-lines-correspond-to.png)

```
Id (mA)
  │
  │                     ┌─────────────── Vgs = 2.5V
  │                    ╱│
  │                 ╱─  │ Saturation (Id constant)
  │              ╱──    │
  │          ╱───┌──────────────── Vgs = 2.0V
  │       ╱──   ╱│
  │    ╱──   ╱── │
  │ ╱──  ╱───┌──────────────── Vgs = 1.5V
  │──╱────  ╱│
  │──────╱───│
  │─────────╱│
  ├──────┬───┴──────────────────────> Vds (V)
  0    Vdsat
       │
  Linear│Saturation
  Region│ Region
```

---

### 🎯 **Saturation Point**

```
┌────────────────────────────────────────────────────┐
│                                                    │
│  Vdsat = Vgs - Vt                                 │
│                                                    │
│  At this point:                                   │
│  • Channel depth → 0 at drain end                 │
│  • Further ↑Vds doesn't ↑Id (ideal)              │
│  • Transistor acts as current source              │
│                                                    │
└────────────────────────────────────────────────────┘
```

---

### ⚙️ **Key Characteristics**

| Property | Linear Region | Saturation Region |
|----------|---------------|-------------------|
| **Condition** | Vds < Vgs-Vt | Vds ≥ Vgs-Vt |
| **Id vs Vds** | Linear then quadratic | Constant (ideally) |
| **Id vs Vgs** | Linear (small Vds) | Quadratic |
| **Channel** | Continuous | Pinched at drain |
| **Output R** | Low | High (current source) |
| **Use Case** | Switches, resistors | Amplifiers, current sources |

---

### 🔬 **Non-Ideal Effects**

```
Real Saturation Current:

Id,sat = (1/2)·μn·Cox·(W/L)·(Vgs - Vt)²·(1 + λ·Vds)
                                           ↑
                                Channel Length Modulation
                                
Where λ = 1/(L·VA)  and  VA = early voltage

┌────────────────────────────────────────┐
│                                        │
│  Channel length modulation causes      │
│  slight ↑Id with ↑Vds in saturation   │
│                                        │
│  Shorter L → Larger λ → More effect   │
│                                        │
└────────────────────────────────────────┘
```

---

### 💡 **Design Applications**

```
╔══════════════════════════════════════════════════════════╗
║  🎯 SATURATION REGION APPLICATIONS:                      ║
║                                                          ║
║  ✓ Analog Amplifiers (high gain)                        ║
║  ✓ Current Mirrors (precise current copy)               ║
║  ✓ Active Loads (high impedance)                        ║
║  ✓ Differential Pairs (input stage)                     ║
║  ✓ Digital Logic (switching, high/low states)           ║
║                                                          ║
╚══════════════════════════════════════════════════════════╝
```

---

## 🔬 Section 9: Introduction to SPICE

### 🎯 **What is SPICE?**

```
╔══════════════════════════════════════════════════════════════╗
║                                                              ║
║   SPICE = Simulation Program with Integrated Circuit        ║
║           Emphasis                                           ║
║                                                              ║
║   Originally developed at UC Berkeley (1973)                ║
║   Industry standard for circuit simulation                  ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

---

### 🛠️ **SPICE Variants**

| Variant | Type | Features | Use Case |
|---------|------|----------|----------|
| **ngspice** | Open-source | Free, cross-platform | Learning, research |
| **HSPICE** | Commercial | Fast, accurate | Industry standard |
| **LTspice** | Free | User-friendly GUI | Analog design |
| **Spectre** | Commercial | Advanced models | Cadence flow |

---

### 📝 **SPICE Netlist Structure**

```
┌─────────────────────────────────────────────────┐
│  SPICE NETLIST COMPONENTS:                      │
│                                                 │
│  1. Title Line (first line, comment)           │
│  2. Element Statements (devices)                │
│  3. Model Statements (.model)                   │
│  4. Source Specifications (V, I)                │
│  5. Analysis Commands (.dc, .tran, .ac)         │
│  6. Output Commands (.print, .plot)             │
│  7. Control Statements (.end)                   │
│                                                 │
└─────────────────────────────────────────────────┘
```

---

### 📋 **Basic SPICE Netlist Example**

```spice
* NMOS Characteristic Simulation
* Title line (always first)

* MOSFET Declaration
M1 drain gate source bulk NMOS_MODEL W=1u L=0.18u

* Model Definition
.model NMOS_MODEL nmos level=1 vto=0.7 kp=110u

* Voltage Sources
Vgs gate 0 DC 2.5
Vds drain 0 DC 0

* Analysis Command
.dc Vds 0 3 0.01 Vgs 0 3 0.5

* Output
.print dc v(drain) i(Vds)

.end
```

---

### 🔧 **MOSFET Element Syntax**

```
╔═══════════════════════════════════════════════════════════════╗
║                                                               ║
║   M<name> <drain> <gate> <source> <bulk> <model> [params]   ║
║                                                               ║
║   Example:                                                    ║
║   M1 2 1 0 0 NMOS W=10u L=0.5u                              ║
║       │ │ │ │   │    └─────┴──── Parameters                 ║
║       │ │ │ │   └─────────────── Model name                 ║
║       │ │ │ └─────────────────── Bulk node                  ║
║       │ │ └───────────────────── Source node                ║
║       │ └─────────────────────── Gate node                  ║
║       └───────────────────────── Drain node                 ║
║                                                               ║
╚═══════════════════════════════════════════════════════════════╝
```

---

### ⚡ **Analysis Types**

```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  📊 DC ANALYSIS (.dc)                                   │
│     → Sweeps DC voltages/currents                       │
│     → Gets Id-Vds, Id-Vgs curves                        │
│                                                          │
│  ⏱️  TRANSIENT ANALYSIS (.tran)                         │
│     → Time-domain simulation                            │
│     → Rise/fall times, delays                           │
│                                                          │
│  〰️  AC ANALYSIS (.ac)                                  │
│     → Frequency response                                │
│     → Gain, bandwidth                                   │
│                                                          │
│  🎯 OPERATING POINT (.op)                               │
│     → DC steady-state solution                          │
│     → Bias points                                       │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

### 📊 **DC Sweep Example**

```spice
* Id vs Vds for different Vgs values

.dc Vds 0 3 0.01 Vgs 0.5 2.5 0.5
     │   │ │  │    │   │   │   │
     │   │ │  │    │   │   │   └─ Step for outer sweep
     │   │ │  │    │   │   └───── Stop value (outer)
     │   │ │  │    │   └─────────  Start value (outer)
     │   │ │  │    └──────────────  Outer sweep variable
     │   │ │  └────────────────────  Step increment
     │   │ └───────────────────────  Stop value
     │   └────────────────────────── Start value
     └─────────────────────────────── Sweep variable

Result: Multiple curves (one for each Vgs)
```

---

### 💻 **Running SPICE Simulation**

```bash
# Command line (ngspice)
$ ngspice nmos_char.cir

# Or interactive mode
$ ngspice
ngspice 1 -> source nmos_char.cir
ngspice 2 -> run
ngspice 3 -> plot i(vds)
ngspice 4 -> quit

# Batch mode with output
$ ngspice -b nmos_char.cir -o output.txt
```

---

### 📈 **Plotting Results**

```
In SPICE:
plot i(vds)              → Current vs default x-axis
plot vds#branch          → Alternative current syntax
plot v(drain)            → Voltage at node 'drain'
plot i(vds) vs v(drain)  → Custom x-axis

In Python (post-processing):
import matplotlib.pyplot as plt
import numpy as np

# Read SPICE output
data = np.loadtxt('output.txt')
vds = data[:,0]
id = data[:,1]

plt.plot(vds, id*1e3)
plt.xlabel('Vds (V)')
plt.ylabel('Id (mA)')
plt.grid(True)
plt.show()
```

---

## 📝 Section 10: Define Technology Parameters

### 🎯 **SKY130 Technology Overview**

```
╔══════════════════════════════════════════════════════════════╗
║                                                              ║
║   SKY130 = SkyWater 130nm Open-Source PDK                   ║
║                                                              ║
║   Key Features:                                             ║
║   • 130nm CMOS process                                      ║
║   • Open-source PDK (Process Design Kit)                    ║
║   • Supports analog, digital, and mixed-signal designs      ║
║   • Available through SkyWater Foundry                     ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```
