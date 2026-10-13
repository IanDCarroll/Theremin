# Volume Control: Candidate Designs

Five candidate circuits for turning *volume-antenna* capacitance into
the DC control voltage that sets the theremin's loudness (the job
described in [VOLUME-OSCILLATOR.md](../notes/VOLUME-OSCILLATOR.md)),
one paired with each of the five pitch designs in
[OSCILLATOR-DESIGNS.md](OSCILLATOR-DESIGNS.md) (four of them mirror
their pitch twin; Design 4 is the historical RCA channel, which does
not, see below). The stage that
consumes the voltage is in [VCA-DESIGNS.md](VCA-DESIGNS.md); the
volume subsystem and the VCA meet only at that one voltage (the
contract is in the notes), so any volume design can be paired with any
VCA design and any pitch design.

Diagram convention is documented in [DIAGRAMMING.md](DIAGRAMMING.md).
Component values shown are illustrative starting points, to be refined
once we're actually prototyping, and everything below is expected
behaviour, not bench-verified.

Each design lives in its own directory, like the pitch designs: a
`README.md` with the prose and a `diagram.mmd` with just the Mermaid
source.

## How the mirror works

Designs 1 to 3 and 5 are their pitch design's oscillator circuit
**copied node for node** (the same node names, so a plain `diff` of the two
`diagram.mmd` files shows exactly what the volume channel adds) with
its antenna input renamed Volume Antenna, followed by an AC-to-DC
stage. That stage is what changes between designs, and it is the only
thing pitch has no counterpart for: pitch must stay AC all the way to
the speaker, volume has to turn the oscillator's hand-dependent AC
into a slowly varying DC level.

**Design 4 mirrors only half.** In the RCA original the volume channel
was one oscillator and a detuned loop, not a second heterodyne pair,
so [Volume Design 4](volume-4-rca-historical/README.md) copies just
the pitch design's reference oscillator node for node and replaces the
rest. See "The historical design" below.

Each pitch README and its volume README carry a note that they mirror
each other. Where the volume oscillator's component values differ from
the pitch design's, the volume README lists each difference and why:
the volume oscillator has to run at a different frequency from the
pitch oscillator (two similar oscillators close together pull on and
lock to each other), and in Designs 1 to 3 the frequency also has to
be low enough for the converter to count.

## The AC-to-DC stage, shared by Designs 1, 2, 3 and 5

These four use the same stage, an **LM2907-N frequency-to-voltage
converter** (Design 4 uses a diode and capacitor detector instead) (8-pin PDIP, TI datasheet SNAS555D):

- The converter turns its input frequency *f* into a DC voltage, V_OUT
  = V_CC × f × C1 × R1 (datasheet equation 5, gain constant about 1,
  ±0.3% typical linearity). Because the output is proportional to the
  supply, the component values are chosen so that the output at "hand
  away" is about half the supply, which is where the VCAs expect full
  scale, and everything tracks a changing supply together.
- Polarity is natural: a hand near the antenna adds capacitance and
  lowers the oscillator's frequency, so the output falls as the hand
  approaches. Higher voltage means louder, which is the direction the
  control-voltage contract asks for, with no inverting stage.
