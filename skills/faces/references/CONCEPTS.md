# What is Faces

The Faces Platform is a compiler for the human mind.

Feed it source material — documents, essays, interviews, conversations — and the compiler extracts the minimal set of cognitive primitives that define a persona. These primitives are the machine code of the mind. Like amino acids, this small formal vocabulary is sufficient to construct any human persona. The personas that result are coherent, consistent, complex, and contradictory — just like real people.

The output is a **Face**: a compressed cognitive instruction set stored as embedded semantic vectors. The principal components of a Face are frequency bands — `epsilon`, `delta`, `beta`, and `alpha` — each capturing a different resolution of cognition. `face` is their equal-weight composite. A Face reshapes any LLM into a new next-token predictor — not a model wearing a costume, but one whose probability distributions are fundamentally altered by the persona. This overcomes model collapse: instead of regression to the mean, you get a sharply individuated mind.

Because the primitives are embedded, they are semantic objects. You can measure the distance between two Faces. You can find which Faces are nearest or furthest on any component. This is the mathematics of the mind.

You can also do boolean algebra on Faces. A **composite Face** is defined by a formula over concrete Faces using four operators:

| Operator | Syntax | Meaning |
|---|---|---|
| Union | `a \| b` | All knowledge from both; `a` wins on conflicts |
| Intersection | `a & b` | Only knowledge present in both |
| Difference | `a - b` | Knowledge in `a` that is not in `b` |
| Symmetric diff | `a ^ b` | Knowledge in exactly one, but not both |

Operators can be parenthesized: `(a | b) - c`. Composite Faces are live — if you sync new knowledge into a component Face, the composite reflects it automatically on the next inference request.

## Value Proposition

Programmable and composable plug-and-play personas as nuanced as a real human for fewer tokens than any other approach. More persona, less cost.

For example use cases, see [USE_CASES.md](USE_CASES.md).
