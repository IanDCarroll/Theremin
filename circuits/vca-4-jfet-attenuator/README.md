# VCA Design 4: JFET Attenuator

Part of [VCA-DESIGNS.md](../VCA-DESIGNS.md)'s candidate set. Diagram
convention is documented in [DIAGRAMMING.md](../DIAGRAMMING.md).
Schematic source: [diagram.mmd](diagram.mmd) — paste into
[mermaid.live](https://mermaid.live) or open with a Mermaid-aware
viewer/editor extension to render it.

A JFET in its ohmic region is a voltage-controlled resistor. Placed as
the lower arm of a divider under a series resistor, it attenuates the
audio by an amount set by its gate voltage: most attenuation with the
gate at the source (the channel at its lowest resistance), none once
the gate is pulled past the pinch-off voltage (the channel open). A
discrete design, from the same family of parts as the oscillators. Its
interface is the one in [VCA-DESIGNS.md](../VCA-DESIGNS.md): pitch
audio in, control voltage in (0 V silent, about half the supply full),
scaled audio out.

**How the diagram works.**
- A Virtual Ground IC (TLE2426) makes half the supply, and everything
  in the audio path sits on it, so the audio can swing both ways
  around it on a single supply. The JFET's source is on it.
- Op amp A (half of the TL072) is an inverting summing amplifier. One
  input resistor takes the control voltage, another (the Offset
  Resistor) takes the supply, and the non-inverting input is on the
  virtual ground. With the two input resistors equal, its output is
  the virtual ground minus (feedback resistor ÷ input resistor) × the
  control voltage: at 0 V control the gate sits on the source (most
  attenuation), and at half the supply the gate is about 2.3 V below
  it. That inversion is what makes more control voltage mean more
  signal.
- Op amp B (the other half) is a unity-gain buffer on the attenuator's
  output, so the next stage does not load the divider.
- The two Linearizing Resistors feed half of the signal voltage back
  to the gate. This is the usual way to flatten the JFET's resistance
  against signal level; the Gate Drive Resistor keeps the op amp's low
  output impedance from shorting that feedback.

**Performance — pros and cons (expected, not yet bench-verified):**
- **Pro:** Cheap, discrete and well precedented: DEAK Software's
  all-hardware theremin uses a pair of 2N5484 JFETs and an LM358 as
  its VCA, with the same kind of virtual ground and a resistor from
  the drain node to the gates.
- **Pro:** Fast. A JFET responds to its gate voltage in microseconds,
  so articulation is limited by the control voltage's own filter, not
  by this stage.
- **Pro:** The gate itself draws almost no current, so the control
  source only has to supply the 100 kΩ input resistor's few tens of
  microamps.
- **Con:** Distortion is the weak point. Linear Systems' application
  note says that for small signals (under about 500 mV peak-to-peak
  across the drain and source) the distortion is "reasonably" low,
  typically under 3%, with the second harmonic dominant; it offers a
  balanced pair of matched FETs to cancel the even-order part. I did
  not find measured distortion for the half-signal feedback used here,
  so its benefit is unverified. This is the least pure of the four.
- **Con:** The input has to be small: at the point where the JFET's
  resistance equals the series resistor, half of the input appears
  across the JFET. Keep the input to about 1 V peak-to-peak or less.
- **Con:** Not true silence. The floor is the series resistor against
  the JFET's on-resistance: with 47 kΩ and a few hundred ohms, roughly
  −44 dB, and I did not check the 2N5484's on-resistance. A lower
  on-resistance part (the J111's is at most 30 Ω in the onsemi
  datasheet) deepens it.
- **Con:** Pinch-off varies enormously from unit to unit. The InterFET
  datasheet gives the 2N5484's gate-source cutoff as −0.3 to −2 V, the
  onsemi 2N5457 as −0.5 to −6 V, and the J111 as −3 to −10 V. With one
  fixed gate-drive gain, some units would cut off at a fraction of the
  control range and others would not cut off at all; the Gate Drive
  Feedback Resistor wants to be a trimmer per build.
- **Con:** Most parts of the four (13), including two ICs.
- **Con:** The 2N5484 is through-hole in TO-92 (InterFET still
  publishes its datasheet) but one source I saw calls it out of
  production; its lifecycle is unconfirmed. The 2N5457 and J111 are
  the substitutes, with the gate-drive range that goes with them.

**Notes:**
- Control law: the JFET's resistance rises steeply as the gate
  approaches pinch-off, so gain against control voltage is strongly
  nonlinear and part-dependent. The contract proposes that the VCA
  owns the gain law; this one's is whatever the JFET gives.
- Op amp A's inputs and output stay well away from the rails (the gate
  sits between about 3.7 V and 6 V on a 12 V supply), so a TL072 is
  fine here; the TI datasheet gives 4.5 V to 40 V single-supply
  operation and 0.003% typical distortion.
- The TL072 package has two amplifiers and this design uses both, as
  noted on the IC. Pin 4 is the negative supply, which here is Ground.
- Drain DC Return Resistor: gives the drain a DC path to the virtual
  ground, since the series capacitor and the buffer's input would
  otherwise leave it floating when the JFET is cut off.
- The Input and Output Coupling Capacitors are not
  frequency-determining for timing. Values shown are illustrative
  starting points, to be refined once we're actually prototyping.

**Sources:**
- [A Guide to Using FETs for Voltage Controlled Circuits, Linear Systems (Bob Cordell)](https://linearsystems.com/applicationnotes/jfets-for-vc-circuits)
  — the JFET attenuator, the ohmic-region resistance formula,
  distortion figures and the balanced alternative. Text read directly
  (Figures 1 to 7); the feedback linearization used here is described
  in later figures I did not have.
- [2N5484 / 2N5485 / 2N5486 datasheet, InterFET](https://www.aeri.com/pn/InterFET/2n5484)
  and the onsemi datasheets for the 2N5457 and J111 — cutoff-voltage
  ranges and, for the J111, on-resistance. Datasheet text read
  directly.
- [Building a Theremin, DEAK Software](https://deaksoftware.com.au/articles/building_a_theremin/)
  — its VCA schematic (Figure 16) shows the JFET pair, virtual ground
  and drain-to-gate resistor. Schematic read directly; the article
  text only through a search summary.
- [TL072 datasheet, Texas Instruments](https://www.ti.com/lit/ds/symlink/tl072.pdf)
  and
  [TLE2426 datasheet, Texas Instruments](https://www.ti.com/lit/ds/symlink/tle2426.pdf)
  — supply range and distortion; rail-splitter pins, input range and
  output accuracy. Datasheet text read directly; lifecycle status of
  neither part was checked.
