# Volume: Turning a Second Antenna Into a Slowly-Varying Signal

**Status: preliminary, for review.** Nothing here is bench-verified.
Companion to [PITCH-OSCILLATOR.md](PITCH-OSCILLATOR.md). This note
covers the **volume subsystem**: oscillator, detector and smoothing
filter, ending in one control voltage. The antenna is a separate block
upstream, shared in concept with pitch (see
[ANTENNA-CAPACITIVE-SENSOR.md](ANTENNA-CAPACITIVE-SENSOR.md)). The stage that consumes
that voltage is in [VOLTAGE-CONTROLLED-AMP.md](VOLTAGE-CONTROLLED-AMP.md). None of Designs 1–5 in
[../circuits/OSCILLATOR-DESIGNS.md](../circuits/OSCILLATOR-DESIGNS.md)
has a volume antenna input — each diagram has exactly one `ANTENNA` node.

## Subsystem boundary

The volume subsystem sits between two boundaries. Upstream, the antenna
hands it a capacitance on a single wire; downstream, it ends at a single
DC voltage representing the volume level, where the amplifier (VCA)
starts. The subsystem does not know what kind of antenna feeds it or what
the voltage drives; the VCA does not know how the voltage is made. All
detection, filtering and calibration belong here; none of it leaks into
the antenna or the VCA.

**Input contract (from the antenna).** A capacitance to ground on one
wire into the oscillator's sensing node: a baseline value plus the
hand-dependent change. The expected baseline and change for a given
antenna geometry are not yet measured, and the tank has to be sized for
them. The zero-point trimmer is part of this subsystem.

Output contract (every value is proposed or TBD, for the reviewer):

| Property | Contract |
|---|---|
| Signal | Single-ended DC voltage, referenced to ground |
| Polarity | Higher voltage = louder (hand away = high) |
| Floor | Hand fully down gives a defined floor voltage the VCA treats as silence (proposed 0 V) |
| Full scale | TBD; set by the VCA's control range and the supply |
| Source impedance | Low enough to drive the VCA control input without loading; TBD |
| Speed | Follows the hand at articulation speed (tens of ms or faster); TBD |
| Noise and ripple | Below audibility at the floor once through the VCA; TBD |
| Law | Linear in the sensed level; the gain law belongs to the VCA (proposed) |
| Calibration | Zero-point trim and range set here, never in the VCA |

Testing alone: replace the antenna with a variable capacitor (or
trimmer) as a stand-in for hand and room, then record control voltage
against capacitance, the step response, and ripple on a scope or meter. Because
the contract is only a voltage, the same harness will later test a
software replacement of this stage.

## The job, and how it differs from pitch

Same physics as pitch (see
[ANTENNA-CAPACITIVE-SENSOR.md](ANTENNA-CAPACITIVE-SENSOR.md)): hand
capacitance on an antenna shifts an LC oscillator's frequency. What
changes is what the output must be.

| | Pitch | Volume |
|---|---|---|
| Output | Audio-range signal (beat frequency) | Conventionally a slowly varying DC level (a control voltage); see "Alternatives" below |
| Response wanted | Fine frequency resolution across several octaves | Fine, low-noise resolution from silence to full volume, and fast enough for articulation (see "Volume is an expressive control") |
| Hand near antenna | Pitch rises | Volume falls (conventional) |
| Heterodyne sensitivity gain | Essential | Open question: not essential for a usable level, but may help near-antenna resolution |
| Drift | Heard directly as detuning | Moves the "full volume" point; fixed by a trimmer |

## Design principle: the conversion chain

Heterodyning helps pitch because frequency *is* the signal: a tiny
frequency shift becomes a large beat-frequency shift. Volume needs
amplitude, so the same AC mechanism has to be converted back down. The
slope-detector route is capacitance → frequency shift → amplitude (via
the resonant slope) → DC control voltage, a conversion at each step.

The cost of that round trip is smaller than it looks. The last step is a
diode and an RC filter; the real costs are the filter's lag (which
affects articulation) and the ripple and noise it has to reject. The
resonant tank is also where the sensitivity comes from: a high-Q tank
turns a few pF into a large amplitude change on its slope, so skipping it
gives up gain rather than just a stage.

Routes with fewer conversions, and what each gives up:

| Route | Chain | Tradeoff |
|---|---|---|
| Slope detector on a resonant tank | C → frequency → amplitude → DC | Highest sensitivity; most conversions |
| Fixed-frequency AC through the antenna, rectified directly (capacitance-to-voltage style) | C → amplitude → DC | Fewer stages; no resonant gain, so much lower sensitivity (untested) |
| Microcontroller RC timing | C → time → digital value | Deferred: this phase is hardware only (see "Scope") |

