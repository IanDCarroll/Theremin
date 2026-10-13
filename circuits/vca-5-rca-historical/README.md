# VCA Design 5: Historical Analog of the RCA Theremin's Gain Control

Part of [VCA-DESIGNS.md](../VCA-DESIGNS.md)'s candidate set. Diagram
convention is documented in [DIAGRAMMING.md](../DIAGRAMMING.md).
Schematic source: [diagram.mmd](diagram.mmd) — paste into
[mermaid.live](https://mermaid.live) or open with a Mermaid-aware
viewer/editor extension to render it.

This is the one **historical** entry in the set, and unlike the other
four it is allowed to colour the sound: the distortion and gain law it
produces come from the circuit's design, as they did in 1929. It is
built from the same references as the other historical designs
(Theremin's patent first, the RCA schematic where the patent is silent;
see [Oscillator Design 4](../oscillator-4-rca-historical/README.md)),
with modern transistors in the tubes' roles. Which transistor suits a
tube is discussed in
[TUBE-EMULATION.md](../../notes/TUBE-EMULATION.md); the short version
is that a JFET is much closer to a triode at the control electrode, so
every grid-controlled tube here is a JFET.

**What the original did.** In the RCA AR-1264 (1929), the first audio
stage (V4, a UY-227 triode) has its *plate supply* run through another
tube (V5, a UX-120 with its grid tied to its filament, so it acts as a
diode). V5's filament is heated by radio-frequency current from the
volume circuit; the hotter it is, the more current it can pass, so V5
is a variable current source feeding V4's plate. With the hand far from
the volume antenna, V5 is hot and V4 gets full plate current and
amplifies fully; with the hand near, V5 is cold, V4 is starved, and
nothing passes. The pitch audio enters V4's grid through a transformer,
and leaves through another to the output tube (V7, a UX-171A). There
is no rectifier and no separate control voltage: the filament's heating
is the conversion from radio frequency to DC, and the converter and the
gain control are fused in one part. Theremin's patent (Fig. 11) shows
the same arrangement: a radio-frequency coil in the filament circuit of
the amplifier tube whose emission the hand controls.

**What this design does with it.** The repo's boundary puts the
conversion to DC in the volume subsystem, so this design starts where
the filament's *heat* would be: the control voltage plays the filament's
role, and everything after it is the same idea.

| RCA element | This design |
|---|---|
| V5 (UX-120), emission from a heated filament, a current source into V4's plate | Supply Valve Transistor (a bipolar, see below) and its adjustable emitter resistor: control voltage in, plate-supply current out |
| Thermal inertia of V5's filament, plus a resistor and capacitor in V4's plate supply that the service notes say set "the volume control's time constant" | Control Lag Resistor and Capacitor (about 100 ms) |
| V4 (UY-227), cathode grounded, grid at zero volts DC through the transformer | Preamplifier JFET, source grounded, gate at zero volts DC through a gate resistor |
| T4's primary as V4's plate load, with C13 (0.5 µF) from the supply end to ground | Plate-Load-Style Drain Resistor and Plate Supply Bypass Capacitor |
| T7 and T4 coupling the audio in and out | Capacitors, with a resistive divider on the input; no transformers |
| R12 and V6's grid-leak tap holding V4 cut off at the nearest hand position | Nothing needed: with the control voltage below about 0.8 V the supply valve passes no current, so the preamplifier has none |
| V7 (UX-171A), the audio power amplifier tube | Output Stage JFET, run as a voltage amplifier |
| Loudspeaker, play/off switch | Not part of this stage |

**Why the supply valve is not a JFET.** The rule here is a JFET
wherever the tube is grid-controlled. V5 is not: it is a diode whose
current is set by *heat*, which no transistor has. What it contributes
is a current that rises with a control, and from zero. A depletion
JFET conducts at zero gate voltage and needs a negative gate to cut
off, which a single positive supply cannot give a control voltage that
starts at 0 V. A bipolar emitter follower does cut off at zero control,
which is why it stays here.

**How the diagram works.**
- The Supply Valve Transistor is an emitter follower. Its base follows
  the control voltage through the lag network; the emitter resistor
  turns the voltage left over into a current into the plate supply
  node. That current is the only thing that varies the volume. The
  resistor is adjustable because the preamplifier JFET's saturation
  current varies from part to part; it sets the *full-volume* current.
- The preamplifier JFET's gate sits at zero volts, so it would like to
  draw its full saturation current. It is only supplied less than that,
  so it is held in its ohmic region: its drain voltage, and with it its
  gain and the largest signal it can pass, fall together as the
  supply current falls. This is the "starved plate" of the original.
  The ohmic region is the one the JFET literature calls the "triode
  region".
- The Input Divider takes the pitch audio down by about four, because
  a starved JFET can swing only a few tenths of a volt at its drain.
- The Output Stage JFET is the output tube's role: a JFET common-source
  stage with an unbypassed source resistor and a drain resistor chosen
  by the Fetzer Valve rules (source resistor about 0.83 × the JFET's
  pinch-off voltage ÷ its saturation current, which is a published way
  of bringing a JFET's square law closer to a triode's 3/2 law). It
  restores the level the starved stage gave away.

**Expected figures, from a hand model with an assumed JFET (3 mA
saturation current, 1.5 V pinch-off) and the illustrative values shown.
Treat as an indication, not a prediction:**
- The preamplifier's gain from its gate to its drain, with the emitter
  resistor adjusted for a full-volume current of about 85% of the
  saturation current: about 0.7 at 6 V control, 0.27 at 4 V, 0.16 at
  3 V, 0.08 at 2 V. That is about −8 dB, −13 dB and −19 dB below full:
  a steep top end and a long quiet tail, a little like an audio taper.
- Silence below about 0.8 V control (the supply valve's base-emitter
  drop plus the small plate-supply voltage).
- With 125 mV peak at the gate (1 V peak-to-peak in), distortion of
  about 6% at full volume and 3% at lower settings, expected to be
  mostly second harmonic (a JFET's square law).
- With the output stage's gain of about 3 (the Fetzer formula:
  0.54 × (supply ÷ pinch-off − 2), about 3.2 here), the whole stage
  delivers about half the input's amplitude at full volume (about
  −5 dB).

