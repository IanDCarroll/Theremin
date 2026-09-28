# Theremin Build — Self-Directed Learning Project

A hands-on learning project in physical electronics prototyping, using a
theremin as the driving example. Not client work — this is professional
development time spent building intuition for hardware that software
engineers increasingly need to touch: sensors, signal paths, and the
boundary between analog and digital.

## Why a theremin

A theremin is a good teaching vehicle because it's simple in concept
(two antennas sense hand proximity via capacitance, which shifts oscillator
frequencies for pitch and volume) but touches a wide range of fundamentals:
oscillators, capacitive sensing, mixing, and audio output. It also has an
obvious, satisfying "it works" moment — you can hear the result changing
as you move your hand.

## Goals

1. **Physical fundamentals first.** Understand breadboards, basic
   components (resistors, capacitors, transistors, op-amps), how circuits
   are actually laid out and debugged by hand, and how to read a schematic
   well enough to build one.
2. **Then bridge to software.** Once the physical/analog side is
   understood, bring in a microcontroller (Arduino or another open-source
   platform) to read sensors, control behavior, and mix analog circuit
   design with code.
3. **End state:** a working (or at least pitch-controllable) breadboard
   theremin prototype, and a working knowledge of the hardware prototyping
   loop — breadboard, measure, debug, iterate.

## Approach

- Start with breadboard basics and simple circuits unrelated to the
  theremin (LEDs, simple oscillators, potentiometers) to build fluency
  with the physical medium.
- Progress toward the theremin's core building blocks: a heterodyning
  oscillator pair, capacitive antenna sensing, and audio mixing/output.
- Layer in Arduino once there's a physical circuit worth instrumenting —
  e.g. reading a sensor value, controlling an LED/speaker, or eventually
  replacing part of the analog signal chain with digital logic.
- Keep notes on what was built, what didn't work, and why — the point of
  this project is the learning trail, not just a finished object.

## Structure (evolving)

```
theremin/
├── README.md                                   # this file
├── notes/                                       # research, terminology, working assumptions
│   ├── PART-SELECTION-CRITERIA.md
│   ├── RESEARCH-STANDARDS.md
│   ├── CAPACITORS.md
│   ├── PITCH-OSCILLATOR.md
│   └── ANTENNA-CAPACITIVE-SENSOR.md
├── circuits/                                    # schematics, one directory per candidate design
│   ├── DIAGRAMMING.md                           # diagram tooling/convention decision
│   ├── OSCILLATOR-DESIGNS.md                    # index + comparison, ordered least → most complex
│   ├── oscillator-1-cmos-inverter/              # 3 parts — simplest
│   │   ├── README.md                            # prose: context, rationale, notes
│   │   └── diagram.mmd                          # Mermaid schematic source, nothing else
│   ├── oscillator-2-555-timer/                  # 5 parts
│   │   ├── README.md
│   │   └── diagram.mmd
│   ├── oscillator-3-comparator-lm393/           # 7 parts
│   │   ├── README.md
│   │   └── diagram.mmd
│   ├── oscillator-4-hartley-heterodyne-historical/  # 17 parts
│   │   ├── README.md
│   │   └── diagram.mmd
│   └── oscillator-5-colpitts-heterodyne/        # 21 parts — most complex
│       ├── README.md
│       └── diagram.mmd
│   └── antenna-sensor/                          # the sensor half of the pitch circuit
│       ├── README.md
│       └── diagram.mmd
└── arduino/                                     # sketches, once we get to the software bridge
```

## Terminology note

Working prototype exists (built from a kit), but some of its components
are opaque — e.g. a flashed microcontroller standing in for what should
be a transparent, documented circuit. The physical-electronics analog to
software "refactoring" is **discrete redesign**: replacing a black-box
part with common, well-documented components (resistors, transistors,
op-amps, etc.) that reproduce the same external behavior. Getting there
requires **reverse engineering** first — understanding what the black-box
part actually does before it can be reproduced in the open.

This is distinct from **value engineering**, which is the formal
industry term for redesigning toward minimal cost at scale — not a goal
here, since this isn't headed for mass production.

## Part selection

Working assumptions for choosing specific components during discrete
redesign (target temperature/humidity range, what grade of part that
implies) are documented separately in
[PART-SELECTION-CRITERIA.md](notes/PART-SELECTION-CRITERIA.md).

Component-specific research (starting with capacitors — how they work,
how they differ from batteries, which types to prefer for our
oscillator/timing circuits) lives in [CAPACITORS.md](notes/CAPACITORS.md).

Our working bar for what counts as a good reference source when we do
this kind of research is documented in
[RESEARCH-STANDARDS.md](notes/RESEARCH-STANDARDS.md).

How antenna capacitance becomes a variable-frequency electrical signal
— heterodyne (dual-oscillator) vs. direct-RC (555-style) design, where
an op-amp fits, and Leon Theremin's original vacuum-tube approach — is
in [PITCH-OSCILLATOR.md](notes/PITCH-OSCILLATOR.md).

How the antenna and the player's body form that capacitance in the
first place — not a component we buy, but a sensor made of geometry,
air, and the player — is in
[ANTENNA-CAPACITIVE-SENSOR.md](notes/ANTENNA-CAPACITIVE-SENSOR.md).

Five concrete candidate circuits for the pitch oscillator (CMOS
inverter, 555, comparator IC, historical-analog heterodyne, heterodyne),
ordered least to most complex and each diagrammed and compared, are in
[OSCILLATOR-DESIGNS.md](circuits/OSCILLATOR-DESIGNS.md). How we diagram circuits
at all — the tooling decision and the diagram convention — is in
[DIAGRAMMING.md](circuits/DIAGRAMMING.md).

The other piece of the pitch circuit — the antenna itself, diagrammed
separately so it can be reasoned about and interconnected with any of
the five oscillator designs above — is in
[circuits/antenna-sensor/](circuits/antenna-sensor/README.md). True to
the physics in ANTENNA-CAPACITIVE-SENSOR.md, it turned out to be exactly
as simple as expected: one wire, no dedicated sensing circuit.

## Status

Kit-built prototype exists and works. Planned steps:

1. Breadboard fundamentals — what's on one, how components go into it,
   building a first trivial circuit unrelated to the theremin.
2. Reverse engineer the kit's opaque components (flashed MCU and any
   other black boxes) to understand what they actually do.
3. Discrete redesign — rebuild those components using common,
   documented parts, preserving the theremin's behavior.
4. Bridge to software — bring in Arduino (or similar) once there's a
   physical circuit worth instrumenting.
