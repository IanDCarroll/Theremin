# Design 4: Heterodyne — Hartley BJT Oscillators, Historical-Analog

Part of [OSCILLATOR-DESIGNS.md](../OSCILLATOR-DESIGNS.md)'s candidate
set. Diagram convention is documented in
[DIAGRAMMING.md](../DIAGRAMMING.md). Schematic source:
[diagram.mmd](diagram.mmd) — paste into
[mermaid.live](https://mermaid.live) or open with a Mermaid-aware
viewer/editor extension to render it.

Leon Theremin's original 1920s circuit used vacuum tube triodes in a
heterodyne pair, most consistent with the **Hartley** topology (tapped
inductor, single tank capacitor) and tube-style self-biasing — both
common in period RF oscillator practice, and a genuinely different
circuit from [Design 5](../oscillator-5-colpitts-heterodyne/README.md)'s
Colpitts-plus-resistor-divider-bias approach, not just a relabeling of
it.

This substitutes modern transistors for tubes in the *same role*
(sustaining oscillation via gain), and maps the bias scheme role-for-
role onto period tube practice:

| Tube-era element | BJT equivalent here |
|---|---|
| Grid-leak resistor + coupling cap (self-bias via grid rectification) | Grid-Leak-Style Bias Resistor + Grid-Leak-Style Coupling Capacitor (self-bias via base-emitter rectification) |
| Cathode resistor (unbypassed) | Cathode-Style Emitter Resistor (unbypassed) |
| Plate load (the tuned circuit itself) | Collector load (the tuned circuit itself) |

**Performance — pros and cons (expected, not yet bench-verified):**
- **Pro:** Full heterodyne sensitivity gain — along with Design 5, the
  only two of the five that reproduce the wide, nuanced pitch response
  a "real" theremin is known for (see
  [PITCH-OSCILLATOR.md](../../notes/PITCH-OSCILLATOR.md)).
- **Pro:** Grid-leak-style self-biasing is inherently self-limiting as
  oscillation amplitude builds — a genuinely good stabilizing property
  of this classic topology, largely "free" once the bias network is
  sized correctly.
- **Con:** Two independent oscillators have to be tuned close to each
  other; drift in either one (temperature, humidity, capacitor aging)
  shows up directly as absolute pitch drift of the whole instrument —
  the classic reason analog theremins need frequent re-tuning.
- **Con:** Requires a hand-wound tapped inductor — harder to source and
  build consistently than an off-the-shelf part, and its inductance/Q
  is sensitive to winding technique and to nearby objects/metal.
- **Con:** The passive diode mixer is lossy and nonlinear; expect a
  weak, possibly distorted output that likely needs a buffer/amplifier
  stage before it's usable downstream.
- **Con:** 17 parts to hand-build correctly on a breadboard — second
  only to Design 5, and with a custom part (the tapped coil) in the mix.

**Notes:**
- The diode mixer is reused unchanged from Design 5 — it's not a
  modern shortcut here. Vacuum diodes were used the same way (as
  detector/mixer stages) in period radio and oscillator circuits, so
  this stage is itself historically consistent, not just "close enough."
  As in Design 5, the Low-Pass Filter Resistor and Capacitor are drawn
  off the Pitch Signal Output node rather than off the diode itself, to
  keep the Mixer Diode at a clean one-wire-per-lead connection.
- The Tapped Inductor is drawn as a single component with three leads
  (Coil Top, Coil Tap, Coil Bottom) — that's what a real wound coil
  with a tap actually is: one physical part, not two. There's no way to
  fake a tap with an ordinary two-lead inductor, which is exactly why
  [Design 5](../oscillator-5-colpitts-heterodyne/README.md)'s
  modern-convenience version avoids it.
- Feedback comes in on a different transistor lead than Design 5: here
  it's the **base** (through the Grid-Leak-Style Coupling Capacitor, off
  the coil's tap), whereas Design 5's Colpitts feedback comes in on the
  **emitter** (through the capacitive divider). That's a real structural
  difference between the two topologies, not just a relabeling.
- The antenna has a single wire, landing on the Variable Oscillator's
  collector lead alongside the Tank Capacitor — it doesn't get a second
  wire back to Ground; see
  [ANTENNA-CAPACITIVE-SENSOR.md](../../notes/ANTENNA-CAPACITIVE-SENSOR.md)
  for why.
- Where a capacitor here is timing/frequency-determining (each
  oscillator's Tank Capacitor), the
  [CAPACITORS.md](../../notes/CAPACITORS.md) guidance (C0G/NP0 ceramic
  or polypropylene film) applies. Values shown are illustrative
  starting points, to be refined once we're actually prototyping.
