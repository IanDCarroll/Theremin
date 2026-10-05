# The Antenna: A Capacitor That Isn't a Component

Research notes on the *sensing* half of the pitch and volume circuits —
how the antenna and the player's body form a capacitor at all. This is
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

## What the tank value means for the player

The tank is the inductor and capacitor the antenna's capacitance gets
added to. Its values never show up as a number to a player; they show up
as how the instrument *feels*. A first-order approximation (standard, from
f = 1/(2π√(LC)); a reviewer can re-derive it) for a small hand-dependent
change ΔC on top of a total tank capacitance C (tank capacitor plus the
antenna's baseline) is:

  Δf ≈ −f · ΔC / (2C)

So the same hand movement produces a bigger frequency change when the
total capacitance C is *smaller* or the oscillator frequency f is
*higher*. Illustration only (not measured): ΔC = 2 pF on C = 100 pF at
300 kHz moves the frequency about 3 kHz; the same ΔC on C = 400 pF
moves it about 0.4 kHz at the same f. In a heterodyne pair that
difference is what the player hears as pitch range.

What that means in the hand:

| Tank change | What the player experiences |
|---|---|
| Smaller total capacitance (smaller tank capacitor, or an antenna with less baseline) | More sensitive: the pitch range is spread across more hand travel *per pF*, so notes sit in a smaller space and the instrument feels twitchy. Longer reach from the antenna, since a smaller ΔC at a distance still registers. Also less stable: stray capacitance changes from a person walking in, or humidity, shift the null more |
| Larger total capacitance | Calmer and steadier, with more room between notes. Shorter reach, and the top of the range may not be reachable at all |
| Higher oscillator frequency | The same fractional shift becomes more Hz, so more range for the same hand movement; layout and stray capacitance get harder to manage |
| Larger L with smaller C at the same f | Same frequency, but higher sensitivity and higher susceptibility to stray capacitance |
| Antenna baseline outside the trimmer's range | The player cannot null the instrument at all: it has a background tone with the hand far away |

Two things the tank does *not* fix:

- **Note spacing versus distance.** The hand-dependent capacitance
  change rises steeply as the hand nears the antenna, so notes crowd
  together close in and spread out far away. That curve comes mainly
  from the antenna's geometry and field, not from the tank (my
  reasoning, not researched; to be checked on a bench).
- **Playing skill.** Higher sensitivity buys expressiveness and reach
  but demands finer hand control; lower sensitivity forgives errors and
  limits nuance. There is no objectively right setting, only a choice
  about who the instrument is for.

For volume (see [VOLUME-OSCILLATOR.md](VOLUME-OSCILLATOR.md)), the same
logic sets how much hand travel takes the level from full to silence: too
sensitive and the dynamic range is squeezed into a small, fiddly space;
too insensitive and silence or full volume cannot be reached.

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

## One antenna block, used twice (pitch and volume)

The antenna is its own block in the circuit diagrams
([circuits/antenna-sensor/](../circuits/antenna-sensor/README.md)), and
the same block serves both channels: a conductor whose capacitance to its
surroundings shifts with the hand. What differs between pitch and volume
is only what happens downstream (see
[PITCH-OSCILLATOR.md](PITCH-OSCILLATOR.md) and
[VOLUME-OSCILLATOR.md](VOLUME-OSCILLATOR.md)).

**The boundary.** The antenna block's output is a capacitance to ground
at a single wire into the oscillator's sensing node: a baseline value
(wiring, shape, room) plus the hand-dependent change. The oscillator
does not know what kind of conductor is attached, and the antenna knows
nothing about the oscillator. The baseline-nulling trimmer described
above sits on the oscillator side of the wire, in the tank, not in the
antenna.

**A natural test point.** Replace the antenna with a fixed or variable
(trimmer) capacitor of a known value to test an oscillator and
everything after it with no hand or room involved. That is repeatable
in a way a real antenna is not, since the real one drifts with the room.

**Geometry.** Rod, plate and loop are all documented antenna shapes
(see the links in
[circuits/antenna-sensor/](../circuits/antenna-sensor/README.md)), and the
classic arrangement is a vertical rod for pitch and a horizontal loop for
volume. Geometry is a mechanical and playing-feel choice, with one
electrical consequence worth noting: it changes the baseline
capacitance and the size of the hand-dependent change, so the
oscillator's tank has to be sized for the antenna actually used. The
pitch and volume antennas also couple to each other if placed close
together, which adds to the channel-pulling problem noted in
[VOLUME-OSCILLATOR.md](VOLUME-OSCILLATOR.md). Neither point is researched
beyond the sources already linked; the baseline and delta figures for each
shape would need measuring.

## Research links

- [Introduction to Capacitive Touch Sensing — All About Circuits](https://www.allaboutcircuits.com/technical-articles/introduction-to-capacitive-touch-sensing/) — illustrated, beginner-accessible, and precise: explicitly separates the dielectric effect and the conductive/virtual-ground effect rather than collapsing them into a loose "finger is a plate" analogy. Primary source for this document.
- [Theremin — Wikipedia](https://en.wikipedia.org/wiki/Theremin) — background on body capacitance as the sensing principle and the antenna/LC oscillator relationship.
- [Capacitance, Heterodyning and The Strange Music of the Theremin — Mini-Circuits Blog](https://blog.minicircuits.com/capacitance-heterodyning-and-the-strange-music-of-the-theremin/) — same source used in [PITCH-OSCILLATOR.md](PITCH-OSCILLATOR.md); also touches the antenna/hand capacitance relationship.
- [Physics of the Theremin (Skeldon et al., American Journal of Physics, 1998, PDF)](https://isidore.co/misc/Physics%20papers%20and%20books/Zotero/storage/3837CDJ6/Skeldon%20et%20al.%20-%201998%20-%20Physics%20of%20the%20Theremin.pdf) — engineering/physics-level depth for cross-checking the beginner-level explanation above.
- [Theremin — Seven Transistor Labs](https://www.seventransistorlabs.com/Theremin/) — practical build notes that include trimmer-capacitor tuning and antenna-to-ground capacitance adjustment in a real oscillator design.
- [Tuning a Theremin — Sweetwater](https://www.sweetwater.com/sweetcare/articles/tuning-theremin/) — practical, player-facing description of the zero-beat tuning process and why re-tuning is a normal part of using the instrument.
