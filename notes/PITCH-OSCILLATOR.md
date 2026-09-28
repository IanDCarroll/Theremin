# Pitch: Turning Antenna Capacitance Into a Variable-Frequency Signal

Research notes on the first half of a theremin's signal chain: how hand-
proximity-to-antenna capacitance becomes an electrical signal whose
*frequency* varies with hand position. Deliberately stops before that
signal becomes sound through a speaker — that's a separate, later
concern. This document picks up *after* the antenna capacitance already
exists — see [ANTENNA-CAPACITIVE-SENSOR.md](ANTENNA-CAPACITIVE-SENSOR.md)
for how the antenna and the player's body produce that capacitance in
the first place.

## The general concept

An antenna near a hand forms one plate of a capacitor (the hand/body is
effectively the other plate, through air as the dielectric). Moving the
hand changes that capacitance. Wire that antenna into the frequency-
determining part of an oscillator, and the oscillator's output frequency
now tracks hand position. That's the whole job: **variable capacitance
in → variable-frequency signal out.** Everything downstream (amplifying
it, converting it to sound) doesn't need to know or care how the
frequency got set.

There are two real families of circuit that do this job, and the
difference between them matters for sensitivity, not just complexity.

## Approach 1: heterodyne (beat-frequency) — what a "real" theremin does

This is Leon Theremin's original approach, and it's still what most
serious/expressive theremin builds use today, including transistor-based
hobbyist builds.

- Two **radio-frequency LC oscillators** (inductor + capacitor tank
  circuits, typically tuned to a few hundred kHz): one **fixed**, one
  whose tank capacitance includes the antenna (and therefore the hand).
- The two RF signals are mixed together. Mixing two close frequencies
  produces a **beat frequency** — the difference between them — which
  falls in the audible range even though both original signals are far
  above it (inaudible RF).
- As the hand approaches the antenna, the antenna-connected oscillator's
  frequency shifts slightly, which shifts the beat frequency, which is
  the pitch you hear.

**Why go through this trouble instead of just listening to one
oscillator directly:** a hand changes antenna capacitance by only a
tiny amount (a few picofarads), which only shifts an RF oscillator's
frequency by a tiny *percentage*. But because the audible output is the
*difference* between two close RF frequencies, that same tiny percentage
shift in one oscillator becomes a large percentage shift in the
(much smaller) beat frequency. Heterodyning is effectively a sensitivity
amplifier — it's what turns a barely-there capacitance change into a
wide, expressive, audible pitch range. This is also why the antenna
oscillator has to be a high-Q LC tank circuit and not just any RC
oscillator — sensitivity to the tiny capacitance change depends on it.

## Approach 2: direct RC relaxation oscillator — what most cheap kits do

This is the common 555-timer-based approach:

- A single oscillator (a 555 in astable mode, a CMOS logic inverter, or
  an op-amp wired as a comparator/Schmitt-trigger oscillator) whose
  frequency is set by a resistor-capacitor (RC) timing network rather
  than an LC tank.
- The antenna is wired directly into that RC network (as, or alongside,
  the timing capacitor), so hand capacitance directly shifts the
  oscillator's frequency — no second oscillator, no mixing stage.
- Because RC relaxation oscillators can be tuned to run directly in or
  near the audible range with ordinary component values, no
  down-conversion is needed.

Simpler to build (one oscillator, no mixer), but it skips the
sensitivity-amplifying trick heterodyning provides — the same tiny hand-
capacitance change produces a proportionally smaller frequency shift here
than it would feeding an RF heterodyne pair. This is the practical reason
555-timer kit theremins tend to feel less touch-sensitive/expressive than
transistor-based heterodyne builds — it's a real design tradeoff, not
just a difference in build quality.

## Can it be done with an op-amp?

Yes — an op-amp isn't tied to either approach, it's just a possible
substitute for the active/gain device in either one:

- **In the heterodyne (LC) approach**, an op-amp can serve the same role
  a transistor or vacuum tube does in a Colpitts or Hartley oscillator —
  providing the gain that sustains oscillation in the LC tank. This is
  less commonly seen in hobbyist writeups than transistor versions, but
  it's the same underlying oscillator topology with a different active
  device.
- **In the direct RC relaxation approach**, an op-amp can be wired as a
  comparator-based relaxation oscillator — functionally similar to how a
  555 timer or a CMOS logic inverter is used, just built from a more
  general-purpose part instead of a single-purpose timer IC.

