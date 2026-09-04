# PWM-Controlled Voltage-Controlled Current Source (VCCS)

## 1. Project Overview

This repository contains the LTspice design and simulation work for a PWM-controlled Voltage-Controlled Current Source (VCCS) developed for the Electronics Design Engineer assignment.

The objective of the design is to convert a PWM voltage command into a controlled, monophasic output current of 0–5 mA while maintaining the commanded current over a representative load range.

The design was implemented and tested in LTspice using a low-side, op-amp-controlled NMOS current-sink topology.

---

## 2. Assignment Requirements

| Parameter | Requirement |
|---|---:|
| Input signal | PWM voltage |
| Input voltage | 0–5 V |
| Frequency | 1–200 Hz |
| Pulse width | 20–500 µs |
| Output current | 0–5 mA |
| Transconductance | 1 mA/V |
| Current direction | Monophasic / unidirectional |
| Load impedance | 1–10 kΩ |
| Maximum circuit voltage | 70 V |
| Target current accuracy | ±10% |

Required nominal input/output relationship:

```text
0 V → 0 mA
1 V → 1 mA
2 V → 2 mA
3 V → 3 mA
4 V → 4 mA
5 V → 5 mA
```

The assignment also requests LTspice/equivalent simulation evidence for current versus load impedance, frequency response, and current accuracy.

---

## 3. Selected Circuit Topology

### Low-Side Op-Amp Controlled NMOS Current Sink

The selected topology uses an operational amplifier to control an NMOS transistor while sensing the voltage across a source resistor.

Basic current path:

```text
              +60 V
                |
              RLOAD
                |
                D
               M1
                S
                |
              SENSE
                |
          RSENSE = 1 kΩ
                |
               GND
```

The op-amp compares the PWM command with the SENSE voltage and drives the MOSFET gate through negative feedback.

The intended relationship is:

```text
V(SENSE) ≈ V(PWM)
```

Therefore:

```text
I_OUT = V(SENSE) / R_SENSE
```

With:

```text
R_SENSE = 1 kΩ
```

the nominal transconductance is:

```text
I_OUT ≈ V_PWM / 1 kΩ
```

or:

```text
1 mA/V
```

---

## 4. Complete Circuit Connections

```text
+60 V supply
    |
  RLOAD
    |
M1 Drain

M1 Source
    |
    +----------------→ U1 inverting input (−)
    |
  RSENSE = 1 kΩ
    |
   GND

PWM source (+)
    |
R_IN = 1 kΩ
    |
    +----------------→ U1 non-inverting input (+)

PWM source (−)
    |
   GND

U1 output
    |
R_G = 100 Ω
    |
M1 Gate
    |
R3 = 100 kΩ
    |
   GND

U1 V+
    |
+15 V

U1 V−
    |
   GND
```

All circuit grounds share the same reference.

---

## 5. Component Values

| Reference | Component | Value / Model |
|---|---|---|
| V_HV | High-voltage supply | +60 V |
| V_CC | Op-amp supply | +15 V |
| U1 | Operational amplifier | UniversalOpamp2 |
| M1 | NMOS | MYNMOS |
| RLOAD | Load resistor | `{Rload}` |
| RSENSE | Current-sense resistor | 1 kΩ |
| R_IN | PWM input resistor | 1 kΩ |
| R_G | Gate resistor | 100 Ω |
| R3 | Gate pull-down resistor | 100 kΩ |

MOSFET simulation model:

```spice
.model MYNMOS NMOS(LEVEL=1 VTO=2 KP=0.01 LAMBDA=0.005)
```

---

## 6. Design Calculations

### 6.1 Current-Sense Resistor

The feedback loop is designed to force:

```text
V(SENSE) ≈ V(PWM)
```

The output current is determined by the sense resistor:

```text
I_OUT = V(SENSE) / R_SENSE
```

For the required 1 mA/V relationship:

```text
R_SENSE = 1 V / 1 mA
        = 1 kΩ
```

At the maximum command:

```text
I_OUT = 5 V / 1 kΩ
      = 5 mA
```

### 6.2 Sense Resistor Dissipation

At 5 mA:

```text
P = I²R
  = (5 mA)² × 1 kΩ
  = 25 mW
```

### 6.3 High-Voltage Supply and Compliance

