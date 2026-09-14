# Phase-II laser/modulator scoping — question list and candidate directions

**Status: exploratory, per this repo's own mandate (`ai-agents/ORANGE_TEAM.md`
Hard Rule 5) — a scoped question list and candidate directions, not a
finished laser-bank design.** Every number below traces to a real,
already-committed file; tagged `[checked]` where read directly this pass,
`[citing existing doc]` where reused from earlier reconciliation work in
this repo, `[new finding, unverified]` where this pass derives something
not stated anywhere else.

## 0. Starting point — what's actually locked

`gcb-spec/docs/optical_geometry_spec.md` (v1.0.0, LOCKED) `[checked, full
re-read this pass]` sizes the Phase-II integrated array. It is silent on
wavelength, laser type, or any optical-source parameter (zero grep matches
on "wavelength"/"laser"/"O-band" — reconfirmed this pass) but it **does**
lock the electrical/timing envelope the source and modulator bank must fit
inside:

| Symbol | Value | Source |
|---|---|---|
| `N` (output channels) | 64 | §1 |
| `L` (contraction lanes/fire) | 16 | §1 |
| `T_fire` | 500 ps → **2.0 GHz** | §1 |
| Operand codes driven onto the modulator bus, per fire | `L + N·L` = **1040** (16 activation + 1024 weight) | §2.1, table row "operand codes driven onto the modulator bus" |
| Operand width | 8 bit (INT8/FP8 sizing case) | §1 |

The weight slice is **not** held constant for a whole 256 ns window — each
fire consumes a *new* 16-weight slice per channel (`W[n, k+j]`, `j=0..15`)
read from the on-die weight-stationary buffer's 128-bit port at the same
2.0 GHz `T_fire` cadence (§3.3 of the spec). So the 1024 weight operand
codes genuinely change every 500 ps, not once per window — confirmed by
re-reading §2.1's own per-fire table and §3.3's `WT_BUF_RD_CLK_MHZ = 2000`
localparam, not assumed. `[checked]`

## 1. The real discontinuity: Phase-I's modulator choice cannot carry forward as-is

`gcb-spec/task3_physical_bom.md` §1 `[citing existing doc]` chose a
**linear VOA** over a LiNbO₃ MZ modulator (`TKT-028`), and the BOM's own
stated rationale is explicit: *"bandwidth only needs to cover the bench's
`fire` strobe cadence, not a high-speed telecom line rate"* — i.e. the VOA
choice was justified specifically by Phase-I's *low, unspecified* bench
cadence.

Phase-II's `T_fire` is not low or unspecified — it is **locked at 500 ps
(2.0 GHz)**. A VOA (thermo-optic or MEMS-based attenuator, µs–ms settling
in the commercial parts surveyed for Phase-I) cannot plausibly settle at a
2.0 GHz update rate. **This means Phase-II's modulator technology is not
"the Phase-I VOA, scaled up" — it is a different device class entirely**,
almost certainly a GHz-class electro-optic modulator (silicon-photonic MZI
or ring-resonator, the same general family `TKT-028` rejected for Phase-I
specifically because Phase-I didn't need the speed). `[new finding,
unverified — no gcb-spec document states this explicitly; it follows
directly from the locked T_fire number and the VOA's own bench-BOM
rationale, but no one has written "the Phase-II modulator must be an MZI
or ring" as an actual requirement anywhere]`.

**Open question 1**: does `gcb-pic-core` (the actual photonic-mesh repo)
already assume a GHz-class modulator family anywhere, or is this a genuine
gap parallel to the laser-count gap `GCB-043` already filed? Not checked
this pass — `gcb-pic-core` is outside this repo's assigned scope (Hard
Rule 4 covers the combiner specifically; the modulator-bank question is
this repo's own mandate item 3, so it's in scope, but a `gcb-pic-core`
grep has not yet been run).

## 2. The real scale-up: modulator/driver *count*, not just speed

Phase-I's BOM (§1 of `task3_physical_bom.md`) specifies **one** modulator
and **one** DAC-driver channel (AD9767-class, one dual-channel part).
Phase-II's own locked table above requires **1040** independently
addressable modulator elements (1024 weight + 16 activation), each paired
with its own 2 GS/s-class analog drive path feeding an 8-bit operand onto
the optical carrier every fire.

That is a **~130×** jump in modulator/driver channel count from Phase-I's
largest configuration (N=8, per `bench_emulation_parameters.md`) — not a
proportional scale-up of the same bench architecture, a different order
of engineering problem: 1040 parallel GHz-class driver channels is closer
to a `gcb-rtl`/ASIC driver-array problem than a bench BOM line item.
`[new finding, unverified]` — flagged as a real open question, not
resolved here per this repo's Hard Rule 5 (exploratory only).

**Open question 2**: is a 1040-channel discrete driver bank ever
realistic, or does this force a different encoding — e.g. time-
multiplexing fewer physical modulators across multiple logical lanes
within one `T_fire`, or moving the DAC function on-die next to the
modulator array rather than as discrete parts? `optical_geometry_spec.md`
itself is silent on *how* the 1040 operand codes reach "the modulator
bus" — its own §2.1 table literally ends that row with "modulator DACs
(black box past here)" — so this is a real, acknowledged black box in the
locked spec itself, not something this repo is inventing a gap in.

