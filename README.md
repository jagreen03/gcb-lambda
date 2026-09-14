# gcb-lambda

**Want to explore this with another AI chat (ChatGPT, Gemini, etc.)
that has no access to this repo?** Paste
[`EXTERNAL_AI_BRIEFING.md`](EXTERNAL_AI_BRIEFING.md) — it's
self-contained, no repo access required.

**One isolated problem: the program's own documentation says "the
laser/modulator bank has zero real spec" — that's only true for the
Phase-II scale-up. Reconcile what's actually on file for Phase I first,
then spec what's genuinely missing for Phase II.**

## The question, stated plainly — corrected twice now, each time by reading more carefully

`gcb-wiki/16_hardware_parts_systems_specification.md` Part 9 calls the
laser/modulator bank "**CONCEPT ONLY** ... the least-specified part in
the entire inventory" with "no wavelength count, channel spacing, laser
type, modulator type/bandwidth, [or] power budget ... recorded anywhere."

**That is not accurate as a blanket claim.** Real Phase-I bench content
exists. But — corrected from this repo's own first-draft reading —
**it is not "two separate, compatible experiments."** Both documents
below explicitly share the same **Task 3** designation
(`bench_emulation_parameters.md`: *"Phase I Task 3 / Objective O3"*;
`task3_physical_bom.md`: *"Task 3 — Phase-I Guaranteed Bench"*) and
`gcb-wiki/02_receiver_architecture.md`'s own "Phase I — Guaranteed
Bench" section states its purpose as proving **both** the
optical-summation **and** ratiometric-cancellation physics together,
not two separate tests. **These are two real specs for the *same*
bench, disagreeing on how its light source is built:**

1. **`gcb-spec/bench_emulation_parameters.md` §2.1–2.2**: **4 to 8
   separate** O-band (~1310 nm) DFB-or-Fabry–Pérot laser diodes, **one
   per leaf channel**, each directly, individually intensity-modulated
   (8-bit/256-level) by its own DAC channel, each TEC-stabilized,
   feeding the fused-fiber combiner tree directly.
2. **`gcb-spec/task3_physical_bom.md` §1–§2** (section 1 is titled
   "Laser Source**s** & Modulation" but specifies exactly **one**):
   **one** O-band (1310 nm) CW DFB laser (`GCB-LSR-01`, `[CANDIDATE]`)
   through **one** linear VOA (`GCB-VOA-01`, `[CANDIDATE]`, decided
   over an LiNbO₃ MZ modulator per `TKT-028`), then split via a **1×8
   PM fiber splitter** (`GCB-SPLT-1X8`, `[TBD]`) into one reference leg
   and up to seven result-combination legs feeding what this document
   itself calls an "**8×8 unitary-mesh bench coupling emulation**" —
   explicitly labeled a passive, single-wavelength COTS-splitter proxy,
   *not* a real WDM/wavelength-multiplexed claim (so `TKT-033`'s core
   finding — no real WDM anywhere in `gcb-spec` — still stands; only
   the laser-*count* question is newly in question here).

**This is a real, unreconciled architecture question, not a reading
error on either document's part: does the Task-3 bench use N
independent laser diodes, or one laser split N ways?** Each choice has
real, different consequences (per-channel power/wavelength matching
and calibration for N independent diodes, vs. splitter insertion-loss
budget and a single point of failure for the split approach) — this
repo does not pick one. Filed as `GCB-043` for Architect disposition,
the same way `TKT-031` filed the TIA-vs-switched-charge-integrator
conflict rather than choosing unilaterally.

**What genuinely has zero spec, confirmed by direct check:**
`gcb-spec/docs/optical_geometry_spec.md` (LOCKED v1.0.0), which sizes the
real Phase-II integrated array (`N=64` channels, `L=16` contraction lanes
per fire, explicitly "no channel mux" — i.e. not wavelength-division
multiplexed), is **completely silent on wavelength, laser type, or any
optical-source parameter** (zero matches on a direct grep). Nothing
bridges either Phase-I bench design to this real, locked Phase-II
digital sizing.

## What this repo needs to actually produce

1. ~~**Reconciliation, not new invention, first**: confirm or correct
   the initial "two separate bench apparatuses" hypothesis directly
   against both source files and `02_receiver_architecture.md`.~~
   **Done, 2026-09-13**: the hypothesis was wrong — it's one bench,
   two conflicting light-source specs, not two separate experiments.
   `16_hardware_parts_systems_specification.md` Part 9 corrected to
   describe what's real for Phase I (a documentation-accuracy fix,
   same category as `GCB-040`/`GCB-041`) while the laser-*count*
   conflict itself is filed as `GCB-043`, not decided here.
2. ~~**Now**, scope the real, currently-nonexistent gap: what would
   a laser/modulator design need to look like to satisfy
   `optical_geometry_spec.md`'s locked `N=64`/`L=16` Phase-II sizing —
   starting from "is this still N discrete per-channel sources, scaled
   up 8x/16x, or does 64 channels force a genuinely different
   architecture (actual WDM, a laser array, a different combiner
   topology)?" — a real open question, not a component to spec in
   isolation from the answer.~~ **Done, 2026-09-13**: see
   [`PHASE2_SCOPING.md`](PHASE2_SCOPING.md) — a scoped question list,
   not a finished design, per this repo's own exploratory mandate. Four
   real findings: Phase-I's VOA modulator choice cannot carry forward
   (Phase-II's locked 2.0 GHz `T_fire` contradicts the VOA's own
   low-rate rationale); modulator/driver count jumps ~130× from Phase-I's
   largest config; reusing `gcb-lumen`'s real FDTD loss figure against a
   1024-way distribution tree gives a ≈19.6 dB splitting-loss floor,
   flagged as a real link-budget concern; the 1310 nm wavelength carries
   forward only as an assumption, not a stated Phase-II requirement.
   None of these are decided here — filed as open questions, same
   posture as `GCB-043`.

## Not the source of truth

`gcb-spec` is. Every claim above traces to a real file cited by name;
this repo's job is to reconcile and then extend those files, not to
re-describe them from memory (the mistake Part 9 itself appears to have
made).

See [`intent.md`](intent.md) for scope, and
[`ai-agents/ORANGE_TEAM.md`](ai-agents/ORANGE_TEAM.md) for the dense
AI-session context.