Worst-case load:

```text
I_MAX = 5 mA
R_LOAD(MAX) = 10 kΩ
```

Load voltage:

```text
V_LOAD = I × R
       = 5 mA × 10 kΩ
       = 50 V
```

Sense resistor voltage:

```text
V_SENSE = 5 mA × 1 kΩ
        = 5 V
```

Allowing approximately 5 V for MOSFET headroom gives:

```text
V_HV ≈ 50 V + 5 V + 5 V
     ≈ 60 V
```

The simulation therefore uses:

```text
V_HV = +60 V
```

which is below the 70 V maximum circuit-voltage requirement.

### 6.4 MOSFET Voltage Requirement

When the MOSFET is off, its drain can rise toward the +60 V rail. A 100 V device class is therefore targeted for a practical implementation.

The LTspice simulation uses the self-contained `MYNMOS` model defined in the schematic.

---

## 7. LTspice Simulation Setup

### 7.1 PWM Source

The PWM input is implemented using a standard LTspice voltage source configured as:

```spice
PULSE(0 {Vamp} 0 100n 100n {pw} {per})
```

Base parameters:

```spice
.param Vamp=5
.param pw=200u
.param per=20m
.param Rload=5k
```

Baseline condition:

```text
PWM amplitude = 5 V
Pulse width   = 200 µs
Period        = 20 ms
Frequency     = 50 Hz
Load          = 5 kΩ
```

### 7.2 Transient Analysis

Baseline transient command:

```spice
.tran 0 60m 0 20n
```

### 7.3 Current-Level Sweep

The current-command sweep is defined by:

```spice
.step param Vamp list 1 2 3 4 5
```

The nominal target levels are:

| PWM amplitude | Target current |
|---:|---:|
| 1 V | 1 mA |
| 2 V | 2 mA |
| 3 V | 3 mA |
| 4 V | 4 mA |
| 5 V | 5 mA |

### 7.4 Load Sweep

The load sweep is defined by:

```spice
.step param Rload list 1k 5k 10k
```

This evaluates the current-source behavior at the representative load values required by the assignment.

---

## 8. Current Measurement

The primary output-current probe in LTspice is:

```text
I(Rsense)
```

The SENSE-node voltage can also be monitored:

```text
V(SENSE)
```

The current-sense relationship is:

```text
I_OUT = V(SENSE) / 1 kΩ
```

---

## 9. Baseline Simulation Status

A working baseline simulation was run with:

```text
Vamp        = 5 V
Rload       = 5 kΩ
Frequency   = 50 Hz
Pulse width = 200 µs
```

Observed baseline behavior:

- The PWM source generates the expected pulse waveform.
- The SENSE node rises to approximately 5 V during the active pulse.
- `I(Rsense)` rises to approximately 5 mA.
- The current returns to approximately 0 mA between pulses.
- The current pulse has a flat top over the active interval.
- No pronounced sustained ringing or flat-top droop was observed in the baseline waveform.

This demonstrates the intended nominal 5 mA current-sink behavior for the tested baseline operating point.

---

## 10. Load Sweep Status

A load sweep was performed using:

```spice
.step param Rload list 1k 5k 10k
```

At the 5 mA command, the current traces for the tested loads overlap closely.

Tested loads:

```text
1 kΩ
5 kΩ
10 kΩ
```

The observed behavior is consistent with the intended current-source characteristic: the output current remains approximately constant while the load resistance changes across the tested values.

---

## 11. Current-Level Sweep Status

A PWM-amplitude sweep was performed using:

```spice
.step param Vamp list 1 2 3 4 5
```

This evaluates the nominal current-command range:

```text
1 mA
2 mA
3 mA
4 mA
5 mA
```

The current is evaluated using:

```text
I(Rsense)
```

The simulation outputs should be interpreted from the actual LTspice traces rather than from the nominal equations alone.

---

## 12. Simulation and Schematic Images

### Schematic

The `Schematic/` directory contains the schematic screenshots captured during the design process:

```text
Schematic/
├── sch_1.png
├── sch_2.png
├── sch_3.png
├── sch_4.png
├── sch_5.png
├── sch_6.png
├── sch_7.png
└── sch_8.png
```

These images document the development of the LTspice schematic and the final circuit arrangement.

### Simulation Results

