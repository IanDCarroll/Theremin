# Design 4: Heterodyne — RCA Armstrong Oscillators, Historical Analog

Part of [OSCILLATOR-DESIGNS.md](../OSCILLATOR-DESIGNS.md)'s candidate
set. Diagram convention is documented in
[DIAGRAMMING.md](../DIAGRAMMING.md). Schematic source:
[diagram.mmd](diagram.mmd) — paste into
[mermaid.live](https://mermaid.live) or open with a Mermaid-aware
viewer/editor extension to render it.

**What this is a copy of.** This is the pitch half of the RCA AR-1264
theremin (1929), the commercial instrument built to Leon Theremin's
design, with modern parts in the tubes' roles. It replaces an earlier
"Hartley" design that had no source behind it; the repo's historical
designs now all follow one reference, in this order:

1. **Theremin's own patent**, [US 1,661,058](https://patents.google.com/patent/US1661058A/en)
   (German priority 1924, filed 5 December 1925, granted 28 February
   1928), decides the *architecture*: two nearly equal radio-frequency
   oscillators of the same kind, one with the pitch antenna on its grid
   (Fig. 1), combined in a detector with a grid leak so that only their
   difference is heard. The patent names no oscillator topology and
   gives no values.
2. **The RCA AR-1264 schematic** supplies the concrete circuit wherever
   the patent is silent: Armstrong-type oscillators (a tuned grid coil
   and a magnetically coupled plate coil, with plate-to-grid
   capacitors), a two-grid mixer tube, and the antenna coupled through
   a large series coil. The two sources agree on everything they both
   show; where they differ (the patent's triode detector, RCA's
   two-grid mixer; the antenna wired straight to the oscillator grid,
   or through a coil) this design follows RCA.

The other historical parts are
[Volume Design 4](../volume-4-rca-historical/README.md) and
[VCA Design 5](../vca-5-rca-historical/README.md). The modern
heterodyne is [Design 5](../oscillator-5-colpitts-heterodyne/README.md).

**How the tube roles were replaced.** JFETs are used wherever a tube's
grid controls the current, because a JFET is much closer to a triode at
the control electrode (conducts at zero volts, cut off by a negative
gate, almost no input current, a gate junction that conducts like a
grid); the reasoning and the places the match is poor are in
[TUBE-EMULATION.md](../../notes/TUBE-EMULATION.md).

| RCA element | This design |
|---|---|
| V3 and V1, UY-227 triodes, cathode grounded, grid fed by a coil | N-channel JFETs (2N5484), source grounded, gate on the grid coil; both have a very high input resistance, so the tuned circuit is not loaded the way a bipolar base would load it |
| T6 and T8, each a tuned grid winding and a plate winding on one core | Two coils each, drawn as separate parts with a dotted magnetic-coupling line (not a wire) |
| C19 (1000 pF) and C18 (100 pF variable), plate to grid | The same two capacitors, drain to gate |
| C16 (55 pF variable, the front-panel Pitch control) on the reference oscillator's grid coil | Pitch Tuning Capacitor |
| R15 (5 kΩ) and C17 (0.5 µF) returning the grid coil to ground | Return resistor and bypass capacitor with the same time constant (see Notes) |
| T9, the pitch antenna's large single-layer coil and a small "concentrated" coil, and C20 (2200 pF) in series, from the variable oscillator's grid to the antenna | Antenna Coil, Concentrated Coil and Antenna Blocking Capacitor, in that series order |
| V2, UY-224 tetrode mixer: the reference oscillator on the control grid, the variable oscillator through R16 (10 kΩ) on the screen grid | Two JFETs in series (a cascode) with one gate each: the closest breadboard analogue of a two-grid tube, since dual-gate FETs are no longer easy to buy |
| T7 transformer taking the mixer's plate output | A drain resistor, a bypass capacitor against the sum frequency and an output capacitor |
| B+ decoupling, R14 (10 kΩ) and C15 (2 µF) | The Supply Decoupling Resistor and Capacitor, scaled to a 12 V supply |
| Audio preamplifier V4, output tube V7, loudspeaker | Not part of this stage (see VCA Design 5 for V4's role) |

**How the diagram works.**
- Each oscillator regenerates through its coupled coils and the
  plate-to-grid capacitors; the grid circuit sets the frequency. The
  reference oscillator is tuned by its variable capacitor to about
  173 kHz.
- In the variable oscillator the antenna circuit replaces that
  capacitor. The Concentrated Coil, Blocking Capacitor and Antenna Coil
  together form a series branch that resonates near 172 kHz with the
  antenna's own capacitance. The hand adds capacitance, lowers that
  resonance, and pulls the oscillator frequency down: the original's
  range was about 1.4 kHz at a hand distance of about 1 inch.
- The mixer's two controlling gates are driven by the two oscillators;
  its drain current contains the sum (about 345 kHz) and the difference
  (0 to about 1.4 kHz) of their frequencies. The bypass capacitor
  removes most of the sum, as the transformer's limited bandwidth did.
- **Setting up** (from the RCA description): with the hand away from the
  antenna, trim the variable oscillator slightly *above* the antenna
  circuit's resonance, then set the reference oscillator to the same
  frequency, which gives zero beat; the front-panel capacitor and the
  adjustable plate-to-grid capacitor give the range.

**Performance — pros and cons (expected, not yet bench-verified):**
- **Pro:** It is the instrument the repo's historical story is about,
  down to its way of setting the pitch range.
- **Pro:** Full heterodyne sensitivity, like Design 5, from the
  original's arrangement: the antenna pulls the oscillator through its
  own series branch rather than being part of the oscillator's tank.
- **Con:** Most parts of any pitch design (28, including four JFETs
  and six coils), all of them hand-wound; the antenna branch's 50 mH
  and 7 mH are large inductors.
- **Con:** The range is narrow by modern standards, about 1.4 kHz at
  the closest practical hand distance, as the original's was.
- **Con:** The JFET's transconductance (3 to 6 mS for a 2N5484) is
  lower than the tube's, and nothing here has been checked for whether
  these oscillators start; this is the largest unknown.
- **Con:** Still susceptible to the two oscillators locking together
  near zero beat, as every two-oscillator theremin is, so shielding and
  separation matter.

**Notes:**
- **None of the coil or most capacitor values are from the service
  notes.** They give the frequencies (173.4 kHz reference, 172 kHz to
  about 173.4 kHz variable, 420 kHz volume), the capacitors and
  resistors shown on the schematic and the coils' DC resistance only. I
  chose the coils so the frequencies come out: with 1100 pF from the two
  plate-to-grid capacitors and the two coils adding (0.74 mH), about
  176 kHz before trimming, ignoring the mutual coupling; and 57 mH
  in the antenna branch against about 15 pF of antenna, about 172 kHz.
  Both are hand arithmetic, not measurements.
- The original's grid-return resistor is 5 kΩ with a 0.5 µF bypass, a
  time constant of 2.5 ms. A JFET's gate draws almost no current, so
  5 kΩ would give no self-bias; the resistor here is 100 kΩ with 22 nF,
  the same 2.2 ms, to keep the same "grid-leak" timing. Whether the
  oscillators then limit their amplitude as the tube circuit did is
  untested.
- The antenna is the rod described in the original's drawing (17.5 in
  by 0.44 in); see [antenna-sensor](../antenna-sensor/README.md) for the
  antenna as a part.
- The JFET's biasing at the mixer's upper gate (the Upper Gate Bias
  Resistor from gate to source) keeps its gate-source voltage at zero
  volts without the lower device's drain voltage cutting it off.
- The original's mixer plate fed an audio transformer; the audio
  output's level here has not been calculated, and VCA Design 5 expects
  about 1 V peak-to-peak.

**Volume mirror.** [Volume Design 4](../volume-4-rca-historical/README.md)
copies this design's reference oscillator node for node (same names)
and replaces everything after it, because in the original the volume
channel was one oscillator and a loop rather than a second heterodyne
pair. A change to the oscillator here should be carried over to it, and
vice versa. See [VOLUME-DESIGNS.md](../VOLUME-DESIGNS.md).

**Sources:**
- [US 1,661,058, L. S. Theremin, Method of and apparatus for the generation of sounds](https://patents.google.com/patent/US1661058A/en)
  — the drawings (Fig. 1, 11 and 12 read as images from the patent PDF)
  and text. A local copy of the PDF is in
  [notes/US1661058.pdf](../../notes/US1661058.pdf).
- [The RCA Theremin, theremin.us (Arthur Harrison)](https://theremin.us/RCA/rca_theremin.html)
  — the redrawn 1929 schematic (page 2 of 2, read as an image),
  frequencies, setup and the Armstrong description.
- [RCA Theremin Service Notes, thereminvox.com](https://thereminvox.com/stories/tech/rca-theremin-service-notes/6/)
  — the service notes' own text, read through a summary.
