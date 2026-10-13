# VCA Design 2: Operational Transconductance Amplifier (LM13700)

Part of [VCA-DESIGNS.md](../VCA-DESIGNS.md)'s candidate set. Diagram
convention is documented in [DIAGRAMMING.md](../DIAGRAMMING.md).
Schematic source: [diagram.mmd](diagram.mmd) — paste into
[mermaid.live](https://mermaid.live) or open with a Mermaid-aware
viewer/editor extension to render it.

An operational transconductance amplifier turns an input *voltage*
into an output *current*, and a separate bias current sets how much
current per volt. Feed the bias current from the control voltage and
the part is a voltage-controlled amplifier. The LM13700 adds
linearizing diodes at its inputs and Darlington output buffers, and is
the part TI's own datasheet shows as a volume control. This design
follows the datasheet's Figure 20 ("Voltage-Controlled Amplifier") and
adapts it from dual to single supply. Its interface is the one in
[VCA-DESIGNS.md](../VCA-DESIGNS.md): pitch audio in, control voltage
in (0 V silent, about half the supply full), scaled audio out.

**How the diagram works.**
- A Virtual Ground IC (TLE2426) makes half the supply; the OTA's
  inputs and output sit on it, so the audio swings both ways around
  it.
- The Input Resistor and the Balance Trimmer form the datasheet's
  input divider (30 kΩ and 1 kΩ, about 31:1), taking a 1 V peak input
  to about 33 mV peak-to-peak at the OTA. The datasheet's example
  needs the differential input under 60 mV peak-to-peak for under 0.1%
  distortion with the linearizing diodes. The trimmer's wiper goes to
  the virtual ground; the datasheet adjusts it "to minimize the
  effects of the control signal at the output", which is the
  control-feedthrough null.
- The Diode Bias Resistor feeds the linearizing diodes about 1 mA (the
  datasheet's recommended value, and required to exceed twice the
  signal current): about 5.3 kΩ for a 12 V supply with the diodes one
  drop above the 6 V virtual ground, so 5.1 kΩ.
- The Control Current Resistor turns the control voltage into the
  amplifier bias current. The bias pin sits two diode drops above the
  negative supply (about 1.2 V with that supply at ground), so the
  current is about (control voltage − 1.2 V) ÷ 9.1 kΩ, up to about 0.5
  mA at 6 V. The datasheet says the gain is linear with the applied
  control voltage.
- The OTA's output current flows through the Load Resistor to the
  virtual ground and straight into the Darlington buffer, whose output
  goes through the Output Coupling Capacitor; the Buffer Load Resistor
  is the datasheet's 5 kΩ to the negative supply.

**Performance — pros and cons (expected, not yet bench-verified):**
- **Pro:** A single through-hole part (DIP-16 in the TI datasheet)
  with a published voltage-controlled-amplifier circuit and a stated
  distortion target: under 0.1% for a 1 V peak input and ±15 V
  supplies, with the input held under 60 mV peak-to-peak.
- **Pro:** True cut-off at the bottom: below about 1.2 V of control
  voltage no bias current flows, so the stage is off, and the first
  fifth of the control range acts as a guard band against noise on the
  control voltage at the quiet end.
- **Pro:** Fast. The gain follows the bias current with no slow
  element in the path.
- **Con:** Supply. The datasheet's recommended single-supply range
  starts at 9.5 V (the dual-supply minimum is ±4.75 V), so a 9 V
  battery is below it; this design assumes 12 V. The datasheet also
  notes that ±5 V supply systems lose signal dynamic range because the
  PNP transistors have a higher base-emitter voltage than the NPNs.
- **Con:** The small input costs signal-to-noise: the datasheet notes
  that larger signals improve it, and this design starts from 33 mV
  peak-to-peak. The output swing is then restored by the Load
  Resistor, so gain at full control needs setting (the datasheet's
  Figure 21 equivalent circuit gives the relations; I did not derive
  them here).
- **Con:** Linear law. Gain in volts against control volts puts only a
  small part of the control range into the quiet end, where a player
  wants fine control (perceived loudness is roughly logarithmic; that
  is general psychoacoustics, not something this repo has measured).
  The exponential-law part,
  [VCA Design 3](../vca-3-that2180/README.md), fits that better.
- **Con:** One calibration, the Balance Trimmer, and the datasheet's
  own caution that it has to be adjusted to keep the control signal
  out of the output.
- **Con:** The LM13700 is a dual part and this uses half of it; ten
  parts in all, two of them ICs.

**Notes:**
- The channel used is amplifier A: pins 1 (Amp Bias Input), 2 (Diode
  Bias), 3 (Input+), 4 (Input−), 5 (Output), 7 (Buffer Input), 8
  (Buffer Output), with pin 11 (V+) and pin 6 (V−). Amplifier B's pins
  aren't shown — unused and out of scope, not omitted by mistake.
- Pin 5 and pin 7 share one wire because the datasheet connects the
  OTA's output directly to the buffer input.
- Values are the datasheet's Figure 20 where it gives them (30 kΩ
  input resistor, 1 kΩ pot, 30 kΩ load resistor, 5 kΩ buffer load);
  the diode bias and control resistors are re-derived for a 12 V
  single supply. The coupling capacitors are illustrative starting
  points, to be refined once we're actually prototyping.

**Sources:**
- [LM13700 datasheet, Texas Instruments (SNOSBW2F)](https://www.ti.com/lit/ds/symlink/lm13700.pdf)
  — pin functions, recommended operating conditions (single-supply and
  dual-supply ranges), sections 7.3.2, 7.4, 8.2 and 8.3.1, Figures 17
  to 21. Datasheet text and Figure 20 read directly; lifecycle status
  was not checked.
- [TLE2426 datasheet, Texas Instruments](https://www.ti.com/lit/ds/symlink/tle2426.pdf)
  — rail-splitter pins, input range and output accuracy. Datasheet
  text read directly; lifecycle status was not checked.
