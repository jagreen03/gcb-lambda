# gcb-lambda

**One isolated problem: the program's own documentation says "the
laser/modulator bank has zero real spec" — that's only true for the
Phase-II scale-up. Reconcile what's actually on file for Phase I first,
then spec what's genuinely missing for Phase II.**

## The question, stated plainly — corrected from what the existing docs claim

`gcb-wiki/16_hardware_parts_systems_specification.md` Part 9 calls the
laser/modulator bank "**CONCEPT ONLY** ... the least-specified part in
the entire inventory" with "no wavelength count, channel spacing, laser
type, modulator type/bandwidth, [or] power budget ... recorded anywhere."

**That is not accurate as a blanket claim, checked directly against
`gcb-spec` this session.** Two real, if modest, Phase-I bench laser/
modulator designs already exist — in two different files, describing
what appear to be two different pieces of apparatus, never reconciled
against each other or against Part 9's framing:

1. **`gcb-spec/bench_emulation_parameters.md` §2.1–2.2** — the
   **combiner/reduction-tree test bench**: 4 (minimum) to 8 (target)
   separate O-band (~1310 nm) DFB-or-Fabry–Pérot laser diodes, **one per
   leaf channel**, each directly intensity-modulated (8-bit/256-level)
   by its own DAC channel, each TEC-stabilized, feeding the fused-fiber
   combiner tree. Tests the "sum N optical signals" mechanism.
2. **`gcb-spec/task3_physical_bom.md` §1** — the **readout/calibration
   bench**: a single O-band (1310 nm) CW DFB laser (`GCB-LSR-01`,
   `[CANDIDATE]`) through a single linear VOA intensity modulator
   (`GCB-VOA-01`, `[CANDIDATE]`, decided over an LiNbO₃ MZ modulator per
   `TKT-028`), feeding `gcb-wiki/02_receiver_architecture.md`'s
   reference-tap/result-tap ratiometric-cancellation readout. Tests the
   detection/ENOB mechanism, not the combiner.

**These look like two different, compatible pieces of apparatus for two
different sub-experiments — not a contradiction** (one needs N
independent sources to sum; the other needs one source split to two taps
to cancel common loss). But nobody has written that down. Part 9 treats
"the laser bank" as one monolithic, unspecified thing, when Phase I
actually has two real, separately-decided designs on file.

**What genuinely has zero spec, confirmed by direct check:**
`gcb-spec/docs/optical_geometry_spec.md` (LOCKED v1.0.0), which sizes the
real Phase-II integrated array (`N=64` channels, `L=16` contraction lanes
per fire, explicitly "no channel mux" — i.e. not wavelength-division
multiplexed), is **completely silent on wavelength, laser type, or any
optical-source parameter** (zero matches on a direct grep). Nothing
bridges either Phase-I bench design to this real, locked Phase-II
digital sizing.

## What this repo needs to actually produce

1. **Reconciliation, not new invention, first**: confirm or correct the
   "two separate bench apparatuses" reading above directly against both
   source files and `02_receiver_architecture.md`, and fix Part 9 of
   `16_hardware_parts_systems_specification.md` to say what's actually
   real for Phase I (this is a documentation-accuracy correction, the
   same category as `GCB-040`/`GCB-041`'s register-map findings — not a
   new architectural decision).
2. **Only then**, scope the real, currently-nonexistent gap: what would
   a laser/modulator design need to look like to satisfy
   `optical_geometry_spec.md`'s locked `N=64`/`L=16` Phase-II sizing —
   starting from "is this still N discrete per-channel sources, scaled
   up 8x/16x, or does 64 channels force a genuinely different
   architecture (actual WDM, a laser array, a different combiner
   topology)?" — a real open question, not a component to spec in
   isolation from the answer.

## Not the source of truth

`gcb-spec` is. Every claim above traces to a real file cited by name;
this repo's job is to reconcile and then extend those files, not to
re-describe them from memory (the mistake Part 9 itself appears to have
made).

See [`intent.md`](intent.md) for scope, and
[`ai-agents/ORANGE_TEAM.md`](ai-agents/ORANGE_TEAM.md) for the dense
AI-session context.
