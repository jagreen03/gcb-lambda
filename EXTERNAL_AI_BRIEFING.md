# Paste this whole file into any AI chat to get help on this problem

You don't need repo access to use this — everything needed is inlined
below. This is one isolated problem from a larger hardware program
(GreenCoyote-Bay, a photonic/electronic AI-inference accelerator); you
are being asked to help with exactly this one piece, nothing more.

## The problem, in one sentence — updated 2026-09-13, the reconciliation below is DONE

The program's own documentation used to call the laser/modulator light
source "CONCEPT ONLY... the least-specified part in the entire
inventory" — that blanket claim has been corrected. **What's actually
true, confirmed by careful reading**: two real Phase-I bench documents
describe the *same* bench (not two separate sub-experiments, as an
earlier pass here had guessed) but disagree on laser **count** — filed
as a real architecture conflict for human disposition, not resolved
by an AI. **The remaining open question is different and narrower**:
what would a laser/modulator design need to look like for the real,
locked *Phase-II* 64-channel scaled target, which has genuinely zero
spec either way. See the bottom of this file for that actual question
— read the reconciliation below for context, but don't spend effort
re-litigating it; it's settled.

## What already exists — two real designs, quoted in full

**Design A — the combiner-tree test bench** (from
`gcb-spec/bench_emulation_parameters.md` §2.1–2.2, verbatim):

> ### 2.1 Laser sources (leaves)
>
> | Item | Spec |
> |---|---|
> | Count | 4 (min) to 8 (target), **one per leaf channel** |
> | Type | O-band DFB **or** Fabry–Pérot laser diode, ~1310 nm |
> | Per-channel optical power at full digital value | 0.1 mW (100 µW) at the combiner input, matched across channels |
> | Modulation | Intensity modulation linear in the driving digital value, 8-bit (256-level) resolution |
> | Power matching | Channel-to-channel spread trimmed by a per-channel gain-trim path; target pre-trim spread ≤ ±5% |
>
> **Temperature stabilization.** DFB and FP diodes drift in both
> wavelength (~0.1 nm/K for DFB) and output power with junction
> temperature. Each source needs a TEC holding the case to ±0.1 K over
> a measurement run.
>
> ### 2.2 Modulation / drive
>
> | Item | Spec |
> |---|---|
> | Drive | One DAC channel per laser (or a multi-channel DAC board), ≥8-bit, update rate ≥ the ADC sample rate |
> | Linearity | Digital value → optical power transfer characterized per channel; residual nonlinearity folded into the numerical model |
> | Bias | Above threshold, in the linear L–I region |

This is **N independent laser diodes** (4 to 8 of them), each directly,
individually intensity-modulated, feeding a fused-fiber power-summing
combiner tree — the apparatus for testing "does the optical summation
mechanism work."

**Design B — the readout-calibration bench** (from
`gcb-spec/task3_physical_bom.md` §1, the BOM for "the two matched
PD+TIA detection chains"):

> | Item | Spec | Candidate part |
> |---|---|---|
> | CW laser source | O-band, 1310 nm center wavelength; DFB (single-longitudinal-mode, narrow linewidth); fiber-pigtailed, singlemode, FC/APC; ≥ a few mW output | `GCB-LSR-01` [CANDIDATE], target +3 dBm |
> | Optical isolator | 1310 nm band, in-line fiber-pigtailed; isolation ≥ 30 dB single-stage | `GCB-ISO-01` [TBD] |
> | Optical intensity modulator | O-band compatible; linear intensity control, ≥8-bit effective resolution; driven from a 14-bit parallel DAC word | `GCB-VOA-01` [TBD] — **a linear VOA, decided over an LiNbO₃ MZ modulator** (see decision rationale below) |
> | DAC-to-modulator driver | 14-bit parallel DAC (Analog Devices AD9767-class) + a precision op-amp current-to-voltage/scaling stage | both [CANDIDATE — parametric fit] |

**Decision already made and locked** (ticket `TKT-028`, do not
re-open): **VOA over Mach-Zehnder modulator**, because (1) bandwidth
only needs the bench's low-rate "fire" strobe cadence, removing MZ's
speed advantage; (2) MZ's cosine-squared response needs an active
quadrature-bias servo this bench doesn't otherwise need; (3) a VOA's
attenuation-vs-control-voltage curve is smooth/monotonic — a more
direct path to 8-bit-linear control.

This is **ONE shared laser**, externally intensity-modulated by one
VOA, split via a 1×8 PM splitter into a reference leg and
result-combination legs feeding (confirmed by cross-checking
`gcb-wiki/02_receiver_architecture.md`'s "Phase I — Guaranteed Bench"
section) a **reference-tap / result-tap ratiometric-cancellation
readout**.

## Resolved — this was the same bench, not two different ones

**An earlier pass here guessed Design A and Design B were two
different, compatible sub-experiments. That guess was wrong, corrected
by reading more carefully**: both documents explicitly label themselves
"Task 3," and `02_receiver_architecture.md`'s own "Phase I — Guaranteed
Bench" section states one combined purpose (proving optical-summation
*and* ratiometric-cancellation together) — this is **one bench**, and
the two documents simply **disagree on how many lasers it uses**: N
independent diodes (Design A) vs. one diode split N ways (Design B).
That conflict has been filed as a real architecture question for human
disposition (this program's `GCB-043`) — **not something to resolve
here**; don't re-litigate it.

## What genuinely has zero spec, confirmed

`gcb-spec/docs/optical_geometry_spec.md` (status: **LOCKED**, v1.0.0)
sizes the real target integrated array: **N=64** parallel optical
output channels (1:1 with 64 electronic readout channels, explicitly
"no channel mux" — i.e., not wavelength-division multiplexed, every
channel at essentially one wavelength band), and **L=16** contraction
lanes combined per channel per firing event. **This document contains
zero mentions of wavelength, laser, or O-band** (confirmed by direct
text search) — it specifies the digital/electrical sizing completely
and says nothing about what light source would actually drive 64
channels at this scale. Neither Design A nor Design B above addresses
64 channels — both are Phase-I bench setups at 4-8 inputs.

**A separate, already-flagged concern, for context, not yet resolved
either:** an earlier pass at this program's photonic-core repo
described "an 8×8 unitary mesh" and wavelength-division multiplexing
for this same component — that framing was checked against the actual
spec documents and found to have **no supporting citation anywhere**
in them. Treat any "WDM" or "wavelength-multiplexed mesh" framing you
encounter for this system as unconfirmed unless it traces to a real
spec document — the locked geometry spec above explicitly says "no
channel mux."

## The actual question to help with — the reconciliation above is done, this is what's left

**Scope the real gap**: starting from the locked `N=64`/`L=16` digital
sizing, with *zero* existing optical-source spec to build from (neither
Design A nor Design B addresses 64 channels — both are Phase-I setups
at 4–8 inputs) — does scaling either Phase-I design (N≈8) up to N=64
stay architecturally the same (just "more of the same lasers/VOAs," in
whichever count Design A/B's conflict eventually resolves to), or does
64 channels force a genuinely different approach (an actual laser
array, integrated modulators, or something else)? What are the real
engineering tradeoffs at that scale — thermal management for 64
co-located sources, power budget, calibration complexity across that
many channels, and whether the N-vs-1-split question even stays
meaningful at 8x the scale?

**House rule**: cite the exact document/section for every claim. Don't
assume wavelength-division multiplexing or any "8×8 unitary mesh"
framing beyond what's directly supported by the locked spec above,
which explicitly says "no channel mux." "I don't know, here's what
would need deciding first" is a completely valid answer — this
genuinely is an open question, not one with a hidden right answer.