The `Results/` directory contains the LTspice waveform screenshots generated during the simulation and verification process:

```text
Results/
├── res_1.png
├── res_2.png
├── res_3.png
├── res_4.png
├── res_5.png
├── res_6.png
├── res_7.png
├── res_8.png
├── res_9.png
├── res_10.png
├── res_11.png
├── res_12.png
├── res_13.png
├── res_14.png
├── res_15.png
├── res_16.png
├── res_17.png
├── res_18.png
├── res_19.png
├── res_20.png
├── res_21.png
├── res_22.png
└── res_final.png
```

The numbered screenshots preserve the simulation and verification history. `res_final.png` contains the final selected simulation result.

---

## 13. Repository Structure

```text
VCCS/
├── LICENSE
├── README.md
│
├── LTspice/
│   └── VCCS.asc
│
├── Results/
│   ├── res_1.png
│   ├── res_2.png
│   ├── res_3.png
│   ├── res_4.png
│   ├── res_5.png
│   ├── res_6.png
│   ├── res_7.png
│   ├── res_8.png
│   ├── res_9.png
│   ├── res_10.png
│   ├── res_11.png
│   ├── res_12.png
│   ├── res_13.png
│   ├── res_14.png
│   ├── res_15.png
│   ├── res_16.png
│   ├── res_17.png
│   ├── res_18.png
│   ├── res_19.png
│   ├── res_20.png
│   ├── res_21.png
│   ├── res_22.png
│   └── res_final.png
│
└── Schematic/
    ├── sch_1.png
    ├── sch_2.png
    ├── sch_3.png
    ├── sch_4.png
    ├── sch_5.png
    ├── sch_6.png
    ├── sch_7.png
    └── sch_8.png
```

---

## 14. Reproducing the Baseline Simulation

1. Open `LTspice/VCCS.asc`.
2. Confirm the MOSFET value is `MYNMOS`.
3. Confirm the PWM source is:

```spice
PULSE(0 {Vamp} 0 100n 100n {pw} {per})
```

4. Confirm the parameters are:

```spice
.param Vamp=5
.param pw=200u
.param per=20m
.param Rload=5k
```

5. Confirm the MOSFET model is:

```spice
.model MYNMOS NMOS(LEVEL=1 VTO=2 KP=0.01 LAMBDA=0.005)
```

6. Confirm the transient command is:

```spice
.tran 0 60m 0 20n
```

7. Run the simulation.
8. Plot `I(Rsense)` to observe output current.
9. Plot `V(SENSE)` to observe the feedback voltage.

Expected nominal baseline behavior:

```text
≈5 mA during the active PWM interval
≈0 mA during the inactive interval
```

---

## 15. Current Project Status

### Completed / demonstrated

- LTspice VCCS schematic created.
- Low-side op-amp/NMOS current-sink topology implemented.
- +60 V high-voltage rail implemented.
- +15 V single-supply op-amp rail implemented.
- PWM voltage source implemented.
- 1 kΩ current-sense resistor implemented.
- 100 Ω gate resistor implemented.
- 100 kΩ gate pull-down implemented.
- Baseline 5 mA current waveform demonstrated.
- Baseline 5 kΩ load demonstrated.
- Baseline 50 Hz / 200 µs pulse demonstrated.
- Load sweep across 1 kΩ, 5 kΩ, and 10 kΩ performed.
- PWM-amplitude/current-level sweep performed.
- Schematic and simulation screenshots collected.

### Verification status

The design and simulations demonstrate the intended current-source behavior for the operating conditions that were actually run.

A complete claim of ±10% accuracy across every combination of current, load, frequency, and pulse width should only be made after numerical measurements have been completed for those combinations.

Similarly, complete verification of the full 1–200 Hz and 20–500 µs operating ranges requires the corresponding boundary-condition simulations and measurements.

The repository therefore distinguishes between the design target and the operating conditions actually demonstrated by simulation.

---

## 16. Safety and Scope

This project is an educational simulation for the supplied electronics assignment.

The circuit is not intended to be connected to a person or used as a medical device.

The LTspice circuit, model, and simulation results are provided for educational and engineering evaluation only and do not constitute medical-device validation, certification, or safety approval.

---

## 17. License

This project is released under the MIT License. See the `LICENSE` file for details.
