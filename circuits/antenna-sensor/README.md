# Antenna Capacitive Sensor — Circuit Element

The second piece of the pitch-circuit puzzle, diagrammed separately
from the five oscillator designs in
[OSCILLATOR-DESIGNS.md](../OSCILLATOR-DESIGNS.md) so the two pieces can
be reasoned about — and interconnected — independently. This element
is the physical sensor; the oscillator designs are what turns its
output into a variable-frequency signal. The underlying physics (why a
hand near a conductor changes its capacitance at all) is covered in
[ANTENNA-CAPACITIVE-SENSOR.md](../../notes/ANTENNA-CAPACITIVE-SENSOR.md)
— this document only covers the circuit side. Diagram convention is in
[DIAGRAMMING.md](../DIAGRAMMING.md). Schematic source:
[diagram.mmd](diagram.mmd) — paste into
[mermaid.live](https://mermaid.live) or open with a Mermaid-aware
viewer/editor extension to render it.

## Why there's only one design here

Checked this against outside research rather than assuming it — every
real build found (see Research links) does the same thing: the antenna
is bare wire, connected directly to whichever node in the oscillator
needs to sense its capacitance. There's no dedicated "sensor circuit"
standing between the antenna and the oscillator — the antenna's
physical presence *is* the sensor. That matches every one of the five
oscillator designs already documented, each of which already draws the
antenna as a single wire straight into its sensing node (see each
design's own README for exactly where). This diagram just isolates that
same interface as its own piece, with a generic "To Oscillator" output
standing in for whichever specific design it eventually plugs into.

## Reading the diagram: Hand, Antenna, and what's actually flowing

The diagram has three nodes and two very different kinds of connection
between them, and the distinction matters:

- **Hand** is a circle — the true external actor. It's outside our
  build entirely: we don't wire it, solder it, or control it.
- **Antenna** is a box — the real, physical, buildable part (a rod or
  plate of conductive material we choose and mount).
- The **Hand–Antenna connection is dotted**, not solid, because there
  is no conductor there. The hand never touches anything; the coupling
  happens through the air, via the antenna's field (per
  [ANTENNA-CAPACITIVE-SENSOR.md](../../notes/ANTENNA-CAPACITIVE-SENSOR.md)).
  A solid line would wrongly imply a wire a person could solder between
  a hand and a circuit. See [DIAGRAMMING.md](../DIAGRAMMING.md)'s
  convention rule 11 for this as a general drawing rule, not a one-off.
- The **Antenna–oscillator connection is solid**, labeled "Charge/
  Discharge Path (driven by oscillator)" — this *is* a real wire, and
  it carries a real, if very small, current.

**What's actually moving, and where the power comes from.** It's easy
to read "the hand affects the antenna" as the hand somehow generating a
signal — it doesn't. No current or power originates at the hand or the
antenna. The oscillator's own active device (the 555's discharge
transistor, a CMOS gate, a comparator's output stage, or the BJT in the
heterodyne designs) is what actually charges and discharges the total
capacitance at the sensing node, drawing that current from the DC
supply rails — that loop never leaves the circuit. What the hand does
is purely passive: by capacitively coupling to the antenna, it changes
*how much* capacitance is at that node, which changes how long the
charge/discharge cycle takes, which changes the oscillation frequency.
The oscillating voltage this produces does have a tiny capacitive
"tail" reaching out through the antenna to the hand (a genuine, if
minuscule, displacement current) — but that's the *signal* extending
outward, not the *supply rails* extending outward. Vcc and Ground stay
inside the circuit; only the AC voltage they produce, at the sensing
node, has any presence beyond the antenna.

