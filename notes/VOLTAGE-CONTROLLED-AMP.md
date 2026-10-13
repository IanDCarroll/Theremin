# Voltage-Controlled Amplifier: Combining Pitch and Volume

**Status: preliminary, for review.** Nothing here is bench-verified.
Follows [VOLUME-OSCILLATOR.md](VOLUME-OSCILLATOR.md) and
[PITCH-OSCILLATOR.md](PITCH-OSCILLATOR.md). The VCA takes the volume
subsystem's control voltage as given: how that voltage is produced is
not this note's concern. Five candidate designs (four modern, one historical) are drawn in
[../circuits/VCA-DESIGNS.md](../circuits/VCA-DESIGNS.md); this note is the
reasoning behind them.

## Terminology: "mixer" already means something else

The intuition is right that pitch and volume meet in one place and come
out as a single audio waveform. The usual word for that stage is not
"mixer":

- **Mixer** in theremin circuits means the *heterodyne* mixer: the
  nonlinear stage (the diode in Design 5, the two-input JFET stage in Design 4) that combines two RF
  oscillators to produce their difference frequency. It is already in
  the pitch design and has nothing to do with volume.
- The stage where volume acts on pitch is a **voltage-controlled
  amplifier (VCA)**, also called a gain-control or amplitude-control
  stage, or an *amplitude modulator*. It **multiplies** the pitch audio
  by the volume control voltage.

**Why a summing op-amp "mixer" does not work:** a summing amplifier
*adds* signals. Adding a DC control voltage to the pitch audio only shifts
its DC offset; the loudness is unchanged. Volume needs the audio's
amplitude scaled by the control voltage, which is multiplication.
(An op-amp is still useful as the buffer or output driver around the VCA.)

One real overlap: in the Seven Transistor Labs build the heterodyne
mixer is itself the volume control, because its bias comes from the
volume detector, so the mixer conducts only as the volume channel allows.
The two words coincide there only because one circuit does both jobs.

## Signal chain

```
[ pitch subsystem:  antenna → pitch pair → heterodyne mixer → buffer ]──audio──┐
                                                                               ├→ [ VCA ] → [timbre, later] → output amp → speaker
[ volume subsystem: antenna → oscillator → detector → filter ]──control voltage┘
```

The VCA output is the "pure source" point: sound design (timbre) is a
later, bypassable stage downstream of it, so it always works against a
signal as close as possible to a fixed generator (see
[VOLUME-OSCILLATOR.md](VOLUME-OSCILLATOR.md), "Scope"). This phase is
hardware only.

## Interface

The VCA is a block with two inputs and one output, and nothing else:

| | Signal |
|---|---|
| In | Audio from the pitch subsystem |
| In | Control voltage from the volume subsystem (contract in [VOLUME-OSCILLATOR.md](VOLUME-OSCILLATOR.md), "Subsystem boundary"): 0 V silent, half the supply full scale (proposed) |
| Out | The same audio, scaled by the control voltage, and nothing else added |

It does not know how the control voltage is made (it could be a bench
potentiometer), and has no calibration of its own for the volume
channel's zero or range. That separation lets it be tested alone: feed a
function-generator sine plus an adjustable DC voltage, then measure gain
against control voltage, distortion, full attenuation at the floor
voltage, and feedthrough (thumps or ripple in the output with the audio
input grounded).

## Amplitude control options

Options from simplest, all candidates to verify against
[PART-SELECTION-CRITERIA.md](PART-SELECTION-CRITERIA.md):

| Option | How it works | Notes |
|---|---|---|
| Bias-gated mixer | Detector output biases the mixer transistor (Seven Transistor Labs approach) | Fewest parts. Design 5's passive diode mixer has no bias input, so it would be replaced by a transistor mixer. Volume curve is whatever the transistor does. Also merges the pitch mixer and the VCA, which breaks the boundary above (the control voltage would enter the pitch subsystem), so it cannot be tested as separate blocks |
| Photoresistor/opto cell | Control voltage drives an LED lighting an LDR in a voltage divider | Simple and smooth; slow and part-to-part variable. Drawn: [VCA Design 1](../circuits/vca-1-opto/README.md) |
| JFET as variable resistor | Control voltage sets a JFET's channel resistance in a divider | Cheap; distorts at larger signal levels. Drawn: [VCA Design 4](../circuits/vca-4-jfet-attenuator/README.md) |
| Transconductance amp (OTA) | Gain-control current in, scaled audio out | Drawn with the LM13700: [VCA Design 2](../circuits/vca-2-ota-lm13700/README.md) |
| VCA IC | Dedicated Blackmer-type gain cell with an exponential control law | Drawn with the THAT 2180: [VCA Design 3](../circuits/vca-3-that2180/README.md) |
| Plate-supply (current-starved) amplifier | A control-set current feeds the audio stage's supply; the stage's gain follows its current. What the 1929 RCA theremin did, with a filament-heated tube as the current source | A few percent distortion; adds a dead zone and a strong part-to-part spread. Drawn as a historical analog with JFETs in the audio tubes' roles and a bipolar for the supply: [VCA Design 5](../circuits/vca-5-rca-historical/README.md); the tube-emulation reasoning is in [TUBE-EMULATION.md](TUBE-EMULATION.md) |
| Microcontroller | Measure the volume channel digitally and set a digital pot or PWM level | Deferred: hardware only for this phase; a candidate for the later software-replacement phase |