So "555 timer vs. op-amp" isn't really the fork that matters — the fork
that matters is heterodyne-LC vs. direct-RC. The active device (tube,
transistor, op-amp, 555, CMOS gate) is a separate, mostly interchangeable
choice within either architecture.

## Leon Theremin's original (pre-555, 1920s) approach

Vacuum tubes (triodes) as the active/gain device in exactly the
heterodyne architecture described above: two LC oscillators, one fixed
and one whose tank capacitance included the antenna and (via body
capacitance) the performer's hand, mixed together to produce an audible
beat frequency. The 555 timer didn't exist until 1971 — Theremin's 1920
instrument achieved the same job using vacuum tube oscillators, decades
before transistors, op-amps, or ICs existed to do it any other way.

The core insight worth taking from this: **the underlying principle
(heterodyning two RF LC oscillators) hasn't changed since 1920.** What's
changed over time is only the active device used to sustain the
oscillation — vacuum tube → discrete transistor → op-amp/CMOS/555 (for
builds that keep the heterodyne architecture), or a shortcut to a
simpler single-oscillator RC design for low-cost/simplified kits.

## A likely candidate for what our kit's "flashed microprocessor" is doing

Worth flagging for later reverse-engineering: many modern low-cost kits
skip analog oscillators for pitch entirely and instead have a
microcontroller directly measure antenna capacitance (e.g. by timing how
long it takes an RC network to charge/discharge) and then digitally
synthesize the output tone (PWM or similar). If that's what our kit's
microcontroller is doing, it's a fourth approach — not on the same
spectrum as the three analog ones above — and would need to be reverse
engineered as "read capacitance digitally → compute frequency → generate
tone," rather than as any kind of analog oscillator. Something to check
once we start opening up the actual kit board.

## Research links

- [Theremin — Wikipedia](https://en.wikipedia.org/wiki/Theremin) — background, history, and a clear plain-language description of the heterodyne principle and antenna/hand capacitance.
- [Beat frequency oscillator — Wikipedia](https://en.wikipedia.org/wiki/Beat_frequency_oscillator) — the general beat-frequency mixing principle theremins rely on, independent of theremins specifically.
- [Capacitance, Heterodyning and The Strange Music of the Theremin — Mini-Circuits Blog](https://blog.minicircuits.com/capacitance-heterodyning-and-the-strange-music-of-the-theremin/) — accessible, illustrated (LC circuit diagram + frequency/capacitance equations) starting point; good for the capacitance-to-frequency intuition, though it doesn't spell out the two-oscillator beat-mixing mechanics in full — pair it with the Wikipedia articles above for that part.
- [Theremin — Seven Transistor Labs](https://www.seventransistorlabs.com/Theremin/) — detailed hobbyist engineering build log using a Colpitts LC heterodyne topology with discrete transistors; includes schematics, construction photos, and explicitly calls out polypropylene/polystyrene/C0G/silvered-mica capacitors for frequency stability — a nice independent confirmation of the capacitor-type guidance in [CAPACITORS.md](CAPACITORS.md).
- [Physics of the Theremin (Skeldon et al., American Journal of Physics, 1998, PDF)](https://isidore.co/misc/Physics%20papers%20and%20books/Zotero/storage/3837CDJ6/Skeldon%20et%20al.%20-%201998%20-%20Physics%20of%20the%20Theremin.pdf) — engineering/physics-audience depth, for cross-checking the beginner-level sources above against the full technical treatment.
- [Digital Theremin Circuit — Homemade Circuit Projects](https://www.homemade-circuits.com/digital-theremin-circuit-make-music-with-your-hands/) — a heterodyne build using CMOS logic (CD4069) and a PLL chip (CD4046) as the mixer instead of a dedicated RF mixer stage; useful as a middle ground between the classic analog heterodyne circuit and a fully digital one.
- [555 Theremin — Hackaday.io](https://hackaday.io/project/183538-555-theremin) — a practical direct-RC (single 555 oscillator) build, illustrating the simpler kit-style approach described above.
- [Hartley Oscillator: Working and Design using Op-Amp — ElectronicsHub](https://www.electronicshub.org/hartley-oscillator/) — confirms both Hartley and Colpitts LC oscillator topologies can be implemented with an op-amp as the active device, same principle as tube/transistor versions.
