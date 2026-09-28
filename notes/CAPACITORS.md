# Capacitors — Research Notes

Working notes from research into how capacitors work, how they differ
from batteries, and which capacitor types make sense for the discrete
redesign. Written up because it came up while discussing part selection
and is worth having as reference material going forward.

## Is a capacitor the same as a battery?

No — they store energy through fundamentally different physical
mechanisms, even though both can supply electrical energy to a circuit.

- **Capacitor:** stores energy *electrostatically*. Two conductive
  plates face each other with an insulating layer (the dielectric)
  between them. Apply voltage, and positive charge piles up on one
  plate while negative charge piles up on the other — no chemical
  reaction involved, just charge held apart by the field between the
  plates. Because there's no reaction to wait on, capacitors can charge
  and discharge in microseconds to milliseconds.
- **Battery:** stores energy *chemically*. Two electrodes with
  different chemical potentials sit in an electrolyte; discharging
  drives an actual chemical reaction (a redox reaction) that pushes
  electrons out through the circuit and ions through the electrolyte
  internally. Reaction kinetics are slower, so batteries charge/discharge
  over minutes to hours, not milliseconds — but they can hold far more
  energy per unit of size/weight.

The practical difference this creates: a capacitor is good for a fast,
short burst (a camera flash), while a battery is good for a steady
supply over a long time (running a device for hours).

## Are EV battery packs basically big capacitors?

No. A 10s-of-kWh EV pack is a chemical battery — thousands of
individual lithium-ion cells wired in series/parallel, using the same
electrostatic-vs-chemical distinction above, just at large scale.
Common EV cell chemistries:

