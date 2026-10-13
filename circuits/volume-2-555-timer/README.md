# Volume Design 2: 555 Timer (Direct RC Relaxation Oscillator), Frequency-to-Voltage

Part of [VOLUME-DESIGNS.md](../VOLUME-DESIGNS.md)'s candidate set.
Diagram convention is documented in
[DIAGRAMMING.md](../DIAGRAMMING.md). Schematic source:
[diagram.mmd](diagram.mmd) — paste into
[mermaid.live](https://mermaid.live) or open with a Mermaid-aware
viewer/editor extension to render it.

**Mirrors [Design 2](../oscillator-2-555-timer/README.md).** The
Volume Oscillator subgraph is Design 2's circuit, node for node, wired
to a volume antenna; the second subgraph is the frequency-to-voltage
converter and conditioning that Design 2 doesn't have (shared
explanation in [VOLUME-DESIGNS.md](../VOLUME-DESIGNS.md), contract in
[VOLUME-OSCILLATOR.md](../../notes/VOLUME-OSCILLATOR.md)). The two
designs mirror each other: a change to the oscillator in one should be
carried over to the other. Either can be paired with any pitch design.

**What differs from Design 2:**

| | Design 2 (pitch) | Volume Design 2 | Why |
|---|---|---|---|
| Threshold Resistor | ~10 kΩ (about 690 kHz) | ~680 kΩ (about 10.5 kHz) | The converter can only count tens of kHz, and the volume oscillator must sit away from the pitch frequency |
| Output | Pitch Signal Output | Coupling capacitor into the converter | The oscillator's job here is to feed the converter |
| Antenna | Antenna | Volume Antenna | Same wire, same sensing node |
| Added | — | LM2907-N converter plus floor and range conditioning (10 parts) | Turns frequency into the control voltage |

Everything else — the Discharge Resistor, Timing Capacitor, Control
Voltage Bypass Capacitor, pin 4 tied to supply — is Design 2
unchanged. Total parts: 15 (Design 2's 5, plus 10).

**Performance — pros and cons (expected, not yet bench-verified):**
- **Pro:** The best-documented oscillator IC of the five, and a robust
  output stage, so the coupling capacitor and converter input are no
  load at all to it.
- **Pro:** The TI NE555 datasheet gives numbers for the thing that
  matters here: in astable operation the timing interval drifts about
  150 ppm/°C and 0.3 %/V of supply (typical). That is better than a
  bare CMOS gate's threshold and it is a number, not a guess.
- **Pro:** The converter's output is ratiometric to the supply (V_OUT
  = V_CC × f × C1 × R1), so with these values the output at "hand
  away" is about half the supply at any supply, and the floor trimmer
  tracks the same supply.
- **Con (the big one):** the converter's output is proportional to
  frequency, so the hand's contribution rides on a large constant: a
  few pF of hand capacitance on roughly 100 pF of timing capacitance
  moves the frequency by a few percent, and the conditioning stage has
  to subtract the constant and amplify by about 30×. Drift is
  amplified by the same factor: 0.3 %/V on a 3% span means a 1 V
  battery sag shifts the volume by about a tenth of its range, and 10
  °C is about 5% of it. The one published attempt at this pattern I
  found (a UIUC student project with 8038 oscillator pairs and an
  LM2917) abandoned its volume circuit over offset and drift.
- **Con:** ripple against speed. With the illustrative values the
  ripple filter's time constant is about 47 ms (slow for staccato
  articulation), and the converter's ripple of about 2 mV peak-to-peak
  becomes about 66 mV after the ×31 gain (roughly 1.5% of full scale).
  Making the filter faster makes the ripple worse.
- **Con:** no natural null, so the floor trimmer must be set to the
  oscillator's frequency with the hand nearest, and stays right only
  as long as that frequency does.

**Notes:**
- Frequency plan. The 555 astable frequency is 1.44 / ((R_A + 2 × R_B)
  × C), where R_A is the Discharge Resistor and R_B the Threshold
  Resistor. Design 2's own illustrative values (1 kΩ and 10 kΩ with
  100 pF) work out to about 690 kHz, which is not an audio-range pitch
  and far above anything the converter can count; this design's ~680
  kΩ gives about 10.5 kHz. (That calculation is worth a look on the
  pitch side too.)
- Converter: the maximum input frequency is f_MAX = I2 / (C1 × V_CC)
  from the LM2907-N datasheet, with I2 about 180 µA: 20 kHz at 9 V, 15
  kHz at 12 V, so 10.5 kHz fits at either. C1 = 1 nF (the datasheet's
  minimum is 500 pF), R1 = 47 kΩ gives V_OUT of about half the supply
  at 10.5 kHz, and the ripple filter capacitor sets both ripple and
  response time: ripple is (V_CC/2) × (C1/C2) × (1 − V_CC × f × C1 /
  I2) peak-to-peak and the time constant is R1 × C2.
- The converter's 8-pin input is referenced to ground and compares
  against ±25 mV (typical) with 30 mV of hysteresis. The 555's
  0-to-supply square wave never goes negative, so it is AC-coupled and
  re-centred by the Input Bias Resistor; the datasheet's input bias
  current of about 0.1 µA across 100 kΩ is about 10 mV against that
  threshold.
- The converter's internal op amp does the conditioning: pins 7 (IN−)
  and 4 (EMIT) form a non-inverting amplifier of gain 1 + Gain
  Feedback Resistor / Gain Resistor (about 31) whose gain leg returns
  to the Floor Trimmer rather than to ground, so V_OUT = G × V_IN − (G
  − 1) × V_trimmer. The trimmer is a 3-lead part, so every lead is
  labelled. Whether that op amp is happy at this gain is unverified.
- Pins 6 (Threshold) and 2 (Trigger) are two pins on one wire here
  too, exactly as in Design 2.
- The Timing Capacitor is frequency-determining, so the
  [CAPACITORS.md](../../notes/CAPACITORS.md) guidance (C0G/NP0 ceramic
  or polypropylene film) applies. Values shown are illustrative
  starting points, to be refined once we're actually prototyping.

**Sources:**
- [NE555 / SA555 / SE555 datasheet, Texas Instruments (SLFS022K)](https://www.ti.com/lit/ds/symlink/ne555.pdf)
  — astable temperature coefficient (typ 150 ppm/°C) and supply
  sensitivity (typ 0.3 %/V) in the electrical characteristics.
  Datasheet text read directly.
- [LM2907-N / LM2917-N datasheet, Texas Instruments (SNAS555D)](https://www.ti.com/lit/ds/symlink/lm2907-n.pdf)
  — pinout, electrical characteristics, Figure 15, equations 2–7.
  Datasheet text and Figure 15 read directly.
- [Pat Tarantino, "Theremin" (UIUC Physics 406)](https://courses.physics.illinois.edu/phys406/sp2017/Student_Projects/Spring01/PTarantino/Pat_Tarantino_Theremin.pdf)
  — section 2.3, the abandoned LM2917 volume circuit.
