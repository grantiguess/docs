# Collaboration & Cognitive Substrate Plan

> **Thesis:** Redstring is a medium for *reversible, non-destructive synthesis*. Every
> other system makes synthesis destructive (databases normalize, ontologies flatten).
> Redstring keeps the messy situated cases **and** the general structure, and lets you
> travel between them. Because of this, collaboration is **native to the data geometry**,
> not a feature bolted on: collaborators **converge on identity** (a shared canonical
> Graph) while they **diverge on interpretation** (their own Webs). This is
> "hold contested complexity" expressed as architecture.

Status tags: `[exists]` already in codebase · `[partial]` partly there · `[new]` to build.

---

## Core model: one Graph, many Webs

There is **one prototype Graph** (the canonical, deduplicated concept space — *identity*)
and **many instance Webs** (situated placements/arrangements — *situation*). "Web"
replaces the overloaded current use of "graph" for the instance/definition layer.

Key correction: the Graph is **not** a type/class abstraction. It's `Alice → Married To → Bob`
with all three as prototypes, **not** `Person marriedTo Person`. The fold dedupes instance
*placements* up to canonical concepts; it does **not** abstract Alice→Person. That Is-a /
abstraction move is a separate axis still needed independently — do not conflate fold-dedup
with abstraction.

The Graph and Webs are related by a **two-way membrane**, both directions first-class:

- **Webs → Graph = fold** (lift/aggregate instance triplets to prototype triplets)
- **Graph → Webs = hydrate** (drop prototype triplets into a Web as instances)

The two directions are **inverse-lossy** (fold loses *which instances*; hydrate must
*invent which instances*), so round-trips don't return home. **Provenance** is what makes
the membrane reversible — it is load-bearing, not a nicety.

---

## Feature list

### A. Model foundation
1. `[new]` **Web/Graph terminology split** — rename instance-layer "graph" → **Web**; reserve **Graph** for the prototype layer.
2. `[partial]` **Prototype Graph as a first-class structure** — today it's implicitly recomputed on read (ConnectionBrowser folds across graphs). Make it explicit and addressable.

### B. The fold (Webs → Graph)
3. `[new]` **Fold function** — aggregate instance triplets → prototype triplets `(protoA → predicateProto → protoB)`. Normalize directionality (undirected → symmetric directed pair).
4. `[new]` **Provenance index** — every prototype edge carries `{which Webs, instance count, author}`. Materialized/incremental, kept in a **side index** (not in the saved `.universe`, à la imageCache).
5. `[new]` **Derived-only discipline** — fold output is recomputed/incrementally maintained, never hand-authored, so deleting a Web auto-decrements its contributions (no orphan chasing).

### C. The hydrate (Graph → Webs)
6. `[new]` **Hydrate action** — drop prototype triplet(s) into a Web as instances via a match-by-prototype rule (connect to existing instance if one; ambiguous if many; create-or-skip if none).
7. `[new]` **Automation × area-of-effect dials** — manual / suggested / bulk, crossed with this-node / this-Web / many-Webs. Many-Webs is *propose, never push*.
8. `[new]` **Batch hydrate** — "add all triplets to this node," and "…across all nodes in this Web."

### D. Panel UX
9. `[partial→fix]` **Connections driven by the carousel index** — today components are per-definition but connections are flat (ConnectionBrowser ignores `nodeDefinitionIndices`). Drive both off the same definition index so the arrows move them together.
10. `[partial]` **Per-Web toggle + "All" = prototype Graph view** — replaces today's In-Graph/Universe dropdown with the principled version.
11. `[new]` **Hydrate-from-panel affordance** — "drop these into the Web" from the All view.
12. `[new]` **Observed-vs-asserted distinction + provenance display** — show "seen in N Webs" so hydration doesn't promote a one-off scribble to fact.

### E. Edge model cleanup (prerequisite hygiene)
13. `[new]` **Unify the two edge-typing systems** — fold the legacy/agent `edgePrototypes`/`typeNodeId` path into the primary `definitionNodeIds` (node-prototype) path, so there's one connection-type substrate to fold and serialize.

