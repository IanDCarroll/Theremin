# Pitch Oscillator: Candidate Designs

Five candidate circuits for turning antenna capacitance into a
variable-frequency electrical signal (the job described in
[PITCH-OSCILLATOR.md](../notes/PITCH-OSCILLATOR.md)), each kept as simple as the
topology allows — no extras, no guard components beyond what the part
actually needs to function. The one deliberate exception is the 555
design's control-voltage bypass capacitor, called out where it appears,
because the 555 is genuinely unstable without it.

Diagram convention (net nodes, active-device nodes, passives as labeled
edges) is documented in [DIAGRAMMING.md](DIAGRAMMING.md). Component
values shown are illustrative starting points, to be refined once we're
actually prototyping. Where a capacitor is timing/frequency-determining,
assume the [CAPACITORS.md](../notes/CAPACITORS.md) guidance (C0G/NP0 ceramic or
polypropylene film) applies unless noted otherwise.

Each of the five has a **volume mirror**: the same oscillator circuit
copied onto a volume antenna and followed by a stage that turns its
hand-dependent AC into a DC control voltage. They are indexed in
[VOLUME-DESIGNS.md](VOLUME-DESIGNS.md), and the amplifiers that use that
voltage are in [VCA-DESIGNS.md](VCA-DESIGNS.md).

Each design lives in its own directory: a `README.md` with the prose
(context, rationale, notes) and a `diagram.mmd` with just the Mermaid
schematic source, kept separate so the machine-readable diagram isn't
mixed in with commentary. This page is the index and the side-by-side
comparison, ordered from technically least to most complex (by total
component count — see the table below).

## 1. [CMOS inverter (CD4069UB relaxation oscillator)](oscillator-1-cmos-inverter/README.md)

The simplest of the five: one gate, one resistor, one capacitor.

## 2. [555 timer (direct RC relaxation oscillator)](oscillator-2-555-timer/README.md)

The common kit approach: one oscillator, no mixer, antenna wired
directly into the timing network.

## 3. [Comparator relaxation oscillator (LM393)](oscillator-3-comparator-lm393/README.md)

A dedicated comparator IC in place of an op-amp, since op-amps aren't
generally built for open-loop comparator duty. More parts than the 555
or CMOS versions, mainly for the hysteresis network a comparator-based
oscillator needs to function at all.

## 4. [Heterodyne — RCA Armstrong oscillators, historical analog](oscillator-4-rca-historical/README.md)

The pitch half of the 1929 RCA AR-1264 theremin, built to Leon
Theremin's design (his patent, US 1,661,058, decides the architecture;
the RCA schematic supplies the circuit), with JFETs in place of the
triodes and two JFETs in series in place of the two-grid mixer tube.
Genuinely different from Design 5: Armstrong oscillators (coupled grid
and plate coils) instead of Colpitts, the antenna coupled through a
large series coil, and a two-input mixer in place of a diode. The
most parts of the five (28), so the numbering is by part count for
designs 1 to 3 and by kind for 4 and 5 (historical, then modern).

## 5. [Heterodyne — Colpitts BJT oscillators + diode mixer](oscillator-5-colpitts-heterodyne/README.md)

The general modern-parts version of a "real" theremin pitch circuit: two RF LC oscillators (one fixed, one
antenna-controlled), mixed to produce an audible beat frequency, using
a Colpitts topology (untapped inductor + capacitive divider) because
it's easier to wind a plain coil than a tapped one.

---

## Where these differ, at a glance

| Design | Architecture | Sensitivity | Active device(s) | Total parts |
|---|---|---|---|---|
| [1. CMOS inverter](oscillator-1-cmos-inverter/README.md) | Single RC relaxation oscillator | Lower | 1× CD4069UB gate | 3 |
| [2. 555 timer](oscillator-2-555-timer/README.md) | Single RC relaxation oscillator | Lower | 1× NE555 | 5 |
| [3. Comparator (LM393)](oscillator-3-comparator-lm393/README.md) | Single RC relaxation oscillator | Lower | 1× LM393 | 7 |
| [4. RCA heterodyne (historical analog)](oscillator-4-rca-historical/README.md) | Dual RF Armstrong oscillator + two-input mixer | High (heterodyne gain) | 4× JFET (2N5484) | 28 |
| [5. Colpitts heterodyne](oscillator-5-colpitts-heterodyne/README.md) | Dual RF LC oscillator + mixer | High (heterodyne gain) | 2× BJT (2N3904) | 21 |

"Total parts" counts every box/hexagon component in each design's
diagram — the objective measure behind the least-to-most-complex
ordering above.

Designs 4 and 5 are the only ones that reproduce the actual sensitivity
mechanism a "real" theremin relies on (see
[PITCH-OSCILLATOR.md](../notes/PITCH-OSCILLATOR.md)); 1–3 are simpler,
single-oscillator alternatives common in low-cost kits, trading some
expressiveness for far fewer parts.

## Performance, at a glance

Expected, not yet bench-verified — each design's own README has the
full pros-and-cons reasoning behind these one-line summaries.

| Design | Biggest strength | Biggest limitation |
|---|---|---|
| [1. CMOS inverter](oscillator-1-cmos-inverter/README.md) | Lowest power draw, widest supply tolerance | Least frequency-stable — threshold drifts with supply/temperature |
| [2. 555 timer](oscillator-2-555-timer/README.md) | Best-documented, most forgiving to build/debug | Internal thresholds are ratiometric to supply — drifts as a battery sags |
| [3. Comparator (LM393)](oscillator-3-comparator-lm393/README.md) | We set the switching thresholds ourselves | Most parts of the single-oscillator designs; thresholds still supply-relative |
| [4. RCA heterodyne (historical analog)](oscillator-4-rca-historical/README.md) | The original's circuit: full heterodyne sensitivity, a mixer that keeps the two oscillators separate | Six hand-wound coils (50 mH and 7 mH in the antenna branch) and 28 parts; unverified that the JFET oscillators start |
| [5. Colpitts heterodyne](oscillator-5-colpitts-heterodyne/README.md) | Full heterodyne sensitivity with only off-the-shelf parts | Most parts of the modern designs (21) — most assembly/wiring risk |

The general pattern: Designs 1–3 (single RC relaxation oscillators)
trade away the heterodyne sensitivity gain in exchange for being
simpler, cheaper, and considerably more frequency-stable to build and
tune correctly on a first attempt. Designs 4–5 (heterodyne) are the
only ones that behave like an actual theremin pitch-wise, but that
comes with real, structural costs: two oscillators that must be tuned
to and stay near each other, a lossy passive mixer that likely needs
downstream buffering, and — for Design 4 specifically — hand-wound
coils whose electrical properties depend on how well they're wound. None
of this is verified on a bench yet; it's what the topologies themselves
predict, and prototyping should either confirm or correct it.
