# Volume Design 4: RCA Detuned-Loop Volume Channel, Historical Analog

Part of [VOLUME-DESIGNS.md](../VOLUME-DESIGNS.md)'s candidate set.
Diagram convention is documented in
[DIAGRAMMING.md](../DIAGRAMMING.md). Schematic source:
[diagram.mmd](diagram.mmd) — paste into
[mermaid.live](https://mermaid.live) or open with a Mermaid-aware
viewer/editor extension to render it.

**What this is a copy of.** The volume half of the RCA AR-1264
theremin (1929), whose volume channel is *not* a second heterodyne
pair. It is one oscillator (about 420 kHz) and a passive loop, tuned
just *below* the oscillator, that includes the antenna. The hand adds
capacitance, which lowers the loop's resonance further from the
oscillator, so less radio-frequency current flows in the loop. A
pickup coil coupled to the loop carries that current to a tube
filament in the original, and the filament's heat sets the plate
current of the first audio tube. Theremin's patent shows the same
scheme (Fig. 11: an oscillator, a resonant loop on the volume antenna
coupled to it, and a coil carrying the radio-frequency current to the
cathode circuit of the amplifier tube). For the reference hierarchy
(patent first, RCA's schematic where the patent is silent), see
[Oscillator Design 4](../oscillator-4-rca-historical/README.md).

**What differs from the original, and where the boundary is.** The
original has no separate control voltage: the filament's heating is
both the conversion to DC and the gain control. The repo's boundary
puts the conversion here and the gain in the VCA, so this design
replaces the filament with the closest modern parts, a diode and a
capacitor, and stops at the control voltage that
[VCA Design 5](../vca-5-rca-historical/README.md) (or any VCA)
takes.

| RCA element | This design |
|---|---|
| V6, UX-171A, Armstrong oscillator at about 420 kHz | The same JFET oscillator as the pitch reference oscillator, node for node, with values for 420 kHz |
| C10 (135 pF variable, front-panel "Volume") | Volume Tuning Capacitor |
| T3 primary (single-layer coil, as large as the antenna allows) in series with C8 (2200 pF) from the oscillator's grid to the volume antenna | Antenna Coil, Loop Coupling Capacitor and the Volume Antenna |
| T3 secondary heating V5's filament | Pickup Coil, with a diode and capacitor taking the radio-frequency current to DC |
| V5's emission, a current source | Not here: the diode's DC output is the control voltage |
| R10 and R11 taking a small negative bias from V6's grid circuit to cut V4 off | Not needed: VCA Design 5 passes nothing below about 0.8 V |

**How the diagram works.**
- The oscillator is the pitch design's reference oscillator (same
  parts, same names), with values chosen for 420 kHz.
- The loop is tuned *below* the oscillator (an antenna coil of about
  10 mH with about 15 pF of antenna capacitance resonates near
  411 kHz) on purpose: the original's notes say that exact resonance
  would overload the oscillator and destabilize it, and that working
  below the oscillator's frequency gives a consistent relation between
  hand position and volume. With the hand away, the loop is closest to
  resonance and carries the most current; with the hand near, the
  resonance drops further away and the current falls.
- The pickup coil's radio-frequency voltage is rectified by the
  Detector Diode and stored on the Reservoir Capacitor. The Level
  Potentiometer is both the load (so the level falls when the current
  falls) and the divider that sets full scale. Hand away gives a high
  voltage, hand near gives a low one, so the polarity the VCA wants
  (more control voltage means louder) needs no inversion.
- The control voltage's contract (0 V silence, full scale about 6 V at
  a 12 V supply) is in [VOLUME-OSCILLATOR.md](../../notes/VOLUME-OSCILLATOR.md).

**Performance — pros and cons (expected, not yet bench-verified):**
- **Pro:** The original's sensing method: one oscillator, no second
  oscillator to keep apart from the pitch pair, and no frequency-to-
  voltage converter.
- **Pro:** Sixteen parts, of which the oscillator is half, and one
  hand-wound loop to tune.
- **Pro:** The detector has no pedestal: with no current in the loop the
  output is at zero, rather than a constant with a small change on top
  as with a frequency counted from a relaxation oscillator (see
  [VOLUME-DESIGNS.md](../VOLUME-DESIGNS.md)).
- **Con:** This is slope detection, which needs a small-capacitance,
  high-Q loop (the original's single-layer coil) and, as worked out in
  VOLUME-DESIGNS.md, does not reach zero unless the hand's detuning is
  several loop bandwidths. The output will probably not reach 0 V with
  the hand at its nearest position; VCA Design 5's dead zone below
  about 0.8 V hides some of that, as would the OTA design's below about
  1.2 V, and no other VCA's does.
- **Con:** The detected voltage depends on the oscillator's amplitude
  and the supply as well as on the hand, so it will drift. The
  original also depended on its filament's heat and supply voltage.
- **Con:** Whether the pickup can give about 7 V peak at full volume
  (the contract's 6 V plus the diode drop) from a 12 V JFET oscillator
  is unverified; the original had power to spare (it lit a filament).
  The Pickup Coil's turns, and a larger supply on the oscillator if it
  comes to that, are the levers.
- **Con:** Speed is set by the Reservoir Capacitor against the
  potentiometer (about 10 ms) and then by the VCA's own lag; the
  original's was slow (a filament cooling).

**Notes:**
- **Values are not from the service notes.** The oscillator's coils
  (0.15 mH grid and 0.09 mH plate) give about 420 kHz with the 500 pF
  and adjustable 100 pF plate-to-grid capacitors by hand arithmetic; the
  10 mH antenna coil is chosen to put the loop just below that. The
  notes give the loop coil's DC resistance (183 Ω) and the capacitors
  shown on the schematic.
- The Detector Diode is a Schottky (1N5711) for its low drop, so that a
  weak current registers; the patent calls crystal detectors "generally
  preferable" for its heterodyne detectors. This is a modern
  substitution and unchecked.
- With the pickup coil's second end on ground, the control voltage is
  referenced to ground, as the VCA wants.
- This design shares nothing electrically with the volume designs 1 to
  3 and 5, which count a frequency. It does share the repo's contract
  with them and with the VCAs.

**Pitch mirror.** The oscillator subgraph is
[Oscillator Design 4](../oscillator-4-rca-historical/README.md)'s
reference oscillator, node for node. The rest differs (a loop and
detector instead of an antenna branch, variable oscillator and mixer),
because the original's two channels are not alike.

**Sources:**
- [US 1,661,058, L. S. Theremin](https://patents.google.com/patent/US1661058A/en)
  — Fig. 11 and 12 (volume electrode, resonant loop, coil carrying
  radio-frequency current to the amplifier tube's cathode circuit), read
  as images from the patent PDF.
- [The RCA Theremin, theremin.us (Arthur Harrison)](https://theremin.us/RCA/rca_theremin.html)
  — the volume circuit's description, the 420 kHz oscillator, the loop
  below the oscillator and the redrawn schematic.
- [RCA Theremin Service Notes, thereminvox.com](https://thereminvox.com/stories/tech/rca-theremin-service-notes/6/)
  — the original notes' description of the loop, the filament and the
  time constant, read through a summary.
