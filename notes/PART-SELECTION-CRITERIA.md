# Part Selection Criteria

Working assumptions for choosing components during discrete redesign.
These apply whenever we're picking a specific resistor, capacitor,
transistor, op-amp, etc. to replace a black-box part.

## Target

Solid, standard, widely-available parts — not the cheapest/substandard
tier, but also not over-specified industrial, automotive, or military
grade. The goal is a reliable, well-documented build, not a rugged one.

## Environmental operating assumptions

Build will live in a non-temperature-controlled space in Los Angeles.

- **Temperature range:** ~35°F to 115°F (roughly 1.7°C to 46°C)
- **Humidity:** ordinary indoor humidity, non-condensing

## What this means for part selection

- **Standard commercial-grade parts (0°C–70°C) already cover this range
  with margin.** No need to seek industrial (-40°C–85°C) or automotive
  grade — that would be over-specifying for this context and generally
  harder to find in breadboard-friendly (through-hole) packages.
- **Electrolytic capacitors are the one component worth a second look.**
  Their datasheets specify a "rated life" that derates with sustained
  heat. Not a concern for something that isn't running continuously, but
  avoid the absolute cheapest low-temperature-rated parts if the build is
  expected to sit powered in a hot space for extended periods.
- **Humidity:** non-condensing indoor humidity is the default assumption
  behind uncoated standard parts — no special handling or coating needed.
- **Practical upshot:** default to standard commercial-grade parts from
  common distributors (well-documented datasheets, common footprints).
  No need to hunt for ruggedized or high-temp variants.

## Not in scope

- Value engineering (cost-minimization at scale) — not a goal, this
  isn't headed for mass production. See [README.md](../README.md).
