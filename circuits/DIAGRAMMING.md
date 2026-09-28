# Diagramming Approach

Decision on how we draw and share circuit designs going forward, since
we now have actual schematics to produce (starting with
[OSCILLATOR-DESIGNS.md](OSCILLATOR-DESIGNS.md)).

## Requirements

- **Human readable/reviewable** — by me, and by colleagues I share this
  with, without requiring them to install specialized EDA software.
- **Agent readable** — plain text, parseable, diffable in git, no binary
  or proprietary format standing between the diagram and its meaning.

## Decision: Mermaid flowcharts, as a component/net graph

We're using [Mermaid](https://mermaid.js.org/) `flowchart` diagrams.
Reasoning:

- **It's just text.** No external tool required to author or read it —
  it's plain text, diffs cleanly in git, and reads as plain text even
  with zero rendering.
- **It renders automatically almost everywhere that matters** — GitHub,
  GitLab, VS Code (with a common extension), Obsidian, and most modern
  markdown viewers render Mermaid natively. Colleagues don't need to
  install anything to see the visual.
- **Trivially agent readable** — it's a small, well-documented, plain-
  text graph grammar. No image parsing, no proprietary schema.

The tradeoff: Mermaid has no built-in schematic symbols (no resistor
zigzags, no transistor glyphs). We're not using it to draw
publication-quality schematics — we're using it to draw the circuit as
a **labeled graph of components and their pin-level wiring**, which is
honest about what we actually need at this stage (get the topology and
part values right, understandable and reviewable) rather than
pretending to be a full EDA tool. If/when we move to actually building
and want a fabrication-ready schematic, that's a good point to bring in
real schematic-capture software (e.g. KiCad) — a separate decision for
later.

## File organization: prose vs. diagram source

Each circuit element — an oscillator design, the antenna sensor, and
whatever comes next as we work through the rest of the theremin's
signal chain — gets its own directory, holding exactly two files:

- **`diagram.mmd`** — the raw Mermaid source, and nothing else. No
  markdown fencing, no prose. This is the machine-readable artifact:
  the thing an agent (or a script, or a Mermaid CLI) reads when it
  needs the circuit itself, with no commentary mixed in to parse
  around.
- **`README.md`** — the sidecar: everything that's prose (context,
  rationale, what's included/excluded and why) plus a link out to
  `diagram.mmd`. This is what a human (or an agent building context
  before making a change) starts with.