## 3. The real, quantifiable link-budget flag — reusing `gcb-lumen`'s sibling result

Distributing a single-wavelength CW source's power to 1024+ modulation
sites requires either many independent lasers or a deep passive splitting
tree. `TKT-033` `[citing existing doc]` already established, decisively,
that this program's real architecture is single-wavelength power-summing
— no WDM anywhere in `gcb-spec` — so a Phase-II source almost certainly
inherits that same single-λ constraint rather than reopening it (Hard
Rule 1 forbids reintroducing a WDM framing without new evidence; this
scoping pass does not attempt to supply that evidence).

`gcb-lumen`'s real, resolution-converged FDTD result (`FINDINGS-2026-09-
13-fdtd-v5.md`, `DECISION_LEDGER.md` `LUMEN-FDTD` row) `[citing existing
doc]` measured ≈1.96 dB excess loss **per 2:1 MMI junction** in this
program's own combiner geometry. Applying that same per-junction figure to
a *distribution* tree (same physics, splitting instead of combining, and
loss is symmetric either direction for a passive MMI) sized to reach 1024
leaf sites from one source:

```
stages = log2(1024) = 10
excess loss  ≈ 10 × 1.96 dB ≈ 19.6 dB          [new finding, unverified]
```

That is **on top of** the ~1.5–2.0 dB VOA/isolator-class losses Phase-I's
own budget already carries per stage element, and on top of ordinary
waveguide propagation/coupling loss not modeled here at all. **A ≈20 dB
splitting-tree loss floor, before any other loss source, is a serious
real red flag for whether single-source, all-passive-splitting is
buildable at N·L=1024** — this is not a new experiment, it is the
straightforward arithmetic consequence of a result this program already
has, applied to a distribution problem instead of a combining problem.
Flagged as a genuine link-budget question for whoever eventually owns
Phase-II's optical-source architecture decision — **not** resolved or
decided here (same posture as `GCB-043`).

**Open question 3**: does Phase-II need multiple regional/local laser
sources (trading the single-source assumption for a shorter, shallower
local splitting tree per source) rather than one global source feeding
the full 1024-way tree? This is the most direct mitigation the arithmetic
above suggests, but it reopens the laser*-count* question `GCB-043`
already filed for Phase-I at a much larger scale — worth noting as the
same underlying tension recurring, not a coincidence.

## 4. Wavelength / laser-type carry-forward — an assumption, not a spec

Phase-I's real citations (`task3_physical_bom.md` §1) specify 1310 nm
O-band DFB. Nothing in `gcb-spec` states or implies Phase-II keeps that
same wavelength — it is the only concrete wavelength this program's docs
commit to anywhere (per `TKT-033`'s own finding), so it is the reasonable
default to carry forward, but it is **not itself a Phase-II requirement**
until someone writes it down as one. `[new finding, unverified — a
carried-forward assumption, not a citation]`.

## 5. What this scoping pass deliberately does NOT do

Per Hard Rule 5 and this repo's own mandate: no specific laser part,
modulator part, or driver IC is proposed here (unlike Phase-I's `[TBD]`/
`[CANDIDATE]` BOM entries — Phase-II has no locked bench geometry to size
components against yet, only the digital-side envelope in §0 above). No
attempt is made to resolve open questions 1–3, or to decide between the
single-source and multi-source distribution architectures in §3 — those
are real architecture decisions for the Architect / `gcb-pic-core`'s own
future work, following this program's standing discipline (`TKT-031`,
`GCB-043`) against unilaterally picking a side in a real conflict.

## 6. Summary — scoped question list

1. Does anything in `gcb-pic-core` already assume a specific GHz-class
   modulator family for Phase-II, or is this a genuine unfilled gap
   (§1)? — not checked this pass, `gcb-pic-core` grep is the natural next
   step.
2. How do 1040 operand codes/fire actually reach "the modulator bus" —
   1040 discrete driver channels, time-multiplexed reuse, or an on-die
   driver-array approach (§2)? `optical_geometry_spec.md` itself leaves
   this an explicit black box.
3. Single global source + deep passive splitting tree (~20 dB floor,
   §3), or multiple regional sources with shallower local trees — a
   real link-budget tradeoff, unresolved.
4. Does Phase-II keep the 1310 nm O-band wavelength, or is that only a
   Phase-I-specific choice (§4)? — currently an assumption, not a
   requirement.

## Related

`gcb-spec/docs/optical_geometry_spec.md` (LOCKED v1.0.0, this pass's
primary source), `gcb-spec/task3_physical_bom.md` §1 (Phase-I modulator
rationale being contrasted against), `TKT-028` (VOA-over-MZ decision, not
reopened — cited for its own stated rationale only),
`TKT-033-channel-spacing-conflict-and-fdtd-eme-survey.md` (no-WDM
finding, still binding), `gcb-lumen/FINDINGS-2026-09-13-fdtd-v5.md` +
`gcb-wiki/DECISION_LEDGER.md` `LUMEN-FDTD` row (the real per-junction loss
figure reused in §3), `gcb-workspaces/backlog/GCB-043-...md` (the
Phase-I laser-count conflict this scoping work does not reopen, but whose
underlying tension resurfaces at Phase-II scale in §3).
