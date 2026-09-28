# Design 1: CMOS Inverter (CD4069UB Relaxation Oscillator)

Part of [OSCILLATOR-DESIGNS.md](../OSCILLATOR-DESIGNS.md)'s candidate
set. Diagram convention is documented in
[DIAGRAMMING.md](../DIAGRAMMING.md). Schematic source:
[diagram.mmd](diagram.mmd) — paste into
[mermaid.live](https://mermaid.live) or open with a Mermaid-aware
viewer/editor extension to render it.

The minimal version: one gate, one resistor, one capacitor.

**Performance — pros and cons (expected, not yet bench-verified):**
- **Pro:** CMOS inputs are extremely high-impedance, so the timing/
  antenna node is barely loaded by the gate itself — about as little
  interference with a tiny antenna signal as any of the five designs.
- **Pro:** CD4000-series parts are commonly rated for roughly 3–18V,
  comfortably covering our 4.5–16V target with margin either direction.
- **Pro:** Very low quiescent current — the best fit of the five for a
  battery-powered build.
- **Con:** No heterodyne sensitivity gain (see
  [PITCH-OSCILLATOR.md](../../notes/PITCH-OSCILLATOR.md)) — the same
  hand-capacitance change produces a much smaller frequency shift here
  than in Designs 4–5. Expect the least expressive, most "compressed"
  pitch response of the five.
- **Con:** Frequency depends on the gate's own switching threshold,
  which shifts somewhat with supply voltage and temperature — likely
  the least frequency-stable of the three single-oscillator designs,
  since it has no fixed internal reference the way the 555 or a
  comparator's own hysteresis network does.
- **Con:** Square-wave output only, and no natural "null point"/silence
  zone the way a heterodyne beat-frequency design has — it's always
  audible, which is a different playing feel than a "real" theremin.

**Notes:**
- CD4069UB specifically (the **un**buffered variant) rather than an
  ordinary logic inverter — its softer, more linear transfer curve is
  what lets a single gate self-bias into an active region and function
  as an analog oscillator gain stage. A standard buffered/Schmitt-style
  inverter doesn't behave the same way here.
- This is the true minimal form (one gate, the Feedback Resistor, and
  the Timing Capacitor) — there's a common 3-component variant that
  adds a series input-protection resistor, which is a genuine extra
  (input protection, not required for oscillation) and is left out here
  per "no extras."
- The package has six inverter gates; this design only uses one. The
  other five gates and their pins aren't shown — they're unused and out
  of scope for this circuit, not omitted by mistake. Exact pin numbers
  for the gate and power pins used should be confirmed against the
  CD4069UB datasheet when wiring the actual breadboard.
- The antenna has a single wire, landing on the gate's input alongside
  the Timing Capacitor — it doesn't get a second wire back to Ground;
  see [ANTENNA-CAPACITIVE-SENSOR.md](../../notes/ANTENNA-CAPACITIVE-SENSOR.md)
  for why.
- The Timing Capacitor is frequency-determining, so the
  [CAPACITORS.md](../../notes/CAPACITORS.md) guidance (C0G/NP0 ceramic
  or polypropylene film) applies. Values shown are illustrative
  starting points, to be refined once we're actually prototyping.
