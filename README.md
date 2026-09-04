# PWM-Controlled Voltage-Controlled Current Source (VCCS)

## 1. Project Overview

This repository contains the LTspice design and simulation work for a PWM-controlled voltage-controlled current source (VCCS).

The circuit converts a PWM voltage input into a controlled, unidirectional current pulse using an operational amplifier, an NMOS transistor, and a source current-sense resistor in a low-side current-sink topology.

This project was developed for the Electronics Design Engineer VCCS assignment.

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
| Load | 1–10 kΩ |
| Maximum circuit voltage | 70 V |
| Target current accuracy | ±10% |

Required current mapping:

```text
0 V → 0 mA
1 V → 1 mA
2 V → 2 mA
3 V → 3 mA
4 V → 4 mA
5 V → 5 mA
```

The assignment also requests LTspice/equivalent evidence for current versus load impedance, frequency response, and current accuracy.

---

## 3. Chosen Circuit Topology

### Low-Side Op-Amp Controlled NMOS Current Sink

The selected topology is:

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

The op-amp senses the voltage across `RSENSE` and drives the NMOS gate so that the SENSE voltage follows the PWM command.

Approximately:

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

the intended transconductance is:

```text
I_OUT ≈ V_PWM / 1 kΩ
```

or approximately:

```text
1 mA/V
```

---

## 4. Circuit Connections

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

All circuit returns use the same ground reference.

---

## 5. Component Values

| Reference | Component | Value / Model |
|---|---|---|
| V_HV | High-voltage supply | +60 V |
| V_CC | Op-amp supply | +15 V |
| U1 | Op-amp | UniversalOpamp2 |
| M1 | NMOS | MYNMOS |
| RLOAD | Load | `{Rload}` |
| RSENSE | Current-sense resistor | 1 kΩ |
| R_IN | PWM input resistor | 1 kΩ |
| R_G | Gate resistor | 100 Ω |
| R3 | Gate pull-down | 100 kΩ |

MOSFET model currently used in the LTspice schematic:

```spice
.model MYNMOS NMOS(LEVEL=1 VTO=2 KP=0.01 LAMBDA=0.005)
```

---

## 6. Design Calculations

### 6.1 Current-Sense Resistor

The feedback loop is intended to force:

```text
V(SENSE) = V(PWM)
```

For a resistive current-sense element:

```text
I_OUT = V(SENSE) / R_SENSE
```

For the required 1 mA/V relationship:

```text
R_SENSE = 1 V / 1 mA
        = 1 kΩ
```

Thus the nominal current relationship is:

```text
I_OUT(mA) ≈ V_PWM(V)
```

At 5 V:

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

With approximately 5 V additional MOSFET headroom:

```text
V_HV ≈ 50 V + 5 V + 5 V
     ≈ 60 V
```

The selected simulation rail is therefore:

```text
V_HV = 60 V
```

which is below the assignment's 70 V maximum circuit-voltage requirement.

### 6.4 MOSFET Voltage Requirement

When the MOSFET is off, its drain can rise toward the +60 V supply. The simulation therefore uses a model with a 100 V breakdown target:

```text
BV = 100 V
```

for voltage-margin assessment.

---

## 7. LTspice Configuration

### 7.1 PWM Source

The PWM source is a standard LTspice voltage source using:

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

The baseline condition is therefore:

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

The current-level sweep is defined by:

```spice
.step param Vamp list 1 2 3 4 5
```

This corresponds to the requested nominal mapping:

```text
1 V → 1 mA
2 V → 2 mA
3 V → 3 mA
4 V → 4 mA
5 V → 5 mA
```

### 7.4 Load Sweep

The load sweep is defined by:

```spice
.step param Rload list 1k 5k 10k
```

This checks the current-source behavior over the requested representative load values.

---

## 8. Current Measurement

The primary output-current probe is:

```text
I(Rsense)
```

Because the sense resistor, MOSFET, and load form the same series current path, `I(Rsense)` represents the output current for this simulation.

