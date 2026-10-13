# Amplitude Control (VCA): Candidate Designs

Five candidate circuits for the stage that scales the pitch audio by
the volume subsystem's control voltage (the job described in
[VOLTAGE-CONTROLLED-AMP.md](../notes/VOLTAGE-CONTROLLED-AMP.md)),
ordered from fewest to most parts (the fifth is a historical analog,
see below). The control voltage comes from one
of the five designs in [VOLUME-DESIGNS.md](VOLUME-DESIGNS.md); the
audio comes from one of the pitch designs in
[OSCILLATOR-DESIGNS.md](OSCILLATOR-DESIGNS.md). The VCA does not know
how either is made, so every combination is meant to work, subject to
one level adaptation described below.

Diagram convention is documented in [DIAGRAMMING.md](DIAGRAMMING.md).
Component values shown are illustrative starting points, to be refined
once we're actually prototyping, and everything below is expected
behaviour, not bench-verified. Each design lives in its own directory
with a `README.md` and a `diagram.mmd`.

**Terminology.** "Mixer" in a theremin means the heterodyne mixer that
makes the pitch beat; this stage is a voltage-controlled amplifier. It
*multiplies* the audio by the control voltage, which a summing
amplifier cannot do.

## 1. [Opto, LED and photoresistor](vca-1-opto/README.md)

The simplest: the control voltage lights an LED, the LED lights a
photoresistor in series with the audio. Five parts and no supply in
the audio path, but slow and not quite silent.

## 2. [OTA, LM13700](vca-2-ota-lm13700/README.md)

An operational transconductance amplifier with linearizing diodes,
following the TI datasheet's own voltage-controlled-amplifier circuit,
adapted to a single supply. One through-hole IC plus a virtual-ground
IC.

## 3. [VCA IC, THAT 2180](vca-3-that2180/README.md)

A purpose-built Blackmer-type VCA IC with an exponential control law
and the best specified distortion of the four, working in current,
with an op amp to convert back.

## 4. [JFET attenuator](vca-4-jfet-attenuator/README.md)

A JFET as a voltage-controlled resistor in a divider, with an
inverting stage to drive its gate. Discrete and cheap, from the same
family as the oscillators' transistors, and the least pure of the
four modern ones.

## 5. [Historical analog of the RCA theremin's gain control](vca-5-rca-historical/README.md)

The 1929 instrument gave the first audio tube its plate current
through a second tube whose filament the volume circuit heated. Here a
bipolar transistor plays the second tube (a supply whose current the
control voltage sets), and JFETs, the closest modern parts to a
triode's grid-controlled behaviour, play the two audio tubes: a
starved preamplifier stage and an output stage. No virtual ground and
no op amp, and by design it adds a few percent of distortion. See
[TUBE-EMULATION.md](../notes/TUBE-EMULATION.md) for why.

**Why it is separate.** The other four are held to the repo's
pure-source goal; this one is a study of the original, and the
original's distortion and gain law are part of what it is. It still
meets the shared interface below, so it can be swapped for any of them.

---

## The interface every design shares (proposed)

| Interface | Proposed |
|---|---|
| Supply | A single supply, 12 V nominal. The LM13700's recommended single-supply range starts at 9.5 V, and the THAT 2180 needs at least ±4 V; the opto design needs none. Designs 2 to 4 take half the supply as a virtual ground from a TLE2426 rail splitter |
| Control voltage in | 0 V is silence; full volume is **half the supply** (6 V at 12 V); driven from a low-impedance source. The volume designs are built to deliver exactly this, and each VCA scales it to what its gain element needs, with resistors sized for 6 V |
| Pitch audio in | AC-coupled by the VCA's own input capacitor, so DC on the input does not matter. About 1 V peak-to-peak or less |
| Scaled audio out | AC-coupled, the same waveform with only the amplitude changed, at about the input's amplitude at full control |

Half the supply was chosen for full scale because it is also the
virtual ground's voltage, which makes the exponential THAT 2180's
unity-gain point fall exactly at full scale, and because the volume
converters' ratiometric output lands there at any supply.

## Pairing with the pitch designs

Every VCA preserves the input's amplitude at full volume (the opto
about −1 dB, the OTA to be set), so the question is whether the pitch
design's output is the right size to go in:

| Pitch design | Output at the VCA input | Opto | OTA | THAT 2180 | JFET |
|---|---|---|---|---|---|
| 1 to 3 (relaxation) | Rail-to-rail square, 9 to 12 V peak-to-peak, with a DC offset | Passes, but the output is then equally large | Needs about 10:1 attenuation first (the OTA's own divider is sized for 1 V) | Within the datasheet's signal-current rating (1.5 mA rms through 20 kΩ), not checked for distortion | Needs about 10:1 attenuation first (the JFET wants under about 0.5 V across it) |
| 4 and 5 (heterodyne) | Weak audio riding on the rectified RF level; roughly half a volt peak in the DEAK build's detector | Passes | Passes | Passes | Passes |

So Designs 1 to 3 want a small adapter (an attenuating divider,
AC-coupled) before the OTA or JFET, and Designs 4 and 5 may want a
gain stage if their level proves too low. **Those adapters are not
drawn yet**; the pitch designs stop at the oscillator or mixer, and I
did not add parts to them. Their natural home is the pitch side of the
interface, so that the VCAs stay independent of which pitch design
feeds them.

The historical analog (5) has its own divider (about 4:1) sized for
1 V peak-to-peak, and a starved JFET stage that distorts above about
125 mV at its gate, so it takes Designs 4 and 5 as they are, and
Designs 1 to 3 only after the same 10:1 adapter and a further
reduction to land at about 1 V. Its output at full volume is about half
its input, where the others aim for equal.

## Where these differ at a glance

| Design | Architecture | Gain law | Active device(s) | Total parts |
|---|---|---|---|---|
| [1. Opto](vca-1-opto/README.md) | LED and photoresistor in series with the audio | Set by the cell: nonlinear, part-dependent | LED, photoresistor | 5 |
| [2. OTA (LM13700)](vca-2-ota-lm13700/README.md) | Transconductance amplifier with linearizing diodes | Linear in control volts above about 1.2 V | LM13700, TLE2426 | 10 |
| [3. THAT 2180](vca-3-that2180/README.md) | Blackmer VCA IC plus current-to-voltage op amp | Exponential: linear in dB per volt | THAT 2180, TL072, TLE2426 | 11 |
| [4. JFET attenuator](vca-4-jfet-attenuator/README.md) | JFET as the lower arm of a divider, inverting gate drive | Strongly nonlinear, part-dependent | 2N5484, TL072, TLE2426 | 13 |
| [5. RCA historical analog](vca-5-rca-historical/README.md) | Current-starved supply feeding a JFET preamplifier and a JFET output stage, as in the 1929 instrument | Steep at the top and long at the bottom, like an audio taper (about −8 dB at 4 V, −19 dB at 2 V), above a dead zone of about 0.8 V (hand model) | 2× 2N5484, 1× 2N3904 | 17 |

## Performance, at a glance

Expected, not yet bench-verified — each README has the full reasoning
and its sources.

| Design | Biggest strength | Biggest limitation |
|---|---|---|
| [1. Opto](vca-1-opto/README.md) | Five parts, resistive audio path, no electrical path from control to audio | Slow decay (35 ms in the one datasheet read) and no true silence; photoresistors are getting hard to buy |
| [2. OTA (LM13700)](vca-2-ota-lm13700/README.md) | A published voltage-controlled-amplifier circuit, under 0.1% distortion in the datasheet's example | Needs 12 V (recommended single supply starts at 9.5 V); 33 mV input costs signal-to-noise; linear law cramps the quiet end |
| [3. THAT 2180](vca-3-that2180/README.md) | THD 0.01% to 0.05% maximum, pre-trimmed, exponential law | SIP-8 from one manufacturer; the control port wants a low-impedance drive |
| [4. JFET attenuator](vca-4-jfet-attenuator/README.md) | Cheap and discrete, with a published theremin precedent (DEAK) | Distortion is the weakest of the four, and pinch-off varies 10:1 unit to unit |
| [5. RCA historical analog](vca-5-rca-historical/README.md) | One supply, no IC, true silence at 0 V; the original's architecture with JFETs in the tubes' roles | A few percent distortion (by design), strong part-to-part spread (pinch-off, saturation current), a 0.8 V dead zone, a slow response, and values that are not derived from the original |

On the repo's purity goal
([VOLUME-OSCILLATOR.md](../notes/VOLUME-OSCILLATOR.md), "Scope"), the
order expected from the datasheets is the THAT 2180 first, then the
OTA, then the JFET, with the opto's distortion unknown and its speed
against it. This is a prediction from specifications; none of the four
has been measured.

## Not drawn yet

- The pitch-side adapters above.
- The output amplifier that follows the VCA, and the bypassable timbre
  stage that sits between them (see the notes).
- Any control-voltage smoothing beyond what the volume designs already
  do (an asymmetric fast-attack, slower-release filter is the usual
  trick; not researched here).