We don't duplicate the diagram source inside the README as an embedded
` ```mermaid ` block — that would create two copies of the same
diagram that can silently drift apart. The one tradeoff: unlike an
embedded fenced block, a bare `.mmd` file won't auto-render inline
wherever the README is being read, so the README just links to it and
says to paste it into [mermaid.live](https://mermaid.live) or open it
with a Mermaid-aware viewer/extension.

See [OSCILLATOR-DESIGNS.md](OSCILLATOR-DESIGNS.md) for the directory
layout this produces in practice.

## Convention (v2 — component-centric)

This supersedes an earlier version of this convention that modeled
electrical *nets* as the nodes and passives as labeled edges between
them. That made small circuits read more like an abstract wiring graph
than an actual parts list — harder to answer "what do I need to buy and
where does it go" at a glance. This version puts real, physical
**components** front and center as the nodes, matching how you'd
actually shop for and place parts on a breadboard.

1. **Every physical passive component is a box node** — resistors,
   capacitors, inductors: `["Base Bias Resistor (Upper), ~47 kΩ"]`.
2. **Every IC is a hexagon node** — `{{"Timer IC (NE555)"}}`. ICs are
   the one category that gets a distinct shape from other components,
   since they're multi-pin packages rather than simple two-lead parts.
3. **A wire connecting to a specific IC pin carries an unboxed
   (plain-text) edge label** naming that pin — e.g.
   `IC ---|"Pin 3 (Output)"| OtherComponent`. The label sits on the
   wire, not in its own boxed/shaped node.
4. **Every pin that's part of the design gets its own labeled
   connection** — including a pin that's simply tied straight to a rail
   with no component in between (e.g. an IC's reset pin wired directly
   to the supply). If we're using a pin, it's labeled; nothing is left
   implicit.
5. **Any component with three or more leads gets every lead labeled**
   this way — this covers transistors (base/collector/emitter) exactly
   like it covers ICs, and a tapped inductor (top/tap/bottom) too.
   A plain two-lead passive (an ordinary resistor or non-polarized
   capacitor) doesn't need its leads labeled — there's no ambiguity
   about which end is which.
6. **A polarity-sensitive two-lead part gets its leads labeled anyway**
   — a diode's `"Anode"` / `"Cathode"`, since which way it faces
   changes what the circuit does even though it only has two leads.
7. **Circles are reserved for the circuit's true inputs and outputs** —
   the points where this circuit meets the outside world: the supply
   rails, the final signal output, and (at whatever level of detail a
   given diagram is scoped to) the player. An internal wiring junction
   between components is not a circle; it's just wherever two or more
   edges happen to land on the same component's lead.
8. **A wire between two components carries no label unless it's
   identifying a pin (rules 3–6).** What's on each end already says
   what the connection is — a label repeating that would be noise.
9. **Our own labels never abbreviate.** `R1`, `C4`, `ANT`, `Cb` and
   similar shorthand are gone in favor of full, purpose-descriptive
   names (`"Base Bypass Capacitor"`, not `"Cb"`). The one exception:
   a real part number (`2N3904`, `NE555`, `1N4148`) or a pin name/number
   straight from that part's datasheet stays as-is, so the diagram can
   still be cross-referenced against the actual technical sheet.
   Standard engineering units (`kΩ`, `µH`, `nF`, `V`) aren't treated as
   abbreviations here — they're universal datasheet notation, not our
   own shorthand.

10. **A plain two-lead passive (resistor, capacitor, inductor, diode)
    gets exactly one wire per lead — two wires total, full stop.** Even
    when several things electrically share the net at one end, they
    don't all splice directly onto the passive itself. Route the extra
    connections through whatever's actually sitting at that net and can
    properly absorb multiple wires: a hub-capable device already there
    (an IC or transistor pin, or a tapped inductor's own lead), or, if
    the junction happens to coincide with one of the circuit's true
    inputs/outputs, that I/O circle. A passive with wires arriving from
    more than one direction on more than one of its leads reads as
    ambiguous — it becomes unclear whether the diagram means one part
    or two, or whether a wire landed there by mistake. This was caught
    by exactly that confusion during review of the 555 design (a
    Threshold Resistor had picked up four connections instead of two)
    and the Colpitts/Hartley mixer stage (a diode had picked up five);
    both were fixed by moving the extra connections onto the IC and the
    Pitch Signal Output circle, respectively, rather than left piled on
    the passive.

11. **A dotted/dashed line means non-conductive coupling, not a real
    wire.** A solid line always means "these two things are joined by
    an actual conductor" — a wire, a trace, a lead. Where two things
    interact without any conductor between them (the clearest case
    being the player's hand and the antenna, coupled only through the
    air via a changing electric field — see
    [ANTENNA-CAPACITIVE-SENSOR.md](../notes/ANTENNA-CAPACITIVE-SENSOR.md)),
    we draw that connection dotted instead: `HAND -. "label" .- ANTENNA`.
    This came up because a solid line there would visually claim there's
    a wire a person could solder between a hand and a circuit, which
    isn't true and isn't what's meant.

Where a package has pins we simply aren't using in a given design (e.g.
we use one gate out of six in a hex inverter package, or one comparator
out of two in a dual-comparator package), we don't draw the unused
gates/comparators or their pins — the diagram scopes to what this
specific circuit actually uses, and the corresponding README says so
explicitly rather than leaving it to be inferred.

The antenna itself shows up two ways, depending on what a given diagram
is scoped to. In each of the five oscillator designs, "Antenna" is
drawn as a single input circle — those diagrams are scoped to the
oscillator, and the whole antenna-plus-hand phenomenon is collapsed
into one external input for that purpose. The dedicated
[antenna-sensor](antenna-sensor/README.md) diagram zooms into that same
input at finer detail: there, the **Antenna is a box** (it's the real,
physical, purchasable/buildable part), and the **Hand is the circle**
(it's the true external actor, outside our build, that we don't wire or
control) — connected to the antenna by the dotted field-coupling line
from rule 11, then wired (solid) into the oscillator from there. Both
views are correct at their own level of detail; a reader moving between
them shouldn't be surprised by the shape swap.

Component values shown are illustrative starting points for
understanding the topology, not verified/simulated final values — those
get refined once we're actually prototyping on the breadboard.

Related conventions already in place: [RESEARCH-STANDARDS.md](../notes/RESEARCH-STANDARDS.md) (source quality bar), [PART-SELECTION-CRITERIA.md](../notes/PART-SELECTION-CRITERIA.md) (what grade of part to specify).
