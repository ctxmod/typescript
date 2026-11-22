# Interface Alternatives

Procedure for exploring alternative interfaces in parallel — the main agent frames the problem space, launches designers under distinct constraints, and compares the candidates on depth, locality, and seam placement.

A first interface is unlikely to be the best one. When a deepening candidate deserves exploration, the main agent designs it several ways at once: frame the problem space, launch parallel designers under distinct constraints, then compare the candidates and recommend. The vocabulary is [Deep Module Design](deep_module_design.md); the dependency categories are [Deepening Shallow Modules](deepening_shallow_modules.md).

## 1. Frame the problem space

Before launching designers, write the framing for the chosen candidate:

- the constraints any new interface must satisfy;
- the dependencies it relies on, each labeled with its category;
- a rough illustrative sketch of the current shape — a way to make the constraints concrete, not a proposal.

Record the framing in the work item so the requester and every designer read the same problem, then proceed immediately to step 2: the requester reviews the framing while the designers work in parallel.

Done when the framing names every hard constraint and presupposes no interface.

## 2. Launch parallel designers

Launch at least three agents in parallel. Each produces a radically different interface for the deepened module, so each brief carries a distinct design constraint:

- **Minimize the interface**: one to three entry points at most, maximum leverage per entry point.
- **Maximize flexibility**: many use cases and extension paths supported.
- **Optimize the common caller**: the default case becomes trivial.
- **Ports and adapters** (when cross-seam dependencies exist): design around the seam.

Each brief carries the deep-module vocabulary and the project's domain language from the project's own documentation and context (in ctxmod-managed repositories, its `context/` tree), plus the concrete material: file paths, coupling details, dependency categories, and what sits behind the seam. The brief is independent of the step-1 framing — the designers work from the brief, the requester from the framing. Each designer returns:

1. the interface — types, methods, parameters, plus invariants, ordering constraints, and error modes;
2. a usage example showing how a caller uses it;
3. what the implementation hides behind the seam;
4. the dependency strategy and its adapters;
5. the trade-offs — where leverage is high, where it is thin.

Done when every designer has returned all five parts against its own constraint.

## 3. Compare and recommend

Present the candidates one at a time so each is absorbed on its own, then compare them in prose on the three axes that matter: **depth** (leverage at the interface), **locality** (where change concentrates), and **seam placement**. Then take a position: name the strongest design and why. Where elements from different candidates combine well, propose the hybrid — the requester wants a strong read, not a menu.

Done when the comparison ends in one recommended design (or one named hybrid), recorded in the work item beside the framing.