- **NMC (nickel manganese cobalt oxide):** higher energy density,
  favors range/performance; more expensive (nickel/cobalt aren't cheap).
- **LFP (lithium iron phosphate):** lower energy density but safer
  thermal behavior and much longer cycle life (thousands of cycles);
  cheaper, iron/phosphate are abundant.
- **NCA (nickel cobalt aluminum oxide):** similar territory to NMC,
  notably used by Tesla historically.

Where capacitors *do* show up in EVs: **supercapacitors** are sometimes
used alongside the battery pack, not instead of it. They can't hold much
energy (low energy density) but can absorb or release it very fast
(high power density) — useful for regenerative braking bursts or
smoothing power spikes, complementing the battery's job of holding the
bulk of the energy. A **Ragone plot** (energy density vs. power
density) is the standard chart for visualizing this tradeoff, and it
places supercapacitors between conventional capacitors and batteries.

Rough numbers: commercial supercapacitors land around 5–8 Wh/kg;
lithium-ion battery cells land around 100–180 Wh/kg — roughly a
20-30x gap in how much energy fits in the same weight.

## How does a 9V battery differ from an EV pack, chemistry-wise?

Not in fundamental mechanism — both are electrochemical cells. The
differences are chemistry choice, scale, and current capability:

- **9V battery:** typically alkaline (zinc/manganese-dioxide) for
  disposable ones, or a small lithium or NiMH rechargeable pack for
  rechargeable ones. Low total energy (a few Wh), low max current —
  fine for a smoke detector or a guitar pedal, not for propulsion.
- **EV pack:** thousands of lithium-ion cells (NMC/LFP/NCA as above) in
  series/parallel, tens of kWh total, capable of delivering hundreds of
  amps for acceleration and accepting high current for fast charging.

Same electrochemical principle, wildly different scale and chemistry
tuned for the application.

## What this means for our capacitor choices

For a theremin, the frequency-determining capacitors (in the
oscillators/tank circuits that set pitch) are the ones that matter most
— any drift in their capacitance shows up directly as pitch drift, which
is the whole instrument. For those:

- **C0G/NP0 ceramic** or **polypropylene film** capacitors are the
  standard choices for precision oscillator/timing circuits — both hold
  their rated value stably across temperature and voltage. These are
  well-documented, widely available, unglamorous parts — exactly the
  "common and standard" target from [PART-SELECTION-CRITERIA.md](PART-SELECTION-CRITERIA.md).
- **General-purpose ceramic (X7R)** is fine for non-critical bypass/
  decoupling, not for anything setting pitch.
- **Electrolytic capacitors** are reserved for bulk capacitance (power
  supply filtering) where large values matter more than precision —
  not suitable for timing-critical spots due to temperature/frequency
  instability, and polarized (must be installed the right way round).

## Capacitors in series — what it means at our voltage level

Came up discussing a high-school project that wired disposable-camera
flash capacitors in series to try to pulse a coil hard enough to crush
a can. The relevant takeaway for a breadboard theremin running at
roughly 4.5–16V:

- **Series capacitors divide the applied voltage, they don't add it.**
  If you charge a series string of capacitors from one shared source,
  each capacitor only sees a share of the total voltage (split roughly
  in inverse proportion to its capacitance — the smaller-value
  capacitor in the pair takes the larger share of the voltage). Getting
  a series stack to sum to more voltage than the source requires
  pre-charging each capacitor individually before wiring them together
  — a deliberate high-voltage-pulse technique that has no reason to show
  up in a steady 4.5–16V audio circuit.
- **Why it's still worth knowing at this voltage:** if two capacitors
  ever end up in series in our circuit (for example, wiring two
  electrolytics back-to-back to improvise a non-polarized capacitor, or
  stacking two lower-voltage-rated capacitors for headroom), the total
  capacitance drops (`1/C_total = 1/C1 + 1/C2 + ...`) and, if the two
  values aren't matched, the voltage across them won't split evenly.
  The smaller-value capacitor can end up seeing most of the voltage —
  worth checking so it doesn't quietly exceed its own voltage rating
  even though the pair's combined rating looks fine on paper.
- **Not relevant here:** the voltage-multiplying trick itself (stacking
  many pre-charged capacitors to exceed source voltage) and the total
  energy/pulse-power questions from the can-crusher story — those
  belong to high-voltage pulse circuits, not a low-voltage oscillator
  circuit like ours.

## Research links

- [Capacitors — SparkFun Learn](https://learn.sparkfun.com/tutorials/capacitors/all) — illustrated, beginner-friendly walkthrough of how a capacitor is built and how it charges/discharges.
- [Explainer: How batteries and capacitors differ — Science News Explores](https://www.snexplores.org/article/explainer-batteries-capacitors) — plain-language explainer aimed at a general/beginner audience, doesn't oversimplify the chemistry vs. electrostatics distinction.
- [What's the Difference Between Batteries and Capacitors? — Machine Design](https://www.machinedesign.com/automation-iiot/batteries-power-supplies/article/21831866/whats-the-difference-between-batteries-and-capacitors) — engineering-audience version of the same comparison, more technical detail on charge/discharge behavior.
- [Supercapacitors vs. Batteries — Eaton](https://www.eaton.com/us/en-us/products/electronic-components/topics/supercapacitors-vs-batteries.html) — vendor explainer on the supercapacitor middle ground, includes the energy/power density tradeoff.
- [Energy density vs. power density (Ragone plot) diagram — ResearchGate](https://www.researchgate.net/figure/Energy-density-versus-power-density-of-capacitors-batteries-and-supercapacitors-34_fig5_283243206) — the actual chart referenced above.
- [LFP vs NMC vs Solid-State: EV Battery Types — Electric Car Scheme](https://www.electriccarscheme.com/blog/ev-battery-types-lfp-nmc-solid-state) — current rundown of EV cell chemistries and their tradeoffs.
- [Electric vehicle battery — Wikipedia](https://en.wikipedia.org/wiki/Electric_vehicle_battery) — general background/reference on pack construction and chemistries.
- [Ceramic vs Electrolytic Capacitors: where to use each type — Electronics Notes](https://www.electronics-notes.com/articles/electronic_components/capacitors/ceramic-vs-electrolytic-capacitors.php) — practical guidance on capacitor type selection by application, including timing/oscillator circuits.
