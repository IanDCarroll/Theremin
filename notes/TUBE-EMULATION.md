# Emulating a Tube With Modern Parts

**Status: preliminary, for review.** Nothing here is bench-verified.
It records the idea of building a **custom circuit from modern,
breadboard-friendly parts whose electrical behaviour is as close as
practical to a vacuum tube's**, for use in the repo's historical
designs ([oscillator-4](../circuits/oscillator-4-rca-historical/README.md),
[volume-4](../circuits/volume-4-rca-historical/README.md),
[VCA 5](../circuits/vca-5-rca-historical/README.md)), and what has been
settled so far.

## The idea

Today each historical design swaps a tube for a single transistor in
the same role, and says where the match is poor. A fuller version
would be one reusable *virtual tube* sub-circuit per tube type, a few
parts that present a tube's behaviour at the control electrode, the
output and the supply, so the historical circuits could be drawn with
the original's topology and values closer to as-published. The
purity goal in [VOLUME-OSCILLATOR.md](VOLUME-OSCILLATOR.md) does not
apply to the historical designs: the character a tube circuit adds is
part of what we are studying, so the target is to match the tube, not
to be clean.

## What a tube does, and what a part has to match

| Tube property | Triode | JFET | Bipolar |
|---|---|---|---|
| Conducts at 0 V grid, cut off by negative grid | yes | yes | no (needs about +0.6 V) |
| Input current | almost none until the grid goes positive | almost none; the gate junction conducts when forward-biased | base current at all times |
| Current law | 3/2 power of the control voltage | square law | exponential |
| Output curves | plate resistance low (about 10 kΩ), current depends on plate voltage | pentode-like: flat, plate-voltage-independent | flat, high output resistance |
| Transconductance | about 1 mS | a few mS | tens of mS |
| Miller capacitance (input) | several pF, times the gain | very small | larger |
| Heater, emission and thermal lag (V5 in the RCA) | yes | none | none |
| Supply | 100 V and more | a few volts | a few volts |

On this table the JFET is the closest single part on the first three
rows; the bipolar differs on all of them. The output curves are the
JFET's weakest point for a triode and its best point for a tetrode or
pentode (the RCA's mixer tube). This is why the historical designs use
JFETs wherever a tube's grid controls the current.

## What has been found

- **Square law against 3/2 law.** A JFET's drain current follows a
  square law, a triode's a 3/2 power law. An unbypassed source resistor
  of about 0.83 × the pinch-off voltage ÷ the saturation current is
  published as bringing the JFET's curve close to the 3/2 law near the
  operating point; the Fetzer Valve also gives drain-resistor, bias and
  gain formulas and notes that a gate capacitor of 150 to 250 pF
  restores the Miller-effect bandwidth. They were derived for 2N5457 and
  J201 parts at 9 to 18 V.
- **A closer solid-state triode exists.** The static induction
  transistor is described as a normally-on device with unsaturating
  (triode-like) output curves, the nearest solid-state relation of the
  triode. Availability and cost for breadboard use were not checked.
- **A starved triode keeps its gain.** In a quick hand model (generic
  triode, mu 9; parameters assumed, not an RCA tube's), the small-signal
  gain of a triode held at less than its natural plate current changes
  very little (gain varies as roughly current^0.2), because its plate
  resistance rises as its transconductance falls. A JFET's gain in the
  same situation follows the current closely, and a bipolar's follows it
  exactly. So the RCA's volume control probably works mainly through
  the plate *voltage* collapsing, which limits the swing and cuts the
  stage off, and no single part reproduces that. VCA 5 was drawn as
  JFET stages that approximate it and models the result.
- **A heater-controlled diode (V5) has no transistor analogue.** What
  it contributes is a current that a control sets from zero. A
  depletion JFET cannot cut off from a 0 V control on a single supply,
  so VCA 5 uses a bipolar emitter follower there.

## Open questions

1. Is a multi-part virtual triode worth drawing, for instance a JFET
   with a source resistor, a gate capacitor and a drain-to-gate
   feedback resistor to lower its output resistance toward a triode's?
   That feedback was not researched, and it would change the plate
   characteristic the oscillators rely on.
2. How good is a JFET's match to the *grid-current* behaviour that sets
   the oscillators' amplitude (the grid-leak bias)? The RCA's 5 kΩ
   grid return does not transfer to a JFET gate; the oscillator design
   uses 100 kΩ with the same time constant and has not been tested.
3. Does a static induction transistor, or a JFET operated with a
   specific drain resistor, give a triode-like output at 12 V? A
   triode's low plate resistance mostly goes unmatched in the designs
   as drawn.
4. A two-JFET series (cascode) is the stand-in for the RCA's two-grid
   mixer tube. Is there a better one, such as a dual-gate part that can
   still be bought?
5. A tube's gain and distortion depend on its 100 V and higher supply.
   How much of the character survives at 12 V, and is a boost converter
   for the historical designs allowed, given the single-supply
   convention?

## Research links

- [A closer look at the Fetzer Valve — Runoff Groove](https://runoffgroove.com/fetzervalve.html) — JFET stage formulas for imitating a triode stage. *Read directly.*
- [JFETs: The New Frontier, Part 1 — audioXpress](https://www.audioxpress.com/article/JFETs-The-New-Frontier-Part-1.html) — JFET and triode analogies, "triode" and "pentode" regions. *Search-result summary only.*
- [Triode Emulation (100% Sand) — DIYAudio](https://diyaudio.com/community/threads/triode-emulation-100-sand.123771) — a forum thread on emulating triodes with solid-state parts; source of the 3/2 against square law framing. *Search-result summary only; the page itself would not load.*
- [Introduction to Static Induction Transistors — First Watt](https://www.firstwatt.com/product/sit-intro/) — the SIT as a solid-state triode, with unsaturating curves. *Search-result summary only.*
- [Bipolar transistor as cathode follower — Music Electronics Forum](https://music-electronics-forum.com/forum/amplification/guitar-amps/theory-design/10356-bipolar-transistor-as-cathode-follower) — a bipolar base draws current where a grid does not. *Forum post; search-result summary only.*
- [The RCA Theremin — theremin.us](https://theremin.us/RCA/rca_theremin.html) — the original circuit whose tubes are being replaced.
