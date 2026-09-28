# Design 2: 555 Timer (Direct RC Relaxation Oscillator)

Part of [OSCILLATOR-DESIGNS.md](../OSCILLATOR-DESIGNS.md)'s candidate
set. Diagram convention is documented in
[DIAGRAMMING.md](../DIAGRAMMING.md). Schematic source:
[diagram.mmd](diagram.mmd) — paste into
[mermaid.live](https://mermaid.live) or open with a Mermaid-aware
viewer/editor extension to render it.

The common kit approach: one oscillator, no mixer, antenna wired
directly into the timing network. See
[PITCH-OSCILLATOR.md](../../notes/PITCH-OSCILLATOR.md) for why this
direct-RC approach trades sensitivity for simplicity compared to the
heterodyne designs
([4](../oscillator-4-hartley-heterodyne-historical/README.md),
[5](../oscillator-5-colpitts-heterodyne/README.md)).

**Performance — pros and cons (expected, not yet bench-verified):**
- **Pro:** The classic NE555's documented operating range (4.5–16V)
  matches our target operating window almost exactly.
- **Pro:** Robust output stage, able to source/sink meaningful current
  directly — simplest downstream interfacing of the five designs.
- **Pro:** The best-documented IC of the five, by a wide margin — the
  most reference material available for debugging.
- **Con:** No heterodyne sensitivity gain (see
  [PITCH-OSCILLATOR.md](../../notes/PITCH-OSCILLATOR.md)) — same
  flatter, less expressive pitch response as the CMOS design.
- **Con:** Its internal threshold comparators trip at fixed fractions
  of the supply voltage (1/3 and 2/3 of V+), so oscillation frequency
  has some built-in dependency on supply stability — a sagging 9V
  battery over a play session could show up as audible pitch drift.
- **Con:** Higher quiescent current than the CMOS version (Design 1),
  owing to the 555's bipolar internal design — worth factoring in for a
  battery-powered build.

**Notes:**
- Pin 4 (Reset) tied to Power Supply Positive is required for defined
  behavior per the 555 datasheet — not an optional extra, the chip's
  operation is undefined with it floating.
- The Control Voltage Bypass Capacitor, on pin 5, is the single
  deliberate exception to "no extras" — stated directly in the ask for
  this design, and consistent with the 555's own datasheet guidance for
  astable use.
- The Timing Capacitor needs *some* fixed baseline value in parallel
  with the bare antenna — without it, the timing network's capacitance
  is too small and unstable to oscillate usefully. This is establishing
  a workable base frequency, not a tuning nicety (see the
  baseline-capacitance discussion in
  [ANTENNA-CAPACITIVE-SENSOR.md](../../notes/ANTENNA-CAPACITIVE-SENSOR.md)).
- The antenna has a single wire, landing on the Threshold Resistor's far
  lead alongside the Timing Capacitor — it doesn't get a second wire
  back to Ground; see
  [ANTENNA-CAPACITIVE-SENSOR.md](../../notes/ANTENNA-CAPACITIVE-SENSOR.md)
  for why.
- Pins 6 (Threshold) and 2 (Trigger) are two distinct physical pins on
  the chip that are simply wired to the same external node in this
  design — the diagram labels both on the one wire that reaches them.
- The Timing Capacitor is frequency-determining, so the
  [CAPACITORS.md](../../notes/CAPACITORS.md) guidance (C0G/NP0 ceramic
  or polypropylene film) applies. Values shown are illustrative
  starting points, to be refined once we're actually prototyping.
