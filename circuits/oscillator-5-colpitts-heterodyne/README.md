# Design 5: Heterodyne — Colpitts BJT Oscillators + Diode Mixer

Part of [OSCILLATOR-DESIGNS.md](../OSCILLATOR-DESIGNS.md)'s candidate
set. Diagram convention is documented in
[DIAGRAMMING.md](../DIAGRAMMING.md). Schematic source:
[diagram.mmd](diagram.mmd) — paste into
[mermaid.live](https://mermaid.live) or open with a Mermaid-aware
viewer/editor extension to render it.

The general modern-parts version of a "real" theremin pitch circuit:
two RF LC oscillators (one fixed, one antenna-controlled), mixed to
produce an audible beat frequency. See
[PITCH-OSCILLATOR.md](../../notes/PITCH-OSCILLATOR.md) for why
heterodyning is worth the extra complexity (sensitivity gain from
beat-frequency downconversion).

Each oscillator uses a **Colpitts** topology — an untapped inductor
plus a two-capacitor divider for feedback — because, like the
[Seven Transistor Labs](https://www.seventransistorlabs.com/Theremin/)
build this follows, it's easier to wind a plain coil than a tapped one.
Compare with [Design 4](../oscillator-4-hartley-heterodyne-historical/README.md),
which uses a tapped-coil Hartley topology instead.

**Performance — pros and cons (expected, not yet bench-verified):**
- **Pro:** Same full heterodyne sensitivity gain as Design 4 — the
  widest, most expressive pitch range of the five (see
  [PITCH-OSCILLATOR.md](../../notes/PITCH-OSCILLATOR.md)).
- **Pro:** Uses only ordinary off-the-shelf components — no hand-wound
  tapped coil — meaningfully easier to build correctly than Design 4
  despite the similar part count.
- **Pro:** A very common, well-documented oscillator topology, so
  there's substantial reference material available for tuning and
  troubleshooting.
- **Con:** Still inherits every heterodyne downside: two oscillators to
  match and keep in tune, and absolute pitch drift risk if either one
  drifts with temperature, humidity, or component aging.
- **Con:** Highest total part count of the five (21) — most assembly
  time and the most potential wiring mistakes on a breadboard.
- **Con:** Same lossy, nonlinear diode-mixer limitation as Design 4 —
  likely needs a buffer/amplifier stage before the output is usable.
- **Con:** The capacitive feedback divider needs reasonably well-matched
  capacitor values to hit its intended feedback fraction — a practical
  tuning burden the single-oscillator designs (1–3) don't have.

**Notes:**
- Both oscillators share the same physical Power Supply Positive and
  Ground rails in the real build; the diagram shows them as shared
  nodes reused across both subgraphs for exactly that reason.
- The antenna has a single wire, landing on the Variable Oscillator's
  emitter lead alongside the Feedback Capacitor (Lower) — it doesn't
  get a second wire back to Ground. Physically, its "other plate" is
  the surrounding environment/the player's body, not anything actually
  wired into our circuit — see
  [ANTENNA-CAPACITIVE-SENSOR.md](../../notes/ANTENNA-CAPACITIVE-SENSOR.md)
  for why. Its variability shifts the Variable Oscillator's total
  capacitance at that node, and therefore its frequency.
- The mixer is a simple series-diode envelope detector — the same
  principle a basic AM radio's detector stage uses — followed by a
  single RC low-pass to keep only the audible difference frequency and
  discard the RF and its sum-frequency product. The Low-Pass Filter
  Resistor and Capacitor are drawn hanging off the Pitch Signal Output
  node rather than off the diode directly — that net is genuinely the
  same point electrically, and it keeps the Mixer Diode down to a clean
  one-wire-per-lead diagram (2 signals in on the anode, 1 out the
  cathode) instead of accumulating five connections on one part.
- No tuning trimmer is included — that's a calibration nicety for
  zero-beat setup (see [ANTENNA-CAPACITIVE-SENSOR.md](../../notes/ANTENNA-CAPACITIVE-SENSOR.md)), not part of the core oscillation-and-variability mechanism this document is scoped to.
- Where a capacitor here is timing/frequency-determining (both Feedback
  Capacitors, in each oscillator), the
  [CAPACITORS.md](../../notes/CAPACITORS.md) guidance (C0G/NP0 ceramic
  or polypropylene film) applies. Values shown are illustrative
  starting points, to be refined once we're actually prototyping.
