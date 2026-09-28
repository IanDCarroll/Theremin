# The Antenna: A Capacitor That Isn't a Component

Research notes on the *sensing* half of the pitch circuit — how the
antenna and the player's body form a capacitor at all. This is
deliberately split out from [CAPACITORS.md](CAPACITORS.md), which is
about selecting physical capacitor *components* to buy and place in the
circuit. The antenna's "capacitor" isn't a part we purchase — it's a
relationship created by geometry, the surrounding air, and the player's
body, and it behaves differently enough from a manufactured capacitor
to deserve its own writeup. [PITCH-OSCILLATOR.md](PITCH-OSCILLATOR.md)
covers what happens *after* this capacitance reaches the oscillator;
this document stays entirely on the sensing side.

## What kind of "capacitor" is this, exactly?

Capacitive sensors generally come in two configurations:

- **Mutual capacitance:** two distinct electrodes, one emitting and one
  receiving — used in touchscreens, where you need to resolve many
  independent touch points.
- **Self capacitance:** a single electrode, whose capacitance *to its
  surroundings/ground* is what's measured — used for simple buttons,
  sliders, and this is the theremin's case. The antenna is the one
  electrode; there's no second physical plate anywhere in the circuit.
  The "other plate" is effectively the environment (ground) and,
  incidentally, whatever conductive body — a hand — enters that field.

## Why a hand changes the antenna's capacitance (two separate effects)

It's tempting to picture the hand as simply "the other plate" of a
parallel-plate capacitor, but the actual mechanism (per the physics
of capacitive touch/proximity sensing generally) is two distinct
effects layered together:

1. **Dielectric effect.** An antenna's electric field doesn't stop
   cleanly at its surface — it extends out into the surrounding air (the
   "fringing field"). Air has a dielectric constant of about 1; human
   flesh, being mostly water, has a dielectric constant of about 80.
   When a hand enters that fringing field, it's swapping in a much more
   effective dielectric material in place of some of that air, which
   increases capacitance — without the hand needing to be "between two
   plates" in any literal sense.
2. **Conductive effect.** Human skin is also conductive, and the body as
   a whole has real, if large, self-capacitance to its surroundings —
   it acts as a "virtual ground" with meaningful charge-absorbing
   capacity. So the hand also behaves as an additional conductive
   surface, forming a second capacitor (antenna-to-hand) that adds
   in parallel with the antenna's existing capacitance-to-ground.

Both effects scale with proximity, not just contact — which is exactly
why this is a proximity sensor rather than a touch switch, and why
theremin pitch responds continuously to hand distance rather than
behaving like an on/off button.

## The baseline capacitance, and why theremins need a tuning trimmer

Even with no hand anywhere nearby, the antenna already has some
capacitance to its surroundings — its own wiring, its shape, and the
room around it (walls, other objects, other people). This baseline
shifts the antenna oscillator's resting frequency away from wherever the
fixed reference oscillator sits, which would make the instrument produce
an unwanted background tone (or sit outside the audible beat range
entirely) with no hand present.

Real theremin circuits compensate for this with a **trimmer
capacitor** in the antenna oscillator's tank circuit, adjusted during
setup to bring the two oscillators to "zero beat" (silence) at some
chosen reference hand position — commonly with the hand held far from
the antenna. Everything the player then does with hand position is a
*deviation* from that deliberately-established baseline, not an absolute
capacitance reading. This is also the underlying reason theremins are
famously sensitive to the room they're in — anything that shifts the
antenna's baseline capacitance (another person entering the room,
furniture placement, even humidity, since water vapor content affects
the local dielectric) shifts where that "silent point" sits, and the
instrument needs re-nulling.

## Scale: baseline body capacitance vs. the tiny delta being sensed

Worth keeping these two numbers distinct, since they're easy to
conflate:

- A human body's own self-capacitance to its surroundings/ground is
  commonly cited around **100–200 pF** — a real, fairly large number,
  but it's roughly constant for a given person in a given room.
- The *change* in antenna capacitance the player actually creates by
  moving a hand nearer or farther is much smaller — this is the delta
  the oscillator (and, per [PITCH-OSCILLATOR.md](PITCH-OSCILLATOR.md),
  the heterodyning stage) has to be sensitive enough to pick up. It's
  this tiny delta — not the body's absolute 100–200 pF figure — that a
  well-designed theremin circuit is actually built to resolve.

## What this means for our build

Two practical takeaways, distinct from anything in
[CAPACITORS.md](CAPACITORS.md) or
[PART-SELECTION-CRITERIA.md](PART-SELECTION-CRITERIA.md):

- **There's no "part" to source for this half of the system.** The
  antenna is just a piece of conductive rod or wire, and the "sensor" is
  really the interaction of that antenna, the surrounding air, and the
  player — there's nothing here to discrete-redesign in the sense of
  swapping a black-box component for a documented one. The thing worth
  getting right is the **tuning/trimmer stage** that establishes and
  compensates the baseline, since that's the actual circuit element
  standing between "raw antenna capacitance" and "usable oscillator
  input."
- **Environmental stability matters here independently of component
  choice.** Even with perfectly stable, low-drift capacitors in the
  oscillator (per [CAPACITORS.md](CAPACITORS.md)'s C0G/NP0 or
  polypropylene recommendation), the antenna's baseline can still drift
  with room humidity, nearby objects, or how well-grounded the player
  is. That's a real-world quirk to expect and plan a re-tuning step
  around, not a sign that the electronics themselves are wrong.

The circuit-side diagram of this element (the antenna as a single wire
into the oscillator's sensing node, plus what research turned up on
optional ESD protection and antenna geometry) is in
[circuits/antenna-sensor/](../circuits/antenna-sensor/README.md).

## Research links

- [Introduction to Capacitive Touch Sensing — All About Circuits](https://www.allaboutcircuits.com/technical-articles/introduction-to-capacitive-touch-sensing/) — illustrated, beginner-accessible, and precise: explicitly separates the dielectric effect and the conductive/virtual-ground effect rather than collapsing them into a loose "finger is a plate" analogy. Primary source for this document.
- [Theremin — Wikipedia](https://en.wikipedia.org/wiki/Theremin) — background on body capacitance as the sensing principle and the antenna/LC oscillator relationship.
- [Capacitance, Heterodyning and The Strange Music of the Theremin — Mini-Circuits Blog](https://blog.minicircuits.com/capacitance-heterodyning-and-the-strange-music-of-the-theremin/) — same source used in [PITCH-OSCILLATOR.md](PITCH-OSCILLATOR.md); also touches the antenna/hand capacitance relationship.
- [Physics of the Theremin (Skeldon et al., American Journal of Physics, 1998, PDF)](https://isidore.co/misc/Physics%20papers%20and%20books/Zotero/storage/3837CDJ6/Skeldon%20et%20al.%20-%201998%20-%20Physics%20of%20the%20Theremin.pdf) — engineering/physics-level depth for cross-checking the beginner-level explanation above.
- [Theremin — Seven Transistor Labs](https://www.seventransistorlabs.com/Theremin/) — practical build notes that include trimmer-capacitor tuning and antenna-to-ground capacitance adjustment in a real oscillator design.
- [Tuning a Theremin — Sweetwater](https://www.sweetwater.com/sweetcare/articles/tuning-theremin/) — practical, player-facing description of the zero-beat tuning process and why re-tuning is a normal part of using the instrument.