- The 8-pin input is referenced to ground and compares against ±25 mV
  (typical) with 30 mV of hysteresis, so the signal is AC-coupled
  through a capacitor and re-centred by a resistor. The converter's
  own op amp, on pins 7, 4 and 5, buffers the output (Design 5,
  exactly the datasheet's Figure 15) or provides gain and a floor
  adjustment (Designs 1 to 3).
- Speed and ripple trade against each other through the ripple filter
  capacitor: ripple is (V_CC/2) × (C1/C2) × (1 − V_CC × f × C1 / I2)
  peak-to-peak and the time constant is R1 × C2. The maximum input
  frequency is f_MAX = I2 / (C1 × V_CC) with I2 about 180 µA: 20 kHz
  at 9 V, 15 kHz at 12 V.

**What feeds the converter is what separates the designs.**

- **Design 5 (heterodyne): it counts the beat.** The mixer's
  difference frequency is zero when the two oscillators are tuned
  together and grows as the hand moves, so the converter's output
  starts at 0 V and carries the whole hand movement. Tune the pair so
  the beat is zero with the hand at its nearest (silent) position.
  This is the architecture of the one complete all-hardware theremin
  design I found published (DEAK Software): its volume path is a copy
  of its pitch path (a pair of oscillators, a mixer and an envelope
  detector) followed by a frequency-to-voltage converter, with the
  volume pair at 715 kHz and the pitch pair at 1.06 MHz.
- **Designs 1 to 3 (relaxation): it counts the oscillator.** The
  output is proportional to the oscillator's frequency, so the hand's
  contribution is a few percent of a large constant. The conditioning
  stage has to subtract that constant (the Floor Trimmer) and amplify
  roughly 30×, which amplifies ripple, noise and drift by the same
  factor. The only published attempt at this pattern I found, a UIUC
  student project with 8038 oscillator pairs and an LM2917, abandoned
  its volume circuit over a constant offset of about 1 kHz that could
  not be nulled. These three are expected to be the poorest volume
  controls, as they are the least expressive pitch oscillators.

## 1. [CMOS inverter (CD4069UB), frequency-to-voltage](volume-1-cmos-inverter/README.md)

Mirrors [Design 1](oscillator-1-cmos-inverter/README.md): one gate,
one resistor, one capacitor, retuned to about 10 kHz, feeding the
converter with a floor and gain stage.

## 2. [555 timer, frequency-to-voltage](volume-2-555-timer/README.md)

Mirrors [Design 2](oscillator-2-555-timer/README.md): the same
astable, retuned to about 10.5 kHz by one resistor. The 555's
datasheet gives its timing drift (about 150 ppm/°C and 0.3 %/V), which
is the number that decides how much floor drift to expect.

## 3. [Comparator relaxation oscillator (LM393), frequency-to-voltage](volume-3-comparator-lm393/README.md)

Mirrors [Design 3](oscillator-3-comparator-lm393/README.md), with two
resistors changed to bring it to about 10.6 kHz.

## 4. [RCA detuned-loop volume channel, historical analog](volume-4-rca-historical/README.md)

The volume half of the 1929 RCA theremin: one oscillator (the pitch
design's reference oscillator, copied node for node) and a passive loop
tuned just below it that includes the antenna. A pickup coil, a diode
and a capacitor turn the loop's radio-frequency current into the
control voltage, in the place of the original's heated filament. No
converter and no second oscillator, so it differs in kind from the
other four.

## 5. [Heterodyne, Colpitts BJT oscillators, beat-to-voltage](volume-5-colpitts/README.md)

Mirrors [Design 5](oscillator-5-colpitts-heterodyne/README.md): both
Colpitts oscillators and the mixer copied, with a second low-pass pole
and the converter added.

---

## Where these differ at a glance

| Design | Mirrors | Converter fed by | Active devices | Total parts |
|---|---|---|---|---|
| [1. CMOS inverter](volume-1-cmos-inverter/README.md) | Design 1 | Oscillator output (about 10 kHz) | 1× CD4069UB gate, 1× LM2907-N | 13 |
| [2. 555 timer](volume-2-555-timer/README.md) | Design 2 | Oscillator output (about 10.5 kHz) | 1× NE555, 1× LM2907-N | 15 |
| [3. Comparator (LM393)](volume-3-comparator-lm393/README.md) | Design 3 | Oscillator output (about 10.6 kHz) | 1× LM393, 1× LM2907-N | 17 |
| [4. RCA historical analog](volume-4-rca-historical/README.md) | Design 4's reference oscillator only | Not converted: the loop's current is rectified directly | 1× 2N5484, 1× 1N5711 | 16 |
| [5. Colpitts heterodyne](volume-5-colpitts/README.md) | Design 5 | Beat frequency (about 0 to 2 kHz) | 2× 2N3904, 1× LM2907-N | 30 |

"Total parts" counts every box and hexagon in each design's diagram,
as for the pitch designs; Junction bars (see below) are not parts. The
mirror's part count is the pitch design's plus 10 for Designs 1 to 3
(the converter and its three conditioning parts) and plus 9 for
Design 5 (a second filter pole and the converter). Design 4 is not
counted that way: it is the oscillator's 10 parts plus a loop and a
detector.

## Performance, at a glance

Expected, not yet bench-verified — each README has the reasoning.

| Design | Biggest strength | Biggest limitation |
|---|---|---|
| [1. CMOS inverter](volume-1-cmos-inverter/README.md) | Fewest parts | Hand signal is a few percent of a large constant; least stable floor |
| [2. 555 timer](volume-2-555-timer/README.md) | Timing drift is a datasheet number (150 ppm/°C, 0.3 %/V) | Same pedestal: 0.3 %/V against a few-percent span |
| [3. Comparator (LM393)](volume-3-comparator-lm393/README.md) | Thresholds are a fixed fraction of the supply | Input bias current at megohm timing resistance shifts the floor |
| [4. RCA historical analog](volume-4-rca-historical/README.md) | The original's method; one oscillator; no pedestal, since no current means 0 V | Slope detection: the output may not reach 0 V; depends on oscillator amplitude and supply; unverified that the pickup can give about 7 V |
| [5. Colpitts heterodyne](volume-5-colpitts/README.md) | The same, with off-the-shelf parts | The most parts in the repo (30) |

The pattern mirrors the pitch designs': the single relaxation
oscillators are simpler and less capable, the heterodyne pair (5) is
the one that behaves like a modern theremin, and for volume the gap is
wider because volume needs a true zero, which only a difference
frequency gives. Design 4 is the original's different answer: a
single oscillator and a loop.

## The historical design

[Volume Design 4](volume-4-rca-historical/README.md) is the RCA AR-1264's
volume channel, which is **not** a mirror of its pitch channel. Pitch
used two oscillators and a mixer (heterodyne). Volume used a *single*
420 kHz oscillator and a passive loop (an antenna coil of single-layer,
low-capacitance construction) tuned *below* the oscillator's frequency.
The hand's capacitance detunes the loop further from the oscillator,
which lowers the radio-frequency current in a pickup coil. In the
original that current heated a tube filament, which set the plate
current of the first audio tube; in Design 4 a diode and capacitor
rectify it into the control voltage, and
[VCA Design 5](vca-5-rca-historical/README.md) plays the gain
tube's part. Theremin's patent (US 1,661,058, Fig. 11) shows the same
arrangement. So the original is a slope detector, of the kind the next
section says needs a small-capacitance, high-Q loop, and its AC-to-DC
conversion is thermal.

## Slope detection, and why only one of the five uses it

The textbook way to turn an oscillator's frequency shift into a level
is a **slope detector**: a resonant tank whose amplitude falls as the
frequency moves off its peak. I worked out what it would take on the
*RF* oscillator itself: the amplitude left after a hand shifts the
frequency is about 1 / √(1 + (2 × Q × δ)²), where δ = ΔC / (2 ×
C_total) and Q is the tank's. For a 5 pF hand change and Q = 100:

| Oscillator tank capacitance (C_total) | Amplitude left |
|---|---|
| 3,000 pF (the nF scale that Design 5's 40 µH coil implies at a few hundred kHz) | 0.99 (essentially no change) |
| 200 pF | 0.37 |
| 100 pF | 0.20 |

So slope detection needs a small-capacitance, high-Q tank, quite
unlike the pitch oscillators' values, and still does not reach silence
unless Q × δ is about 5 or more. That is why Designs 1, 2, 3 and 5
count a frequency instead; Design 4 accepts the limits, as the original
did, with a single-layer loop coil built for low capacitance. The real-world version that does work, in
the Seven Transistor Labs build, mixes the volume oscillator with the
local oscillator at an offset of tens of kHz and puts the slope
detector (a 4.7 mH and 0.022 µF tank, by my arithmetic resonant near
16 kHz, and a voltage-doubler Schottky detector) on the difference
frequency, where a few kHz of shift is a large fraction of the tank;
it then uses the detected voltage to bias the pitch mixer. That would
be a candidate for a later sixth design; it merges the volume detector
into the pitch mixer, which breaks the clean separation between the
volume subsystem and the VCA.

## Open issues found while mirroring

None of these has been changed in the pitch designs; each is carried,
unchanged, into the mirror, and each wants a look.

- **Design 1 may not oscillate as drawn.** One inverter with only a
  feedback resistor and a capacitor to ground has too little phase
  shift in the ideal model; usual one-gate RC oscillators use a
  Schmitt-trigger gate, and the UBC build the antenna-sensor README
  cites uses two CD4069UB gates. Not checked on a bench.
- **Designs 2 and 3's illustrative values are not audio.** Design 2's
  values work out to about 690 kHz (1.44 / ((1 kΩ + 2 × 10 kΩ) × 100
  pF)); Design 3's to about 5 MHz, beyond the LM393's typical 1.3 µs
  response time.

## Sources and how well I read them

- LM2907-N / LM2917-N datasheet (TI SNAS555D): text extracted and
  read; Figure 15 rendered and read.
- DEAK Software, "Building a Theremin": the full schematic and the
  envelope-detector, converter and VCA figures were read; the article
  prose came only through a search summary, so the DEAK frequencies
  here are read off the schematic captions.
- Seven Transistor Labs, "Theremin": read in full as a PDF, with the
  oscillator and volume-mixer schematics.
- Pat Tarantino, "Theremin", UIUC Physics 406: the volume-circuit and
  verification sections read in full.
- NE555 and LM393 datasheets (TI): the figures quoted above read from
  the extracted text.
- US 1,661,058 (Theremin, filed 1925): the full patent PDF, drawings
  and text; Fig. 1 and Fig. 11 read as images. The RCA AR-1264
  schematic (theremin.us, Arthur Harrison's 2013 redraw) read as an
  image, and its text; the service notes themselves only through a
  summary. Details in
  [Oscillator Design 4](oscillator-4-rca-historical/README.md) and
  [Volume Design 4](volume-4-rca-historical/README.md).