The SENSE-node voltage is also a useful control-loop measurement:

```text
I_OUT = V(SENSE) / 1 kΩ
```

---

## 9. Baseline Simulation Status

A working baseline simulation has been run with:

```text
Vamp       = 5 V
Rload      = 5 kΩ
Frequency  = 50 Hz
Pulse width = 200 µs
```

Observed behavior:

- SENSE voltage rises to approximately 5 V during the active PWM pulse.
- `I(Rsense)` rises to approximately 5 mA.
- The current returns to approximately 0 mA when the PWM signal is inactive.
- The current pulse has a clean flat top over the active pulse interval.
- No pronounced flat-top droop or sustained ringing was observed in the baseline waveform.

The baseline result therefore demonstrates the intended 5 mA current-sink behavior for that tested operating point.

---

## 10. Load Sweep Status

A load sweep was run using:

```spice
.step param Rload list 1k 5k 10k
```

At the 5 mA command, the plotted current traces overlap closely across:

```text
1 kΩ
5 kΩ
10 kΩ
```

This indicates approximately constant current over the tested load values.

The repository should only claim measured performance that was actually observed in the simulations.

---

## 11. Current Sweep Status

The current-level sweep is run with:

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

The current is evaluated using `I(Rsense)`.

---

## 12. Results Directory

Recommended result filenames:

```text
Results/
├── baseline_5mA_5k_50Hz_200us.png
├── load_sweep_1k_5k_10k.png
└── current_sweep_1mA_5mA.png
```

Only simulations that were actually performed should be represented as results.

---

## 13. Repository Structure

```text
VCCS/
├── README.md
│
├── LTspice/
│   └── VCCS.asc
│
├── Schematic/
│   └── VCCS_schematic.png
│
└── Results/
    ├── baseline_5mA_5k_50Hz_200us.png
    ├── load_sweep_1k_5k_10k.png
    └── current_sweep_1mA_5mA.png
```

### File descriptions

`LTspice/VCCS.asc`  
The editable LTspice schematic containing the circuit and simulation setup.

`Schematic/VCCS_schematic.png`  
Visual representation of the complete circuit schematic.

`Results/`  
Screenshots of LTspice simulation waveforms actually generated during testing.

`README.md`  
Project description, circuit topology, calculations, component values, simulation configuration, and current status.

---

## 14. Current Project Status

### Completed / demonstrated

- VCCS topology implemented in LTspice.
- +60 V high-voltage supply implemented.
- +15 V single-supply op-amp rail implemented.
- PWM voltage source implemented.
- 1 kΩ current-sense resistor implemented.
- NMOS current sink implemented.
- 100 Ω gate resistor implemented.
- 100 kΩ gate pull-down implemented.
- 5 mA baseline current waveform demonstrated.
- 5 kΩ baseline load demonstrated.
- 50 Hz / 200 µs baseline pulse demonstrated.
- Load sweep across 1 kΩ, 5 kΩ, and 10 kΩ performed.
- PWM-amplitude/current-level sweep performed.

### Not claimed as fully verified

The following should only be claimed as fully verified after the corresponding simulations and numerical measurements have been completed:

- Complete ±10% accuracy across every required operating point.
- Complete frequency verification from 1 Hz through 200 Hz.
- Complete pulse-width verification from 20 µs through 500 µs.
- Quantified RMS noise and peak-to-peak ripple across the full required operating range.

The assignment permits partial results when limitations are clearly and honestly documented.

---

## 15. Reproducing the Baseline Simulation

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
8. Plot `I(Rsense)`.

The expected nominal baseline waveform is a current pulse of approximately:

```text
5 mA during the active PWM interval
≈0 mA during the inactive interval
```

---

## 16. Safety and Scope

This project is an educational simulation for the supplied electronics assignment.

The assignment states that the circuit is not recommended for real-world use or connection to a person without appropriate safety, regulatory, and medical-device requirements being satisfied.

The LTspice circuit, model, and simulation results in this repository are intended for educational and engineering evaluation only.