The design question is therefore how much sensitivity to buy with extra
conversion, not whether conversion can be avoided. Which route fits
depends on the sensitivity and noise floor the expressive requirements
below demand.

## Frequency shift to control voltage (inside this subsystem)

Two candidate routes (decision open; see the open questions below):

- **Slope (resonance) detector.** The volume oscillator drives a fixed
  resonant LC circuit through a diode detector. At the oscillator's
  resting frequency the detector sees its peak voltage; a hand lowers the
  oscillator frequency off the peak and the DC output falls. This gives
  the conventional "hand near = quieter" behavior directly, and needs no
  second oscillator.
- **Beat plus frequency-to-voltage.** Heterodyne against a fixed
  oscillator as in the pitch pair, then convert the beat frequency to a
  voltage (e.g. a charge-pump or pulse-integrating converter). More parts
  and a more involved back end; reuses the existing mixer design.

Either way the detector needs a **smoothing filter**. Its time constant
is a tradeoff: too long makes volume lag the hand, too short passes RF or
beat ripple into the control voltage and thus through the VCA into the audio. Because
players use volume for articulation (staccato, attacks), lean toward the
fast end: a starting point is a few to tens of milliseconds, tuned by ear
against ripple, and possibly with a separate faster-attack, slower-release
filter. This is a guess, not a researched value.

## Volume is an expressive control, not a simple level

Earlier drafts of this note under-weighted this. The volume hand
carries the instrument's dynamics and articulation: players fade notes
in and out, shape attacks, and play staccato, accents and legato with it,
since the instrument has no natural attack or decay. The antenna is
described as sensitive enough for pianissimo through fortissimo. So the
requirements differ from pitch rather than being smaller:

- **Speed.** The detector filter must be fast enough for staccato and
  attacks; a slow filter smears them (see "Frequency shift to control voltage").
- **True silence and a clean quiet end.** Noise or ripple in the
  control voltage is most audible at pianissimo, and the amplitude stage
  must attenuate fully with the hand down.
- **Curve shape.** Perceived loudness is roughly logarithmic, so a
  response linear in voltage tends to feel cramped at the quiet end. This
  is from general psychoacoustics, not from a source in this repo; whether
  to shape the curve in the detector or the VCA is a reviewer question.
- **Resolution and adjustable range.** Fine control near the antenna,
  with a trimmer so the player can set where silence and full volume sit.

## Alternatives: must volume be DC, and must it come from an oscillator?

Neither is strictly required, but the reasons are worth recording.

- **Can the volume beat act as a carrier, or modulate the pitch wave's
  amplitude directly?** Not as an audio-rate signal. Multiplying two
  audio-rate signals is a ring modulator: the output is the sum and
  difference of the two frequencies, which gives bell-like, metallic
  timbres rather than a louder or quieter pitch. For the product to act
  as volume, the modulating signal must change slowly compared with
  audio. In practice it is smoothed to near-DC, so the oscillating
  signal ends up as a DC level anyway. (The ring-modulator timbre is
  sound design rather than volume control; see "Out of scope" below.)
- **Amplitude can be set directly, with no separate amplifier stage.**
  In the Seven Transistor Labs build the volume detector's output biases
  the pitch mixer, so the pitch wave's amplitude is set inside the mixer.
  The control signal is still smoothed DC (see [VOLTAGE-CONTROLLED-AMP.md](VOLTAGE-CONTROLLED-AMP.md)). The
  original tube circuit and the Silicon Chip build are described in the
  sources below as detecting the volume oscillator to DC and using it to
  control gain. Those descriptions came from search summaries, and no
  source was read for the Etherwave design, so don't generalise to "all
  classic designs".
- **Sensing capacitance needs an AC excitation, but not necessarily an
  antenna-tuned oscillator.** A capacitor passes no DC, so something
  must drive the antenna. Alternatives: a fixed oscillator driving a
  passive LC containing the antenna, detected by amplitude (slope
  detection); or timing an RC charge/discharge with a microcontroller, as
  the kit approach in PITCH-OSCILLATOR.md does. The first drops the
  antenna-tuned oscillator (and its pulling against the pitch pair) but
  still has an oscillator; the second drops analog oscillators entirely
  but is deferred (see "Scope").

## Scope: hardware only, and a pure source

**Hardware only, for now.** This phase is built exclusively in analog
hardware. Software and microcontroller approaches are deferred, not
rejected: a complete, working hardware design gives a reference to
reverse engineer, reproduce and swap software replacements into one
stage at a time, test them against the hardware, and possibly end with a
complete software copy. Microcontroller options elsewhere in these notes
are listed for that later phase only.

**The output of pitch and volume should be as pure as the components
allow.** The ambition is that the audio signal leaving the pitch/volume
intersection behaves as close as physically possible to a fixed
generator: the intended frequency, an amplitude set only by the volume
control, and nothing else added. Sound design (timbre) is deliberately
the *next* element in the chain, downstream of this one, switchable on
or off and with an adjustable amount, so it always operates on a source
that is as clean as possible. Ring and amplitude modulation at audio rate
(see "Alternatives") belong there, not here.

