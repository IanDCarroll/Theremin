# Voltage-Controlled Amplifier: Combining Pitch and Volume

**Status: preliminary, for review.** Nothing here is bench-verified.
Follows [VOLUME-OSCILLATOR.md](VOLUME-OSCILLATOR.md) and
[PITCH-OSCILLATOR.md](PITCH-OSCILLATOR.md). The VCA takes the volume
subsystem's control voltage as given: how that voltage is produced is
not this note's concern.

## Terminology: "mixer" already means something else

The intuition is right that pitch and volume meet in one place and come
out as a single audio waveform. The usual word for that stage is not
"mixer":

- **Mixer** in theremin circuits means the *heterodyne* mixer: the
  nonlinear stage (the diode in Designs 4 and 5) that combines two RF
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
| In | Control voltage from the volume subsystem (contract in [VOLUME-OSCILLATOR.md](VOLUME-OSCILLATOR.md), "Subsystem boundary") |
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
| Bias-gated mixer | Detector output biases the mixer transistor (Seven Transistor Labs approach) | Fewest parts. Design 4/5's passive diode mixer has no bias input, so it would be replaced by a transistor mixer. Volume curve is whatever the transistor does. Also merges the pitch mixer and the VCA, which breaks the boundary above (the control voltage would enter the pitch subsystem), so it cannot be tested as separate blocks |
| Photoresistor/opto cell | Control voltage drives an LED lighting an LDR in a voltage divider | Simple and smooth; slow and part-to-part variable |
| JFET as variable resistor | Control voltage sets a JFET's channel resistance in a divider | Cheap; distorts at larger signal levels |
| Transconductance amp (OTA) or VCA IC | Dedicated gain-control part, control voltage in, scaled audio out | Cleanest and most predictable; one IC plus support parts, and a specific part number has not been chosen |
| Microcontroller | Measure the volume channel digitally and set a digital pot or PWM level | Deferred: hardware only for this phase; a candidate for the later software-replacement phase |

Recommendation for a first build, given the pure-source ambition: **an
OTA/VCA IC (or another linear gain element)**, judged on distortion,
gain linearity and control-voltage feedthrough. The bias-gated mixer
removes a stage, but it makes the nonlinear mixer do the gain control, so
its distortion and volume curve come along with it; keep it as a
comparison build, not the default. (This reverses the earlier draft's
recommendation, which favoured part count.)

## Open questions for the reviewer

1. Is the bias-gated mixer's distortion and volume curve ever
   acceptable against the pure-source goal, or is it only a comparison
   build? Separately, the passive diode mixer's own output is nonlinear:
   how pure a pitch signal can it deliver before the VCA?
2. Which VCA part, if any, meets the part-selection criteria
   (through-hole, widely available)?
3. What does the pitch subsystem promise at its output (amplitude,
   impedance, DC offset)? The VCA's input contract is not yet written.
   Does the pitch signal need a buffer between mixer and VCA? Design 4's
   README already expects a weak, distorted mixer output.
4. Gain law: this note proposes the VCA owns it (so the control voltage
   stays linear in the sensed level; see the contract). Does the chosen
   part have a suitable (e.g. exponential) law, or should the volume
   subsystem shape the voltage?

## Research links

- [Theremin & FAQ — Carolina Eyck](https://www.carolinaeyck.com/theremin) and [Composing for Theremin — Charlie Draper](https://charliedraper.com/articles/2018/12/13/composing-for-theremin) — player-side descriptions of the volume hand as dynamics and articulation. *Search-result summaries only.*
- [Theremin — Seven Transistor Labs](https://www.seventransistorlabs.com/Theremin/) — slope/voltage-doubler detector biasing the pitch mixer. *From a summarised fetch; confirm against the schematics.*
- [How the Theremin Works (UAF)](http://ffden-2.phys.uaf.edu/211.fall2000.web.projects/Jennifer%20Erland/How%20it%20Works.html) and [Theremin World — Silicon Chip Theremin Modifications](http://www.thereminworld.com/Article/14272/silicon-chip-theremin-modifications) — classic volume chain: resonant LC detuned by the hand, detected to DC, driving a voltage-controlled amplifier. *Search-result summaries only; not read in full.*
- [Beat frequency oscillator — Wikipedia](https://en.wikipedia.org/wiki/Beat_frequency_oscillator) — background on the heterodyne mixer sense of "mixer".