**Performance — pros and cons (expected, not yet bench-verified):**
- **Pro:** It is the original's architecture, so what it does to the
  sound is informative about what a 1929 theremin did: the control acts
  through the stage's supply, not through a part in the signal path.
- **Pro:** True silence at 0 V control, by construction. No virtual
  ground and no op amp; all through-hole, breadboard-friendly parts.
- **Pro:** The gate resistor and coupling capacitor make a "grid-leak"
  input: a loud signal biases itself slightly negative through the
  gate's own diode, as a tube's grid does.
- **Pro:** The volume law is steep at the top and long at the bottom,
  which resembles an audio taper without any shaping parts.
- **Con (by design):** Distortion of a few percent, and more at loud
  settings. This is allowed here and not in the other four.
- **Con:** Part-to-part spread is the main risk. The 2N5484's
  saturation current is roughly 1 to 5 mA (from memory, not rechecked)
  and its pinch-off −0.3 to −2 V (InterFET's datasheet); in
  the hand model a part with a pinch-off near −0.8 V distorted 11 to
  32% and one near −2 V about 2 to 5%. Select parts with a pinch-off of
  at least about 1.2 V, and expect to adjust the emitter resistor
  (470 Ω to 3.3 kΩ in the model) for each. The output stage's gain
  varies with pinch-off too, about 2 to 7.
- **Con:** A dead zone: the bottom 0.8 V of the 0–6 V control range
  does nothing, and its edge moves with temperature by about 2 mV per
  °C.
- **Con:** Seventeen parts, the most of the five, though no IC.
- **Con:** The lag network makes it slow by design (about 100 ms), so a
  quick volume gesture is smoothed. The original was slow too (a hot
  filament cooling), but its figure is not in the service notes.

**Notes:**
- The model is quasi-static (it ignores capacitances) and uses a
  textbook square-law JFET with a gate diode clamp at +0.6 V. It says
  nothing about noise, temperature or the real 2N5484.
- Not found: the original's values for R9 to R12, C8, the UX-120's and
  UY-227's mu, transconductance and emission figures, and the filament
  voltage reaching V5. The values here are illustrative starting points
  chosen to make the hand model work; they are not derived from the
  original.
- Transformers are not reproduced. The original's T7 and T4 also
  filtered and impedance-matched the audio, which these capacitors and
  resistors do not.
- The Control Lag Resistor draws the supply valve's base current
  (about 10 µA), a drop of about 0.1 V at full scale; the emitter
  resistor's adjustment absorbs it.
- The output of this stage is about half of its input at full volume,
  where the other four VCAs aim for about equal. A VCA later in the
  chain, or the next stage, makes up the difference.
- Part counts: two JFETs, one bipolar and fourteen passives; two
  junctions (bars in the diagram) mark the plate supply node and the
  divider midpoint, where three wires meet and no part is there.

**Sources:**
- [The RCA Theremin, theremin.us (Arthur Harrison)](https://theremin.us/RCA/rca_theremin.html)
  — the volume circuit's operation, tube roles, the 420 kHz oscillator,
  and the redrawn schematic (page 2 of 2, read as an image) with R9 to
  R12, C7, C8, C10 to C13 and the transformers. Text and schematic read
  directly.
- [RCA Theremin Service Notes, thereminvox.com (Reid Welch)](https://thereminvox.com/stories/tech/rca-theremin-service-notes/6/)
  — the service notes' own description: the filament lit by pickup
  current, the plate supply through the UX-120, the condenser and
  resistor setting the volume control's time constant, and the tap on
  the grid leak. Read through a summary only.
- [US 1,661,058, L. S. Theremin](https://patents.google.com/patent/US1661058A/en)
  — Fig. 11 and 12 show the same volume arrangement: a radio-frequency
  coil (96) in the filament circuit of the amplifier tube whose
  emission the hand controls. Read as images from the patent PDF; the
  figure is small and I may have mis-traced some connections.
- [A closer look at the Fetzer Valve, Runoff Groove](https://runoffgroove.com/fetzervalve.html)
  — the JFET-as-triode source and drain resistor formulas and the gain
  formula used for the output stage. Read directly; they were derived
  for the 2N5457 and J201 at 9 to 18 V, not the 2N5484.
- [RCA UX-120, radiomuseum.org](https://www.radiomuseum.org/tubes/tube_120.html)
  and [UY-227](https://radiomuseum.org/tubes/tube_uy227.html) — heater
  data only (and the sources disagree on the UX-120's 3 V or 3.3 V);
  no mu or transconductance figures were found.
