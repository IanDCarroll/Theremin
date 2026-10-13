# Volume Design 5: Heterodyne — Colpitts BJT Oscillators + Diode Mixer, Beat-to-Voltage

Part of [VOLUME-DESIGNS.md](../VOLUME-DESIGNS.md)'s candidate set.
Diagram convention is documented in
[DIAGRAMMING.md](../DIAGRAMMING.md). Schematic source:
[diagram.mmd](diagram.mmd) — paste into
[mermaid.live](https://mermaid.live) or open with a Mermaid-aware
viewer/editor extension to render it.

**Mirrors [Design 5](../oscillator-5-colpitts-heterodyne/README.md).**
The Volume Fixed Oscillator, Volume Variable Antenna Oscillator and
Mixer subgraphs are Design 5's circuit, node for node, with the
variable oscillator wired to a volume antenna. The mixer's beat
frequency is what carries the volume: instead of going to a speaker it
goes to a frequency-to-voltage converter, whose DC output is the
control voltage the VCA expects (shared explanation in
[VOLUME-DESIGNS.md](../VOLUME-DESIGNS.md), contract in
[VOLUME-OSCILLATOR.md](../../notes/VOLUME-OSCILLATOR.md)). The two
designs mirror each other: a change to the oscillator pair or the
mixer in one should be carried over to the other. Either can be paired
with any pitch design.

This is the same architecture as the one complete all-hardware
theremin design I found published (DEAK Software's), in which the
volume path is a copy of the pitch path followed by a
frequency-to-voltage converter.

**What differs from Design 5:**

| | Design 5 (pitch) | Volume Design 5 | Why |
|---|---|---|---|
| Oscillator frequencies | Not specified in the diagram | Set well away from the pitch pair, by choosing different tank and feedback capacitor values | Two similar RF oscillators close together pull on and lock to each other |
| Mixer output | Pitch Signal Output, kept as audio | Junction, Mixer Output, then a second low-pass pole | The converter must not see any RF; the beat is only about 0 to 2 kHz |
| Second Low-Pass Filter | — | Resistor and capacitor, ~10 kΩ and ~1 nF | Adds a second pole so RF residue is far below the converter's input threshold |
| Added | — | LM2907-N frequency-to-voltage converter (7 parts) | Turns the beat frequency into the control voltage |

Total parts: 30 (Design 5's 21, plus 9).

**How it is meant to be adjusted.** Tune the volume pair so the beat
is zero with the hand at its *nearest* position (the silent end).
Withdrawing the hand then raises the variable oscillator's frequency,
the beat grows from zero, and the control voltage grows with it:
silence at the floor and loud with the hand away, which is the
polarity the VCA contract asks for, with no inverting stage. Because
the converter's output is proportional to the beat frequency, the
floor is a true 0 V at zero beat — the pedestal that sinks the
single-oscillator volume designs (1 to 3) does not exist here. (The
Diagram omits the zero-beat trimmer, as the pitch diagrams do; it is a
calibration addition for prototyping.)

**Performance — pros and cons (expected, not yet bench-verified):**
- **Pro:** The converter sees the *difference* frequency, not the
  oscillator frequency, so the volume signal starts at zero and
  carries the heterodyne sensitivity gain (see
  [PITCH-OSCILLATOR.md](../../notes/PITCH-OSCILLATOR.md)): a
  hand-induced shift of a few pF becomes a beat of up to a couple of
  kHz.
- **Pro:** A documented, complete precedent exists for the
  architecture (DEAK Software), and its converter, VCA and detector
  circuits are published.
- **Pro:** The converter's output is ratiometric to the supply (V_OUT
  = V_CC × f × C1 × R1, LM2907-N datasheet equation 5). With these
  values the output at a 2 kHz beat is about half the supply at any
  supply, which is where the VCA designs expect full scale.
- **Con:** Two oscillators and a mixer to build, tune and keep tuned,
  plus a converter: 30 parts. Drift in either oscillator shows up as a
  shift of the floor, the same drift the pitch pair has.
- **Con:** The volume pair and the pitch pair must be detuned from
  each other and kept apart (distance and shielding), or they will
  pull on one another. DEAK runs pitch at 1.06 MHz and volume at 715
  kHz; Seven Transistor Labs detunes with series capacitors in the
  tanks (470 pF on the volume oscillator, 1.5 nF on the pitch
  oscillator, none on the shared local oscillator).
- **Con:** Near zero beat two oscillators tend to lock, so the output
  may stay at zero over a small range. For volume that reads as a
  harmless silent zone; for pitch it is the familiar dead zone at the
  bottom of the range (Seven Transistor Labs names phase-locking
  distortion as a design challenge).
- **Con:** Ripple against speed. With the illustrative converter
  values the response time constant is about 24 ms and the converter's
  ripple is about 40 mV peak-to-peak (roughly 1% of full scale); a
  smaller ripple filter capacitor is faster and noisier. That 1% sits
  at the quiet end too, so it will matter for the VCA's quiet-end
  noise.
- **Con:** The most parts of any design in this repo (30), and the
  capacitive feedback dividers need well-matched capacitors, now in
  two more oscillators than Design 5 already has.

**Notes:**
- Two Junction bars are drawn where a node joins three or more passive
  leads and no IC pin or circle is there: the mixer's output node, and
  the node after the second filter pole where the filter capacitor and
  the converter's coupling capacitor meet. A junction is the
  breadboard column strip those leads share; see
  [DIAGRAMMING.md](../DIAGRAMMING.md), rule 12. It is not a part and
  isn't counted in the part total.
- Converter values: C1 = 1 nF, Pump Output Resistor (R1) = 240 kΩ,
  Ripple Filter Capacitor (C2) = 100 nF. The maximum input frequency
  is f_MAX = I2 / (C1 × V_CC), about 20 kHz at 9 V and 15 kHz at 12 V
  with I2 about 180 µA, far above a 2 kHz beat. Ripple is (V_CC/2) ×
  (C1/C2) × (1 − V_CC × f × C1 / I2) peak-to-peak and the time
  constant is R1 × C2.
- The converter's 8-pin input is referenced to ground and compares
  against ±25 mV (typical) with 30 mV of hysteresis. The beat is
  AC-coupled (330 nF with a 100 kΩ bias resistor, a corner near 5 Hz)
  so it swings about 0 V. The mixer diode's DC offset is blocked by
  the coupling capacitor. DEAK's schematic labels its
  envelope-detector output "0.5 Vp", which is comfortably above that
  threshold; DEAK puts a limiting amplifier in front of its converter,
  which this design leaves to the converter's own Schmitt-trigger
  comparator (an inference, not tested).
- RF rejection: each of the two filter poles has a corner near 16 kHz
  (10 kΩ and 1 nF), so each attenuates a 715 kHz carrier by about 45×,
  together about 2000×. A couple of volts of RF becomes about 1 mV,
  well under the converter's input threshold; a single pole would
  leave tens of millivolts and the converter would count RF cycles
  near each beat zero-crossing.
- The converter's internal op amp is wired as a unity-gain follower
  (pin 7, IN−, tied to pin 4, EMIT, with the load resistor to ground),
  exactly as in the datasheet's Figure 15. Pin 5 (COL) and pin 6 (V+)
  share the supply wire. No floor or gain conditioning is drawn,
  because the floor is already 0 V; the op amp's few-millivolt offset
  is a calibration matter.
- Design 5's diagram gives only the tank inductor value (about 40 µH,
  as in the Seven Transistor Labs build it follows) and leaves the
  feedback capacitors unvalued, so the frequency plan above is a
  choice to be made at the capacitors, not something the diagram
  fixes.
- Alternative detection, not drawn. Seven Transistor Labs mixes the
  volume oscillator with the shared local oscillator at an offset of
  tens of kHz and detects at that difference frequency with a resonant
  tank (4.7 mH with 0.022 µF, which by my arithmetic resonates near 16
  kHz) and a voltage-doubler Schottky detector, then uses the detected
  voltage to bias the pitch mixer. That removes one oscillator (the
  local oscillator is shared with pitch) but merges the volume
  detector into the pitch mixer, which breaks the separation between
  the volume subsystem and the VCA. It would be a candidate for a
  later "volume-6".
- The Feedback Capacitors, tank capacitors and mixer capacitors are
  frequency-determining, so the
  [CAPACITORS.md](../../notes/CAPACITORS.md) guidance (C0G/NP0 ceramic
  or polypropylene film) applies. Values shown are illustrative
  starting points, to be refined once we're actually prototyping.

**Sources:**
- [Building a Theremin, DEAK Software](https://deaksoftware.com.au/articles/building_a_theremin/)
  — a complete all-hardware theremin whose volume channel is a copy of
  its pitch channel (a pair of oscillators, a mixer and an envelope
  detector) feeding a frequency-to-voltage converter. Its full
  schematic and the envelope-detector, converter and VCA figures were
  read directly; the prose came only from a search-result summary (it
  states the volume path is functionally the same as the pitch path
  and that the pitch VFO can be detuned by up to about 1.9 kHz), so
  the frequency figures here (1.06 MHz pitch, 715 kHz volume) are read
  off the schematic captions.
- [Theremin, Seven Transistor Labs](https://www.seventransistorlabs.com/Theremin/)
  — read in full. Its volume channel uses a different detector (a
  resonant-tank slope detector on the difference frequency); see the
  notes.
- [LM2907-N / LM2917-N datasheet, Texas Instruments (SNAS555D)](https://www.ti.com/lit/ds/symlink/lm2907-n.pdf)
  — pinout, electrical characteristics, Figure 15, equations 2–7.
  Datasheet text and Figure 15 read directly.
- [Pat Tarantino, "Theremin" (UIUC Physics 406)](https://courses.physics.illinois.edu/phys406/sp2017/Student_Projects/Spring01/PTarantino/Pat_Tarantino_Theremin.pdf)
  — section 2.3: an LM2917 volume circuit fed by an oscillator pair
  that could not be brought to zero beat (a constant offset of about 1
  kHz), abandoned for that reason. Read in full for that section.
