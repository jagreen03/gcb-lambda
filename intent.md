# Intent: gcb-lambda

## Scope

**Two deliverables, strictly in order:**

1. Reconcile `gcb-spec/bench_emulation_parameters.md` §2.1–2.2 (N=4–8
   per-leaf lasers, combiner-tree bench) against
   `gcb-spec/task3_physical_bom.md` §1 (single CW laser + VOA,
   readout-calibration bench) and `gcb-wiki/02_receiver_architecture.md`
   (the ratiometric reference-tap/result-tap readout they feed) —
   confirm whether these are two compatible, separate pieces of bench
   apparatus (the working hypothesis) or an actual conflict, and correct
   `gcb-wiki/16_hardware_parts_systems_specification.md` Part 9's
   blanket "CONCEPT ONLY, zero spec" claim to reflect whichever is true.
2. Scope what a Phase-II laser/modulator design would need, starting
   from `gcb-spec/docs/optical_geometry_spec.md`'s real, LOCKED `N=64`/
   `L=16` digital sizing — which is completely silent on the optical
   source side. This is genuinely open; do not assume it is "the Phase-I
   design, scaled."

## Out of scope

- Redesigning either Phase-I bench apparatus. Both are already-decided
  (`TKT-028` decided VOA-over-MZ for the readout bench specifically).
  This repo reconciles documentation and scopes the Phase-II gap; it
  does not re-open closed Phase-I decisions.
- The MMI combiner-loss extraction (that's `gcb-lumen`) and the analog
  Rx noise gate (that's `gcb-kelvin`) — distinct problems, sibling
  repos.
- Any wavelength-division-multiplexing architecture claim not
  corroborated directly in `gcb-spec`. `optical_geometry_spec.md`'s own
  "no channel mux" note is the locked ground truth until a real
  `gcb-spec` document says otherwise — do not reintroduce a "DWDM laser
  bank" framing without a citation that actually supports it (the
  mistake `TKT-033` already flagged once in the 16_hardware doc).

## Governing discipline

Same Orange-Team "Physical Reality First" discipline as `gcb-lumen`
(`gcb-workspaces/backlog/TKT-000`) — but this repo's first deliverable
is a documentation reconciliation, not a new lab measurement. Get the
record of what's *already real* correct before spending any effort
speculating about Phase II.

## Relationship to the rest of the program

Deliverable 1 is a correction to `gcb-wiki`'s own file, not a new
architecture document — apply it as a normal doc-accuracy fix (stage
only that one file, cite the real sources, commit, push), the same
pattern already used for `GCB-040`/`GCB-041` and the `DEFECT-002`/
`DEFECT-004` fix in `gcb-spec/registers.yaml`. Deliverable 2, if it
produces a real proposal, belongs in `gcb-spec` (or a new doc there),
not invented wholesale in this repo.
