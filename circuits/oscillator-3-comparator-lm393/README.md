# Design 3: Comparator Relaxation Oscillator (LM393)

Part of [OSCILLATOR-DESIGNS.md](../OSCILLATOR-DESIGNS.md)'s candidate
set. Diagram convention is documented in
[DIAGRAMMING.md](../DIAGRAMMING.md). Schematic source:
[diagram.mmd](diagram.mmd) — paste into
[mermaid.live](https://mermaid.live) or open with a Mermaid-aware
viewer/editor extension to render it.

A dedicated comparator IC instead of an op-amp, since op-amps aren't
generally designed for open-loop comparator duty (slow slew rate,
outputs that don't swing fully to the rails) — an LM393 is a common,
well-documented, low-cost part built for exactly this job.

**Performance — pros and cons (expected, not yet bench-verified):**
- **Pro:** We set our own switching thresholds via the Hysteresis
  Divider Resistors, rather than relying on a fixed internal ratio —
  more direct control over the oscillator's behavior than the 555 or
  CMOS versions offer, without adding an active component.
- **Pro:** A purpose-built comparator generally switches cleaner and
  faster than a 555's internal comparators or a CMOS gate run in its
  linear region, for crisper timing edges.
- **Con:** No heterodyne sensitivity gain (see
  [PITCH-OSCILLATOR.md](../../notes/PITCH-OSCILLATOR.md)) — same
  flatter, less expressive pitch response as Designs 1–2.
- **Con:** The hysteresis thresholds are set relative to the supply
  voltage, so — like the 555 — this design shares some risk of pitch
  drift as the supply sags, unless the divider is designed to be
  reasonably insensitive to it.
- **Con:** Most parts (7) of the three single-oscillator designs — more
  to assemble correctly, particularly getting the two Hysteresis
  Divider Resistors well-matched.

**Notes:**
- The Pull-Up Resistor is required, not an extra — the LM393's output
  stage is open-collector and does nothing useful without one.
- The two Hysteresis Divider Resistors and the Hysteresis Feedback
  Resistor form the positive-feedback network that sets the two
  switching thresholds. This is core to how a comparator-based
  relaxation oscillator works at all — without hysteresis, a comparator
  with only an RC charging path wouldn't oscillate, it would just
  settle at one threshold. Not a guard, a load-bearing part of the
  topology.
- The LM393 package has two comparators; this design only uses one.
  The second comparator's pins aren't shown — unused and out of scope
  for this circuit, not omitted by mistake.
- The antenna has a single wire, landing on the Inverting Input
  alongside the Timing Capacitor — it doesn't get a second wire back to
  Ground; see
  [ANTENNA-CAPACITIVE-SENSOR.md](../../notes/ANTENNA-CAPACITIVE-SENSOR.md)
  for why.
- The Timing Capacitor is frequency-determining, so the
  [CAPACITORS.md](../../notes/CAPACITORS.md) guidance (C0G/NP0 ceramic
  or polypropylene film) applies. Values shown are illustrative
  starting points, to be refined once we're actually prototyping.
