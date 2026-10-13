# Volume Design 3: Comparator Relaxation Oscillator (LM393), Frequency-to-Voltage

Part of [VOLUME-DESIGNS.md](../VOLUME-DESIGNS.md)'s candidate set.
Diagram convention is documented in
[DIAGRAMMING.md](../DIAGRAMMING.md). Schematic source:
[diagram.mmd](diagram.mmd) — paste into
[mermaid.live](https://mermaid.live) or open with a Mermaid-aware
viewer/editor extension to render it.

**Mirrors [Design 3](../oscillator-3-comparator-lm393/README.md).**
The Volume Oscillator subgraph is Design 3's circuit, node for node,
wired to a volume antenna; the second subgraph is the
frequency-to-voltage converter and conditioning that Design 3 doesn't
have (shared explanation in [VOLUME-DESIGNS.md](../VOLUME-DESIGNS.md),
contract in [VOLUME-OSCILLATOR.md](../../notes/VOLUME-OSCILLATOR.md)).
The two designs mirror each other: a change to the oscillator in one
should be carried over to the other. Either can be paired with any
pitch design.

**What differs from Design 3:**

| | Design 3 (pitch) | Volume Design 3 | Why |
|---|---|---|---|
| Timing Resistor | ~10 kΩ | ~680 kΩ | Brings the frequency down to about 10.6 kHz |
| Hysteresis Feedback Resistor | ~1 MΩ | ~100 kΩ | Widens the thresholds to about one third and two thirds of the supply, which makes the frequency a simple 0.72 / (R × C) |
| Output | Pitch Signal Output | Coupling capacitor into the converter | The oscillator's job here is to feed the converter |
| Antenna | Antenna | Volume Antenna | Same wire, same sensing node |
| Added | — | LM2907-N converter plus floor and range conditioning (10 parts) | Turns frequency into the control voltage |

The Pull-Up Resistor, both Hysteresis Divider Resistors and the Timing
Capacitor are Design 3 unchanged. Total parts: 17 (Design 3's 7, plus
10).

**Performance — pros and cons (expected, not yet bench-verified):**
- **Pro:** We set the thresholds ourselves, and with the feedback
  resistor at 100 kΩ they are a fixed fraction of the supply, so both
  the oscillator's timing and the converter's output scale with the
  supply together.
- **Pro:** A purpose-built comparator switches cleanly, and at 10 kHz
  its response time (typical 1.3 µs for the LM393 in the TI datasheet)
  is a tiny fraction of the period.
- **Pro:** The converter's output is ratiometric to the supply (V_OUT
  = V_CC × f × C1 × R1), so with these values the output at "hand
  away" is about half the supply at any supply.
- **Con (the big one):** the converter's output is proportional to
  frequency, so the hand's contribution rides on a large constant: a
  few pF of hand capacitance on roughly 100 pF of timing capacitance
  moves the frequency by a few percent, and the conditioning stage has
  to subtract the constant and amplify by about 30×, amplifying
  ripple, noise and drift by the same factor. The one published
  attempt at this pattern I found (a UIUC student project with 8038
  oscillator pairs and an LM2917) abandoned its volume circuit over
  offset and drift.
- **Con:** with a 680 kΩ timing resistor on a 100 pF capacitor, the
  comparator's input bias current (typical 25 nA, maximum 250 nA for
  the LM393 in the TI datasheet) across that resistance is 17 mV
  typical and up to 170 mV worst case. With the thresholds a third and
  two thirds of a 9 V supply (a 3 V swing), 170 mV shifts the
  frequency by up to about 6%, which is larger than the hand's own few
  percent. The floor trimmer has to absorb that offset, and any drift
  of it with temperature is amplified like the signal.
- **Con:** ripple against speed, as in the other single-oscillator
  volume designs: with the illustrative values about a 47 ms time
  constant and about 66 mV (roughly 1.5% of full scale) of ripple
  after the ×31 gain.
- **Con:** most parts of the three single-oscillator volume designs
  (17).

**Notes:**
- Frequency plan. With the thresholds at one third and two thirds of
  the supply, the half period is R × C × ln 2, so f = 0.72 / (R × C),
  about 10.6 kHz for 680 kΩ and 100 pF. Design 3's own illustrative
  values (1 MΩ feedback, so thresholds at about 48% and 52%, and 10 kΩ
  with 100 pF) work out to about 5 MHz, well beyond the comparator's
  response time, so the pitch values would be limited by the part
  rather than by the RC network. (That is worth a look on the pitch
  side too.)
- Converter: f_MAX = I2 / (C1 × V_CC) from the LM2907-N datasheet,
  with I2 about 180 µA: 20 kHz at 9 V and 15 kHz at 12 V, so 10.6 kHz
  fits at either. C1 = 1 nF, R1 = 47 kΩ gives V_OUT about half the
  supply, and the ripple filter capacitor sets ripple and response
  time: ripple is (V_CC/2) × (C1/C2) × (1 − V_CC × f × C1 / I2)
  peak-to-peak and the time constant is R1 × C2.
- The converter's 8-pin input is referenced to ground and compares
  against ±25 mV (typical) with 30 mV of hysteresis, so the
  comparator's 0-to-supply square wave is AC-coupled and re-centred by
  the Input Bias Resistor.
- The converter's internal op amp does the conditioning: pins 7 (IN−)
  and 4 (EMIT) form a non-inverting amplifier of gain 1 + Gain
  Feedback Resistor / Gain Resistor (about 31) whose gain leg returns
  to the Floor Trimmer, so V_OUT = G × V_IN − (G − 1) × V_trimmer. The
  trimmer is a 3-lead part, so every lead is labelled. Whether that op
  amp is happy at this gain is unverified.
- The LM393 package has two comparators; this design only uses one, as
  in Design 3. The second comparator's pins aren't shown.
- The Timing Capacitor is frequency-determining, so the
  [CAPACITORS.md](../../notes/CAPACITORS.md) guidance (C0G/NP0 ceramic
  or polypropylene film) applies. Values shown are illustrative
  starting points, to be refined once we're actually prototyping.

**Sources:**
- [LM393 / LM2903 family datasheet, Texas Instruments](https://www.ti.com/lit/ds/symlink/lm393.pdf)
  — response time and input bias current. Datasheet text read
  directly.
- [LM2907-N / LM2917-N datasheet, Texas Instruments (SNAS555D)](https://www.ti.com/lit/ds/symlink/lm2907-n.pdf)
  — pinout, electrical characteristics, Figure 15, equations 2–7.
  Datasheet text and Figure 15 read directly.
- [Pat Tarantino, "Theremin" (UIUC Physics 406)](https://courses.physics.illinois.edu/phys406/sp2017/Student_Projects/Spring01/PTarantino/Pat_Tarantino_Theremin.pdf)
  — section 2.3, the abandoned LM2917 volume circuit.
