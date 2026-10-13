# VCA Design 3: Blackmer VCA IC (THAT 2180)

Part of [VCA-DESIGNS.md](../VCA-DESIGNS.md)'s candidate set. Diagram
convention is documented in [DIAGRAMMING.md](../DIAGRAMMING.md).
Schematic source: [diagram.mmd](diagram.mmd) — paste into
[mermaid.live](https://mermaid.live) or open with a Mermaid-aware
viewer/editor extension to render it.

A purpose-built voltage-controlled amplifier IC: the THAT 2180 series
is a pre-trimmed Blackmer-type VCA whose gain follows its control
voltage *exponentially*, so the control is linear in decibels. It
works in current: the audio goes in through a resistor, comes out as a
current, and an op amp turns that current back into a voltage. Its
interface is the one in [VCA-DESIGNS.md](../VCA-DESIGNS.md): pitch
audio in, control voltage in (0 V silent, about half the supply full),
scaled audio out.

**How the diagram works.**
- A Virtual Ground IC (TLE2426) makes half the supply; the VCA's
  ground pin, its negative control port and the op amp's non-inverting
  input all sit on it. The datasheet's typical circuit uses ±15 V; on
  a 12 V single supply the same circuit is ±6 V about that point.
- Input: a 20 kΩ resistor into pin 1 (the datasheet's typical circuit,
  with a 10 µF coupling capacitor ahead of it). Output: pin 8 is a
  virtual-ground current output, wired to the op amp's inverting input
  with a 20 kΩ feedback resistor and a 22 pF capacitor (the datasheet
  says the capacitor is needed to cancel the output capacitance and
  keep most op amps stable). The result is a unity-gain, *inverted*
  signal at full control.
- Bias: the 2180's current comes through pin 5 (V−), which sits about
  2.85 V below the ground pin, so the Bias Set Resistor is (3.15 V ÷
  about 2.4 mA) ≈ 1.3 kΩ here, where the datasheet's 5.1 kΩ is for ±15
  V.
- Control: the datasheet's alternative sense is to ground the negative
  port (pin 3) and drive the positive port (pin 2), where a positive
  voltage means more gain, so no inverter is needed. At 0 V relative
  to the ground pin the gain is unity. The two Control Divider
  Resistors reference the control voltage to the virtual ground: with
  the control voltage at half the supply the positive port sits on it
  (unity gain), and at 0 V it sits 0.5 V below, which at the
  datasheet's roughly 6.1 mV per dB is about −82 dB.

**Performance — pros and cons (expected, not yet bench-verified):**
- **Pro:** The best-specified of the four: distortion (THD at 1 V, 1
  kHz, 0 dB) of 0.01% maximum for the 2180A, 0.02% for the B and 0.05%
  for the C, factory-trimmed so no adjustment is needed (pin 4 is left
  open to keep the trim), with control-voltage feedthrough also
  trimmed at the wafer.
- **Pro:** The control law is exponential — gain in decibels is linear
  in control volts — which is also how loudness is perceived (general
  psychoacoustics, not something this repo has measured), so equal
  hand movements give roughly equal loudness steps.
- **Pro:** Supply range of ±4 V to ±18 V (a minimum of 8 V total), so
  the 12 V single-supply design here is inside it.
- **Pro:** Fast; the gain follows the control voltage directly.
- **Con:** Available only as an 8-pin SIP (through-hole, 0.1 inch
  pitch, breadboard-friendly) from one manufacturer; its lifecycle and
  price were not checked.
- **Con:** At the floor it is about −82 dB, deep but not true silence
  (the exponential law has no zero).
- **Con:** The datasheet says to drive the positive control port from
  a low-impedance voltage source. The control divider here presents
  about 9 kΩ, which may need a buffer; unverified.
- **Con:** The minimum supply the datasheet allows depends on the
  signal and bias currents, and THAT notes that low-voltage operation
  needs care; a 9 V supply (±4.5 V) is close to the limit, so this
  design also assumes 12 V.
- **Con:** Two ICs plus the rail splitter, and a current-in,
  current-out part needs the op amp, 11 parts in all.

**Notes:**
- Pins: 1 Input, 2 Ec+, 3 Ec−, 4 Sym, 5 V−, 6 Gnd, 7 V+, 8 Output
  (Table 1 of the datasheet). Pin 4 is left unconnected (as the
  datasheet recommends) and so isn't drawn. Pin 6 is the VCA's
  reference and must not be used as an extra input.
- The op amp is a TL072 (TI datasheet: 4.5 V to 40 V single supply,
  0.003% typical distortion); the datasheet's own example is an OP275.
  The package has two amplifiers and this design uses one.
- The Input and Output Coupling Capacitors are not
  frequency-determining for timing. Values shown are illustrative
  starting points, to be refined once we're actually prototyping.

**Sources:**
- [THAT 2180 series datasheet, THAT Corporation (document 600029 rev 02)](https://thatcorp.com/datashts/THAT_2180-Series_Datasheet.pdf)
  — pin assignments, supply limits, bias current, control ports and
  sense, distortion and the typical circuit. Datasheet text and the
  typical-circuit figure read directly; lifecycle status was not
  checked.
- [TL072 datasheet, Texas Instruments](https://www.ti.com/lit/ds/symlink/tl072.pdf)
  and
  [TLE2426 datasheet, Texas Instruments](https://www.ti.com/lit/ds/symlink/tle2426.pdf)
  — supply range and distortion; rail-splitter pins, input range and
  output accuracy. Datasheet text read directly; lifecycle status of
  neither part was checked.