### F. Entity resolution (identity engine)
14. `[new]` **Per-prototype embeddings** — Transformers.js (MiniLM-class) on WASM, embedding name + bio + serialized connections. Cached in the side index, local/free (no BYOK needed).
15. `[new]` **Gated multi-feature scorer** — `score = G(name,type) · [β + (1−β)·E(bio,connections)]`.
    - **G** (veto, multiplicative/geometric): `(name_sim)^wn · (type_compat)^wt` — either near-0 tanks the score.
    - **E** (vouch, additive/clamped ≥0): `clamp01(e_b·bio_sim + e_c·conn_overlap)` — can only raise.
    - **β** ≈ 0.5–0.6: credit for a perfect name+type match with no other evidence.
    - `name_sim` = blend(string similarity, name-embedding cosine).
    - `type_compat` = `γ^d` over Is-a distance (γ≈0.7); unrelated branch → ε.
    - `bio_sim` = blend(embedding cosine [semantic], TF-IDF cosine [lexical]).
    - `conn_overlap` = **overlap coefficient** `|A∩B| / min(|A|,|B|)` — NOT Jaccard, so extra connections on one side don't penalize. Weight shared connections by provenance.
16. `[new]` **Candidate generation** — block by type, shortlist by vector cosine (ANN at scale), full-score only the shortlist.
17. `[new]` **Calibration path** — hand-weights now → logistic regression once labeled same/different pairs exist (keeps veto/vouch structure via gate input + clamped features).

### G. Identity as a claim
18. `[new]` **Soft, attributed `sameAs` edge** — contestable, owned, reversible. Identity is asserted, not globally decided.
19. `[new]` **Match-review surface** — scores → user-paced confirm/reject → `sameAs` assertions. Never auto-merge.

### H. Universe interfacing (the collaboration payoff)
> Almost every cross-universe interaction reduces to one question: *is your Alice my Alice?*
> So this is really one identity signal (F/G) + a spectrum of what to do with a match.
20. `[partial]` **Universe as the unit of sharing** — both layers + connective tissue travel together; sub-"packs" rejected as lossy.
21. `[partial]` **Clone** — fork a universe or subset; divergence begins.
22. `[new]` **Merge** — entity resolution + union of contested edges (superposition, no winner-picking).
23. `[new]` **Fold-across-universes** — a shared canonical Graph computed from multiple universes' Webs.
24. `[new]` **Transclude / reference by link** — live reference, no copy.
25. `[new]` **Subscribe / pull** — git-remote-style ongoing sync of selected clusters.
26. `[new]` **Diff** — compare two universes before merging.
27. `[exists→extend]` **Load-from-link / load-from-file transport.**
28. `[partial]` **Commit-style history** — already commit-ish via SaveCoordinator; make it a navigable history of interpretations.

---

## Cross-cutting principles (guardrails, not features)
- **Graded-propose, never auto-decide** — every score and fold *suggests*; the human/owner commits. Auto-merging on a score is the convergence-flattening the whole project refuses.
- **Divergence-preserving** — converge on identity (Graph), diverge on interpretation (Webs); contested claims coexist as superposition (union, no winner).
- **Provenance is load-bearing** — it makes fold↔hydrate reversible and the scorer honest (weights evidence by how widely attested it is).
- **Local-first / BYOK only for the LLM** — identity resolution (embeddings) is free, serverless, in-browser; respects the no-WebGPU-required architecture stance.
- **The membrane is the product** — the reconciliation surface (fold-vs-maintained conflict, orphan control, area-of-effect, match review) is invisible and undemoable, but it's where the moat and most of the remaining difficulty live. Do not under-invest because it doesn't demo.

---

## Suggested sequence
- **Phase 0 — hygiene:** #1 Web rename, #13 edge-typing unification.
- **Phase 1 — within-universe membrane:** #3–5 fold + provenance, #9–12 panel, #6–8 hydrate. *The demonstrable core and the moat.*
- **Phase 2 — identity:** #14–17 scorer, #18–19 `sameAs` + review surface.
- **Phase 3 — universe interfacing:** #20–28, roughly in order (clone/diff before merge/fold-across).

---

## Open questions
- **Connection-set key format** for overlap scoring: predicate+neighbor pair? include directionality? how is each shared connection weighted by provenance?
- **Fold-vs-maintained conflict rule:** when a standing Graph edge is no longer supported by any Web, or a fresh fold produces an edge the maintained Graph lacks — wins / coexists-tagged / surfaces-as-orphan?
- **Hydrate disambiguation UX** when a Web has zero or many instances of a target prototype.
- **Is `sameAs` symmetric/transitive?** Soft assertions that chain (A~B, B~C) raise a merge-closure question.
