# The Outside-In Method — agent skills

Two skills that carry a system design method your agent can run against **a system
you already have**, not a curated example.

Full method, worked example and eleven applied walks: **<https://teob.cc/method>**

## Install (Claude Code)

```bash
/plugin marketplace add lambda-house/teob-method
/plugin install outside-in@teob-method
```

Then, in any repository:

```
/outside-in:review     audit what exists
/outside-in:derive     derive what doesn't yet
```

## What they do

**`review`** — takes a running system, a design document or a diagram and:

- recovers what is outside the boundary — clients, tolerances, the organisation
  that pays for it
- types every component into surface / channel / processor / store, decomposing
  where one product plays several roles
- runs the **deletion test**: which constraint's removal would delete this?
- builds the traceability map and reads it both ways — uncited mechanisms,
  unmapped requirements, and mechanisms carrying more than one promise
- labels every figure **measured**, **contractual** or **assumed**, and turns the
  assumed ones into a risk register
- re-derives the arithmetic independently and reports where it disagrees

**`derive`** — runs the ten steps forward from a described problem, refusing to
name any technology until the demand arithmetic binds. Produces the artifacts in
order: client classes and tolerances, data classes, the labelled graph, the
degradation ladder, the operations plan, the trace.

## Two rules they will not bend

**No invented numbers.** A missing figure is reported as a finding, never filled
in from a reference architecture. If arithmetic needs an input that does not
exist, the skill names it, marks it assumed, shows the sensitivity and continues.

**Chesterton's fence in production.** In a design under review the deletion test
is a razor. In a running system an uncited mechanism is reported as *"no recorded
reason — confirm before removing"*, never as an instruction to delete.

## Other agents

The skills are plain markdown with YAML frontmatter — the same files work in
Cursor, OpenCode, Windsurf and Copilot. Copy `skills/<name>/SKILL.md` (and its
`references/`) into wherever that agent reads skills from.

## Licence

MIT.
