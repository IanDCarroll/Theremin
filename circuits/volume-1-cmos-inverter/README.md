# Volume Design 1: CMOS Inverter (CD4069UB), Frequency-to-Voltage

Part of [VOLUME-DESIGNS.md](../VOLUME-DESIGNS.md)'s candidate set.
Diagram convention is documented in
[DIAGRAMMING.md](../DIAGRAMMING.md). Schematic source:
[diagram.mmd](diagram.mmd) — paste into
[mermaid.live](https://mermaid.live) or open with a Mermaid-aware
viewer/editor extension to render it.

**Mirrors [Design 1](../oscillator-1-cmos-inverter/README.md).** The
Volume Oscillator subgraph is Design 1's circuit, node for node, wired
to a volume antenna. What Design 1 doesn't have is the second
subgraph: a frequency-to-voltage converter that turns the oscillator's
hand-dependent frequency into the DC control voltage the VCA expects
(contract in [VOLUME-OSCILLATOR.md](../../notes/VOLUME-OSCILLATOR.md),
"Subsystem boundary"; shared explanation of that stage in
[VOLUME-DESIGNS.md](../VOLUME-DESIGNS.md)). The two designs mirror
each other: a change to the oscillator in one should be carried over
to the other. Either can be paired with any pitch design.

**What differs from Design 1:**

| | Design 1 (pitch) | Volume Design 1 | Why |
|---|---|---|---|
| Feedback Resistor | ~4.7 MΩ (about 1 kHz) | ~470 kΩ (about 10 kHz) | The converter wants roughly 10 kHz, and the volume oscillator must sit away from the pitch oscillator's frequency |
| Output | Pitch Signal Output | Coupling capacitor into the converter | The oscillator's job here is to feed the converter, not a speaker |
| Antenna | Antenna | Volume Antenna | Same wire, same sensing node |
| Added | — | LM2907-N converter plus floor and range conditioning (10 parts) | Turns frequency into the control voltage |

Total parts: 13 (Design 1's 3, plus 10).

**Performance — pros and cons (expected, not yet bench-verified):**
- **Pro:** The fewest oscillator parts of the five volume designs (3),
  and the CMOS gate input barely loads the antenna node — the same
  advantages Design 1 has as a pitch oscillator.
- **Pro:** The converter's output is ratiometric to the supply (V_OUT
  = V_CC × f × C1 × R1, LM2907-N datasheet equation 5). With these
  values the output at "hand away" is about half the supply whatever
  the supply is, and the floor trimmer sits across the same supply, so
  both track a sagging battery together.
- **Pro:** The converter is an off-the-shelf part with a stated ±0.3%
  typical linearity and a pinout and application circuit in a
  manufacturer datasheet.
- **Con (the big one):** the output is proportional to *frequency*, so
  the hand's contribution rides on a large constant. A few pF of hand
  capacitance on roughly 100 pF of timing capacitance moves the
  frequency by only a few percent, so only a few percent of the
  converter's output span carries the volume. The conditioning stage
  has to subtract the constant and amplify by about 30×, and it
  amplifies ripple, noise and drift by the same factor. The one
  published attempt at this pattern I found (a UIUC student project
  with 8038 oscillator pairs and an LM2917) abandoned its volume
  circuit for this reason.
- **Con:** ripple against speed. With the illustrative values the
  ripple filter's time constant is about 47 ms (slow for staccato
  articulation) and the converter's ripple is about 2 mV peak-to-peak,
  which is about 66 mV (roughly 1.5% of full scale) after the ×31
  gain. Making the filter faster makes the ripple worse.
- **Con:** no natural null. The floor trimmer has to be set to the
  frequency the oscillator has with the hand at its nearest, and
  anything that moves the oscillator's frequency afterwards moves the
  floor. CMOS gate thresholds shift with supply and temperature, so
  this is probably the least stable of the three single-oscillator
  volume designs.

**Notes:**
- **Inherited open question from Design 1.** Design 1 draws one
  inverter with only a feedback resistor and a capacitor. In the ideal
  model that loop has too little phase shift to oscillate (one RC lag
  plus the gate itself), and the usual one-gate RC oscillators use a
  Schmitt-trigger gate (CD40106) or two gates; the UBC build linked
  from the antenna-sensor README uses two CD4069UB gates. Whether this
  single-gate form oscillates has not been checked on a bench, and
  this volume design inherits the answer. If Design 1 changes, change
  this one with it.
- Frequency plan. The oscillator frequency is about 1/(2.2 × R × C),
  which gives roughly 9.7 kHz here. The converter's maximum input
  frequency is f_MAX = I2 / (C1 × V_CC) from the datasheet, with I2
  about 180 µA: 20 kHz at 9 V, 15 kHz at 12 V. About 10 kHz sits under
  that limit at either supply.
- Converter values: C1 = 1 nF (the datasheet says not to go below 500
  pF), R1 = 47 kΩ so that V_OUT is about half the supply at 10.5 kHz,
  and the ripple filter capacitor C2 sets both ripple and response
  time: ripple is (V_CC/2) × (C1/C2) × (1 − V_CC × f × C1 / I2)
  peak-to-peak and the time constant is R1 × C2.
- The converter's input is the 8-pin LM2907-N, whose input is
  referenced to ground and compares against ±25 mV (typical) with 30
  mV of hysteresis. A 0-to-V_CC square wave never goes negative, so it
  is AC-coupled through the Input Coupling Capacitor and re-centred
  with the Input Bias Resistor. The datasheet's input bias current is
  about 0.1 µA, which across 100 kΩ is about 10 mV of offset against
  that threshold.
- The converter's internal op amp does the conditioning. Pin 7 (IN−)
  and pin 4 (EMIT) are wired as a non-inverting amplifier of gain 1 +
  Gain Feedback Resistor / Gain Resistor (about 31 here) with the
  "ground" leg of the gain network returned to the Floor Trimmer
  instead of to ground. That shifts the floor: V_OUT = G × V_IN − (G −
  1) × V_trimmer. The trimmer is a 3-lead part, so each of its leads
  is labelled.
- Unused: nothing on the CD4069UB beyond the one gate (as in Design 1:
  the other five gates are not drawn), and no zener (the 8-pin LM2917
  variant adds one, along with a dropping resistor, if supply
  independence matters more than the extra part).
- The Timing Capacitor is frequency-determining, so the
  [CAPACITORS.md](../../notes/CAPACITORS.md) guidance (C0G/NP0 ceramic
  or polypropylene film) applies. Values shown are illustrative
  starting points, to be refined once we're actually prototyping.

**Sources:**
- [LM2907-N / LM2917-N datasheet, Texas Instruments (SNAS555D)](https://www.ti.com/lit/ds/symlink/lm2907-n.pdf)
  — pinout (Pin Functions, 8-pin), electrical characteristics, Figure
  15 (minimum-component converter), equations 2–7. Datasheet text and
  Figure 15 read directly.
- [Pat Tarantino, "Theremin" (UIUC Physics 406)](https://courses.physics.illinois.edu/phys406/sp2017/Student_Projects/Spring01/PTarantino/Pat_Tarantino_Theremin.pdf)
  — section 2.3 describes the LM2917 volume circuit and why it was
  abandoned. Read in full for that section.