**Is that signal actually "AC," and does it ever become "DC" again?**
Yes to the first question, by the literal definition — AC just means a
voltage/current that varies periodically over time, not specifically
"resembles household mains power." What's happening here is much
closer to a clock or carrier signal than to mains AC: it carries timing
information, not delivered power, and (aside from small losses) a
capacitor gives back essentially all the energy it took in over one
full charge/discharge cycle — net power transfer across a cycle is
close to zero, unlike a mains-powered resistive load. On waveform
shape: Designs 1–3 (RC relaxation oscillators) produce a roughly square
wave, since the active device just switches hard between charging and
discharging; Designs 4–5's resonant LC tank rings much closer to a sine
wave. As for a DC-to-AC "conversion point" — there isn't a separate
component for that. The oscillator's active device switching the
battery's DC between the rails *is* the AC-generation mechanism; the
two aren't sequential steps. And within this signal path, it never
converts back to true DC — the mixer stage (Designs 4–5) shifts it down
to audio frequency but it's still AC, and it needs to stay AC all the
way to a speaker, since a speaker cone has to alternate to make sound.
The only point where this stops being "AC" in the ordinary sense is if
it's later digitized (an ADC or a microcontroller's digital input) —
which is a different regime (discrete, sampled logic levels), not a
reversion to constant-voltage DC.

## Two things that came up in research, and why they aren't drawn here

**An optional series resistor for static-discharge protection.**
General ESD-protection engineering practice recommends a small series
resistor ahead of a sensitive gate input, mainly as a current limiter
if a static-charged hand ever gets close enough to discharge into the
circuit. This is a real, legitimate consideration — but it's most
relevant specifically for [Design 1](../oscillator-1-cmos-inverter/README.md),
whose antenna wire lands directly on a raw CMOS gate input (CMOS inputs
are the most ESD-sensitive of anything in our five designs). It's not
drawn as part of the default design here because: (a) it's genuinely
optional, not required for the circuit to function, consistent with
our "no extras" approach elsewhere; and (b) the one independent build
found that most closely matches our own CMOS design (see Research
links) wires its antenna with a bare, unprotected connection and
reports it working fine. Worth revisiting if we see static-discharge
symptoms once this is actually breadboarded — at that point it becomes
a small (~1–10 kΩ) resistor in series between the Antenna and Design
1's gate input, nothing more.

**Antenna shape/geometry (rod, plate, loop).** Research turned up real
alternatives here — a vertical rod, a flat plate, and a loop are all
documented antenna geometries, and they meaningfully affect playing
ergonomics and field linearity (see Research links). None of that
changes the *wiring*: electrically, a plate and a rod both do exactly
the same job — one conductor, one wire, into the oscillator's sensing
node. So it doesn't produce a second circuit diagram. It does change two
numbers the circuit depends on, though: the antenna's baseline
capacitance and the size of the hand-dependent change. Those set how the
oscillator's tank has to be sized, and how the instrument feels to play
(see [ANTENNA-CAPACITIVE-SENSOR.md](../../notes/ANTENNA-CAPACITIVE-SENSOR.md),
"What the tank value means for the player"). So swapping geometry later
is a mechanical decision *and* a retuning exercise: not a different
diagram, but possibly different component values, and neither
number has been measured yet.

## Research links

- [The Pitch-Control Theremin — UBC Physics 420](https://phys420.phas.ubc.ca/p420_2021/kfdeane/) — a documented, illustrated student build using a CMOS RC oscillator (the same family as our Design 1); explicitly confirms the antenna is "connected by a wire" directly to the oscillator's gate input, with no intervening component, and gives practical guidance on antenna plate area/shape.
- [Theremin World — Plate antennae vs. pole and loop](http://www.thereminworld.com/Forums/T/28690/plate-antennae-vs-pole-and-loop) — community discussion of the real tradeoffs between antenna geometries (ergonomics, field behavior), confirming these are physical/playing-feel choices rather than circuit differences.
- [Theremin World — Antenna Alternatives?](http://www.thereminworld.com/Forums/T/26407/antenna-alternatives) — further community discussion of geometry options, useful for cross-checking the plate-vs-pole-vs-loop claims above against a second, independent source.