Consequences for this part of the instrument (measurable, once there is a
bench):

- Judge the amplitude-control stage on added distortion, linearity of
  gain, and control-voltage feedthrough (thumps or ripple leaking into
  the audio), not just on part count. See [VOLTAGE-CONTROLLED-AMP.md](VOLTAGE-CONTROLLED-AMP.md).
- Keep RF and beat ripple out of the control voltage, and keep the two
  channels from pulling each other (the detuning above).
- The reference waveform (a pure sine, or the pitch oscillator's own
  clean output) is not yet decided; the purity target needs one.
- Timbre stage requirements (a true bypass, an amount control) are for a
  later note.

## What carries over unchanged

- LC tank with the antenna as a single wire into the sensing node
  (Design 4's collector or Design 5's emitter lead), high-Q so a few pF
  of hand capacitance matters.
- C0G/NP0 or polypropylene tank capacitors ([CAPACITORS.md](CAPACITORS.md)).
- Reusing whichever of Designs 4/5 is chosen for pitch, rather than
  designing a separate oscillator, so both channels drift alike.

## Differences

- **Different frequency from the pitch pair.** Two similar RF oscillators
  near each other pull on and can lock to one another, which shows up as
  dead zones and jumps in pitch. Detune the volume oscillator from the
  pitch pair (different tank values) and keep the two physically apart
  and shielded. One reference build reports using deliberate detuning
  plus a series attenuating resistor on the pitch oscillator output.
- **Different antenna geometry.** The classic volume antenna is a
  horizontal loop, a different shape from the pitch rod. This is the
  antenna block's concern, not this subsystem's; see
  [ANTENNA-CAPACITIVE-SENSOR.md](ANTENNA-CAPACITIVE-SENSOR.md) for its one
  electrical consequence (different baseline and change, so different
  tank sizing).
- **Inverted sense is by design.** Volume is loudest with the hand away,
  so the detector must give maximum output at the
  oscillator's resting frequency.

## Simplifications

- **The fixed reference may not need to be an oscillator.** Classic
  designs detect the volume oscillator against a fixed *resonant circuit*
  (slope detection), so a passive LC replaces the fixed oscillator of the
  pitch pair. This is the main open design choice; see "Frequency shift to control voltage".
- No audible-range beat is required, so no audio output coupling from
  this stage.
- Less pressure for octave-spanning *frequency* range than pitch. (Not less pressure for sensitivity, speed or noise; see below.)

## Additions

- A **zero-point trimmer** (variable capacitor or inductor core) to set
  the resting frequency relative to the detector's resonance.
- A **detector and smoothing filter** to produce the control voltage (see "Frequency shift to control voltage").
- Shielding or spacing between channels.

## Open questions for the reviewer

1. Slope detector against a passive resonant circuit, or a second fixed
   oscillator plus beat-to-voltage conversion? The first matches the
   references below; the second reuses more of the existing designs.
2. How far apart should the volume and pitch frequencies sit? No numeric
   guidance has been researched yet.
3. If Designs 1–3 were chosen for pitch, is an LC volume oscillator still
   preferable? Probably yes (a RC relaxation volume channel has little
   resonance to detect against), but this is untested reasoning.

## Research links

- [Theremin — Seven Transistor Labs](https://www.seventransistorlabs.com/Theremin/) — the same build log cited in PITCH-OSCILLATOR.md; describes the volume channel as a variable oscillator, detuned from the pitch oscillator, feeding a slope/voltage-doubler detector. *These details came from a summarised fetch of the page, not a full read; the reviewer should confirm them against the page and its schematics.*
- [Theremin World — Silicon Chip Theremin Modifications](http://www.thereminworld.com/Article/14272/silicon-chip-theremin-modifications) and [How the Theremin Works (UAF)](http://ffden-2.phys.uaf.edu/211.fall2000.web.projects/Jennifer%20Erland/How%20it%20Works.html) — descriptions of the classic volume loop: antenna capacitance detunes an LC circuit, lowering its resonant response, which is detected to a DC value. *Seen only as search-result summaries; not yet read in full or checked against an engineering-level source per [RESEARCH-STANDARDS.md](RESEARCH-STANDARDS.md).*
- [Physics of the Theremin (Skeldon et al., 1998)](https://isidore.co/misc/Physics%20papers%20and%20books/Zotero/storage/3837CDJ6/Skeldon%20et%20al.%20-%201998%20-%20Physics%20of%20the%20Theremin.pdf) — the engineering-level cross-check already in PITCH-OSCILLATOR.md; not yet re-read for its volume-circuit treatment.
