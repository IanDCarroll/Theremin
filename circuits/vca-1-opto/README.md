# VCA Design 1: Opto (LED and Photoresistor)

Part of [VCA-DESIGNS.md](../VCA-DESIGNS.md)'s candidate set. Diagram
convention is documented in [DIAGRAMMING.md](../DIAGRAMMING.md).
Schematic source: [diagram.mmd](diagram.mmd) — paste into
[mermaid.live](https://mermaid.live) or open with a Mermaid-aware
viewer/editor extension to render it.

The simplest of the four: the control voltage lights an LED, the LED
lights a photoresistor, and the photoresistor sits in series with the
audio ahead of a load resistor. More light, less resistance, more
signal. There is no supply in the audio path and no active device in
it at all. Its interface is the one in
[VCA-DESIGNS.md](../VCA-DESIGNS.md): pitch audio in, control voltage
in (0 V silent, about half the supply full), scaled audio out.

**Performance — pros and cons (expected, not yet bench-verified):**
- **Pro:** Five parts, no supply connection, no virtual ground. The
  audio path is a capacitor and a variable resistor.
- **Pro:** The controlled element is a resistor, so it should add very
  little distortion at audio levels, and there is no electrical path
  from the control side to the audio (the LED and cell are coupled
  only by light). I did not find measured distortion figures for a
  photoresistor at audio levels, so this is reasoning, not a number.
- **Pro:** Gain rises with control voltage, which is the polarity the
  control-voltage contract asks for, with no inverting stage.
- **Con:** Speed. Cadmium-sulphide cells and the LEDs that drive them
  are slow and have memory: published response times run from tens of
  microseconds for the turn-on to hundreds of milliseconds for the
  turn-off, with the turn-on faster the brighter the light (the DAFx
  2023 vactrol paper). The one commercial part with a datasheet I
  read, the Vactrol VTL5C3, lists a 2.5 ms rise (time to 63%) and a 35
  ms decay. The decay is what limits staccato.
- **Con:** No true silence. Gain with the LED off is the load resistor
  against the cell's dark resistance: 47 kΩ against the VTL5C3's 10 MΩ
  off-resistance is about −46 dB, and that dark resistance is
  specified ten seconds after the light goes out, so right after a
  loud note the quiet end is higher.
- **Con:** Control law set by the cell, not by us: cell resistance
  against LED current is nonlinear and varies part to part, and a red
  LED conducts nothing below about 1.8 V, so the bottom of the control
  range does nothing at all.
- **Con:** At full volume the cell's lit resistance is a few kΩ (the
  VTL5C3: 4 to 7 kΩ at 0.5 mA) in series with the 47 kΩ load, about −1
  dB, so this design never quite reaches unity gain, and its output
  impedance is up to the load resistor's 47 kΩ. The stage after it
  should have an input impedance of several hundred kΩ.
- **Con:** Availability. Photoresistors are cadmium sulphide, and the
  DAFx 2023 vactrol paper says vactrols are becoming difficult to
  find: Perkin Elmer stopped making them in 2010, they are still made
  by Advanced Photonix, and availability has "severely diminished"
  because of an EU ban on cadmium sulphide. A design that depends on a
  particular cell has to check what can actually be bought.

**Notes:**
- The LED and the photoresistor are separate parts here, joined by a
  dotted line: a light path is not a wire (convention rule 11). A
  packaged opto-isolator with a cell and an LED in one body (the
  Vactrol type; VTL5C3 has the datasheet figures above) replaces both
  boxes with one part.
- The cell goes in *series* with the signal and the load resistor to
  ground. A cell in *shunt* would make more light mean more
  attenuation, the wrong way round for the contract.
- Values: the LED Series Resistor of about 1 kΩ gives roughly 4 mA of
  LED current when the control voltage is 6 V (a 12 V supply's half);
  the Load Resistor sets the balance between full-volume loss (a
  larger resistor is better) and the quiet end (a smaller resistor is
  better).
- The audio path is referenced to ground, not to a virtual ground, and
  has no DC in it. Anything after it that biases to mid-supply must
  AC-couple.
- The Input Coupling Capacitor is not frequency-determining for
  timing, but it sets the low-frequency corner against the load: 1 µF
  into 47 kΩ and a cell is a corner below 4 Hz. Values shown are
  illustrative starting points, to be refined once we're actually
  prototyping.

**Sources:**
- [Vactrol VTL5C3 datasheet (hosted by Tubetown)](https://media.tubetown.net/info/datenblaetter/silicon/vtl5C3.pdf)
  — on-resistance, off-resistance, rise and decay times. Datasheet
  text read directly.
- [Power-Balanced Dynamic Modeling of Vactrols: Application to a VTL5C3/2, DAFx 2023](https://www.dafx.de/paper-archive/2023/DAFx23_paper_50.pdf)
  — response-time ranges, the light dependence of the attack time, and
  the availability history. Abstract and introduction read directly; I
  did not check its models.