Recommendation for a first build, given the pure-source ambition: **an
IC gain element**, judged on distortion, gain law and control-voltage
feedthrough. Of the four drawn, the datasheets predict the THAT 2180
first (0.01% to 0.05% maximum THD at 1 V, with an exponential law that
suits loudness), then the LM13700 (under 0.1% in the datasheet's example
with the input held to 60 mV peak-to-peak), then the JFET (typically
under 3% for under 0.5 V across the JFET in one application note), with
the opto's distortion unknown and its decay (35 ms in the one datasheet
read) against it. These are predictions from specifications, not
measurements. The bias-gated mixer removes a stage, but it makes the
nonlinear mixer do the gain control, so its distortion and volume curve
come along with it, and it breaks the volume/VCA separation; it stays a
comparison build and is not drawn.

## Open questions for the reviewer

1. Is the bias-gated mixer's distortion and volume curve ever
   acceptable against the pure-source goal, or is it only a comparison
   build? Separately, the passive diode mixer's own output is nonlinear:
   how pure a pitch signal can it deliver before the VCA?
2. Lifecycle and supply of the four drawn designs' key parts were not
   checked: the THAT 2180 (one manufacturer, SIP-8), the 2N5484 (one
   source calls it out of production), and photoresistors (an EU
   cadmium ban, per a 2023 paper). Which meet the part-selection
   criteria?
3. The pitch subsystem's output contract is now sketched in
   [../circuits/VCA-DESIGNS.md](../circuits/VCA-DESIGNS.md) (about 1 V
   peak-to-peak or less, AC-coupled), with Designs 1 to 3 needing a
   roughly 10:1 attenuator and Designs 4 and 5 possibly a gain stage;
   the adapters are not drawn. Is that the right split, with adapters on
   the pitch side?
4. Gain law: this note proposed the VCA owns it, and the four drawn
   designs have four different laws (opto and JFET nonlinear and
   part-dependent, OTA linear, THAT 2180 exponential). Is the
   exponential law the target, with the others judged against it?
5. The THAT 2180's positive control port is meant to be driven from a low
   impedance; the design's control divider presents about 9 kΩ. Does it
   need a buffer?

## Research links

- [Theremin & FAQ — Carolina Eyck](https://www.carolinaeyck.com/theremin) and [Composing for Theremin — Charlie Draper](https://charliedraper.com/articles/2018/12/13/composing-for-theremin) — player-side descriptions of the volume hand as dynamics and articulation. *Search-result summaries only.*
- [Theremin — Seven Transistor Labs](https://www.seventransistorlabs.com/Theremin/) — slope/voltage-doubler detector on the volume mixer's difference frequency biasing the pitch mixer; the whole page and both schematics were read for the volume designs.
- The datasheets and application notes behind the five VCA designs are listed in each design's README under Sources.
- [The RCA Theremin — theremin.us](https://theremin.us/RCA/rca_theremin.html) and the [RCA service notes — thereminvox.com](https://thereminvox.com/stories/tech/rca-theremin-service-notes/6/) — the 1929 volume chain: a 420 kHz oscillator, a passive loop tuned below it that the hand detunes, a pickup coil lighting a diode-connected tube's filament, and that tube's emission supplying the first audio tube's plate. Page and schematic read directly (service notes through a summary). Note this is *not* the "detected to DC, then drives a VCA" chain in the next entry: the conversion and the gain control are fused in the filament.
- [How the Theremin Works (UAF)](http://ffden-2.phys.uaf.edu/211.fall2000.web.projects/Jennifer%20Erland/How%20it%20Works.html) and [Theremin World — Silicon Chip Theremin Modifications](http://www.thereminworld.com/Article/14272/silicon-chip-theremin-modifications) — classic volume chain: resonant LC detuned by the hand, detected to DC, driving a voltage-controlled amplifier. *Search-result summaries only; not read in full.*
- [Beat frequency oscillator — Wikipedia](https://en.wikipedia.org/wiki/Beat_frequency_oscillator) — background on the heterodyne mixer sense of "mixer".
