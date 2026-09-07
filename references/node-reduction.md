# Node reduction catalogue

Read this before planning any graph. Each entry is a pattern agents reliably produce, why it
happens, and what replaces it. Counts are indicative, not exact.

## Contents

1. Branching and selection
2. Casting and type checks
3. Repetition
4. Data and literals
5. Per-frame work
6. State
7. Structs and collections
8. Maths
9. Things that should not be in a Blueprint at all
10. Anti-patterns in the refactor itself

---

## 1. Branching and selection

**Branch ladder on one value.** Five Branch nodes each comparing the same variable against a
different constant, ~20 nodes with the equality nodes.
Replace with **Switch on Enum** or **Switch on Int** - 1 node. If the value is a string or
name, **Switch on Name/String** exists too. If the outcome is a value rather than a flow,
**Select** collapses the whole thing to 1 node with no exec pins at all.

**Branch that only picks a value.** Branch -> two Set nodes on the same variable, 3-4 nodes.
Replace with **Select** feeding one Set - 2 nodes.

**Duplicated tail.** The same three nodes on both the True and False path, 6 nodes.
Move them past the merge point - 3 nodes. If both paths are identical except one value, the
Branch itself is usually the wrong shape; use Select.

**Nested Branches for a compound condition.** Branch inside Branch inside Branch, 3 nodes plus
routing.
Replace with a single Branch fed by **AND/OR** nodes, or better a pure function named after
the condition (`CanFireWeapon`) - 1 node at the call site and a self-documenting name.

**Sequence node doing unrelated work.** An 8-pin Sequence is not organisation; it is eight
things that should be eight functions or eight events. Sequence is for genuine ordering
requirements inside one concern.

---

## 2. Casting and type checks

**Cast chains for type dispatch.** Cast to A -> failed -> Cast to B -> failed -> Cast to C,
each with its own call, ~15 nodes and a hard dependency on all three classes.
Replace with a **Blueprint Interface** message send - 1 node, no dependency, no cast cost.
This is the single biggest node-count and coupling win available in Blueprints.

**Repeated cast to the same object.** Get Player Character -> Cast To BP_Player appearing in
six places, 12+ nodes and six runtime casts.
Cast once on BeginPlay (or on possession), store in a typed variable, then Get - 1 node per
use afterwards. Guard against the reference going stale if the object can be destroyed.

**Get All Actors Of Class inside a loop or on Tick.** Expensive and usually 5+ nodes of
filtering after it.
Register the actors once, or use a subsystem/manager reference, or gameplay tags.

---

## 3. Repetition

**Copy-pasted subgraph.** The same 12-node block appearing three times, 36 nodes.
Collapse to a **function** - 12 nodes once, 3 call nodes. Rules of thumb:
- Function: single exec in, single exec out, no latent nodes. Preferred - it gets a
  description, shows in the palette, and is callable from other graphs.
- Macro: needed only when there must be multiple exec outputs or a latent node inside.
  Macros do not appear in the debugger the same way; use sparingly.
- Blueprint Function Library: when two or more Blueprints need it. Static, no self reference.
- Pure function: when there are no side effects. Do not mark impure work pure - it re-executes
  per output pin read and produces bugs that look like race conditions.

**Manual repetition instead of a loop.** Six near-identical chains operating on index 0-5,
~40 nodes.
Replace with an array plus **ForEachLoop** - ~8 nodes. If the six things differ only by a
value, that value belongs in the array or a Data Asset.

**Per-element node chain built by the agent because it could not read the array length.**
This happens when the build ran ahead of the read pass. Stop, read the data, use a loop.

---

## 4. Data and literals

**Make Vector / Make Rotator / Make Transform chains for constants.** 4-8 nodes each,
repeated wherever the constant is needed.
Promote to a variable with a default value - 1 Get node. If a designer will ever tune it,
make it instance-editable with a Category, or move it to a **Data Asset**.

**Magic numbers scattered across the graph.** Every literal that appears twice is a variable
waiting to happen. Every literal a non-programmer might want to change is a Data Asset entry
waiting to happen.

**Long Format Text / Append chains for a string.** 10+ nodes.
One **Format Text** node with named arguments - 1 node, and it is localisable.

**Data Table lookups rebuilt as a Switch.** If the mapping has more than a handful of entries
or will change, it belongs in a Data Table or curve, not in nodes.

---

## 5. Per-frame work

**Logic on Event Tick.** Almost always wrong, and it is where node bloat compounds because
every guard has to be re-evaluated per frame.
- Waiting for something: **event dispatcher** or delegate binding.
- Doing something periodically: **Set Timer by Function Name / by Event**.
- Smooth value over time: **Timeline**.
- Reacting to an overlap or hit: the overlap/hit event.
If Tick genuinely is needed, gate it, set a tick interval, and put the whole body in one
function so the event graph stays readable.

**Polling a boolean every frame.** Replace with a setter function that fires a dispatcher when
the value changes.

---

## 6. State

**Boolean pyramid.** Six bools that are mutually exclusive, plus the ~20 nodes of Branches
keeping them consistent, plus the bugs when two are true at once.
Replace with a **state enum** and one Switch. Add a `SetState` function that is the only
writer - then state transitions are greppable and loggable in one place.

---

## 7. Structs and collections

**Set-per-field.** Six Set nodes writing six fields of the same struct variable.
**Make Struct** into one Set, or **Set Members in Struct** - 2 nodes. Set Members also avoids
the copy-back mistakes that Break/Make round-trips cause.

**Break Struct feeding Make Struct unchanged.** Pure noise - 12 nodes doing nothing. Pass the
struct.

**Parallel arrays kept in sync by hand.** Three arrays indexed together, with all the
bookkeeping nodes that implies.
One array of a struct, or a **Map**.

---

## 8. Maths

**Long arithmetic chains.** A 12-node float expression.
A single **Math Expression** node holds the whole formula as text, is readable, and is one
node. For anything heavy or reused, a C++ Blueprint Function Library call is one node and far
faster.

**Reimplemented engine functions.** Lerp, interpolation, clamping, remapping, distance,
normalisation, angle wrapping - all exist. Check the palette before building. `FInterp To`,
`Map Range Clamped`, `Normalize to Range`, `Ease` cover most hand-built chains.

---

## 9. Things that should not be in a Blueprint at all

- Tight loops over thousands of elements - C++.
- Per-frame maths on many actors - C++, or Niagara, or an ISM/instanced approach.
- Bulk asset edits across the content browser - editor Python, run once, not a Blueprint.
- Config that changes per show, per venue, or per client - Data Asset, Data Table, or ini,
  so it can change without recompiling a Blueprint.

Say so and propose the alternative rather than building the Blueprint version anyway.

---

## 10. Anti-patterns in the refactor itself

The cleanup can be worse than the mess:

- **Over-abstraction.** A function called once, wrapping two nodes, adds a hop for no gain.
  Extract at the second or third occurrence, not the first.
- **Function soup.** Twenty three-node functions with generic names is not more readable than
  one clear 25-node graph. Name by intent; if the name needs "And" in it, it is doing two
  things.
- **Pure functions with side effects.** Silent multiple execution. Do not.
- **Collapsed nodes as a hiding place.** Collapse to Node hides complexity without making it
  reusable or documentable. Prefer a real function.
- **Interfaces for a single implementer.** Adds indirection with no decoupling benefit until
  there are two.
- **Rewriting working production logic to hit a node count.** The budget is a design tool for
  new work and a discussion opener for old work, not a licence to refactor something on air
  this week. Propose, then wait.
