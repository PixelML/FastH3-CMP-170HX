# Static fit — paper arithmetic, no download performed

**Status: arithmetic only. No weights have been downloaded and no VRAM
measurement exists. Every number below is derived from the pinned sizes in
[MANIFEST.md](MANIFEST.md); labels mark exact vs estimated.**

Target: 4× CMP 170HX, 64 GiB VRAM per card, 180 W per card (generic
labeling).

## Lane A — BF16 baseline across 4 cards

| Step | Value | Exact/estimate |
|---|---|---|
| Checkpoint total | 147,870,258,925 B = 137.71 GiB | verified against the revision's file listing at fetch time (2026-08-30); recompute per-file at download time |
| Weights per card (total ÷ 4) | ≈ 34.43 GiB/card | exact arithmetic on the exact byte total |
| VRAM per card | 64 GiB | exact (spec) |
| Headroom per card (64 − 34.43) | ≈ 29.57 GiB/card | estimate |

The ≈ 29.6 GiB/card headroom must additionally cover (all estimates, none
measured):

- activations for 4 transformer forwards per sample (5-point sigma grid),
- latent video tensors (resolution-dependent, not yet pinned),
- text-encoder weights + activations (component not separately pinned),
- VAE decode buffers,
- sequence-parallel comms buffers at SP=4,
- CUDA context + allocator fragmentation.

Rule of thumb we will apply at the fit gate: if estimated
weights + activations exceed ~85% of 64 GiB (≈ 54.4 GiB) per card, the
lane is not attempted without a revised plan. This threshold is a working
convention (inferred), not a measured limit.

## Lane B — Kijai INT8 convrot

| Step | Value | Exact/estimate |
|---|---|---|
| DiT file | 22,898,594,920 B | exact (pinned bytes) |
| DiT in GiB | 22,898,594,920 / 2³⁰ = **21.33 GiB** | exact conversion (2 dp) |
| VAE file | 3,171,670,912 B | exact (pinned bytes) |
| VAE in GiB | 3,171,670,912 / 2³⁰ = **2.95 GiB** | exact conversion (2 dp) |
| Both files | 26,070,265,832 B = **24.28 GiB** | exact conversion (2 dp) |
| Single-card headroom (64 − 24.28) | ≈ 39.72 GiB | estimate (excludes any unpinned components, see caveat) |

Caveat (estimate): the pinned byte counts cover the DiT + VAE only. A text
encoder or other runtime components would add to the footprint and are not
yet pinned; the fit gate must resolve this before download.

### Single-card possibility

24.28 GiB total weights < 64 GiB VRAM → **Lane B plausibly fits on one
card** with ≈ 39.7 GiB of headroom (estimate). This is the simplest
topology and the first thing to try.

### Two-card topology hypothesis (hypothesis — untested)

If single-card throughput is insufficient, a 2-card split of the DiT gives:

| Item | Value | Exact/estimate |
|---|---|---|
| DiT weights per card (22,898,594,920 B ÷ 2) | ≈ 10.66 GiB/card | arithmetic on exact bytes → exact to 2 dp |
| + VAE colocated on one card | +2.95 GiB → ≈ 13.62 GiB on that card | exact to 2 dp |
| Headroom on the heavier card (64 − 13.62) | ≈ 50.38 GiB | estimate |

Motivation: large activation headroom and fewer cards involved than the
4-card Lane A topology, i.e. a cheaper bring-up (inferred). Whether the
pinned ComfyUI heads support sharding this checkpoint across 2 cards is
**untested** and must be answered before this hypothesis is promoted to a
plan.

## Power note (spec arithmetic)

180 W/card × 4 cards = 720 W for accelerator power alone, excluding host —
well inside typical node budgets, but per-card power is a captured metric
in the harness so the 180 W cap can be checked against actuals (see
[docs/harness.md](harness.md)).

## Storage location (binding)

All weights and caches MUST live under `/library/models` (the canonical
model library mount), in a FastH3-specific subdirectory
(`/library/models/fasth3/`). They must never be placed under this repo
working tree, the home directory, `/tmp`, or the system root disk. This is
a storage-policy requirement, not a preference.

Estimated total footprint for the pinned artifacts of both lanes is
**≈ 190 GB** (Lane A ≈ 137.71 GiB + Lane B ≈ 24.28 GiB, plus an allowance
for the unpinned text-encoder/overhead components — estimate). The
harness's phase-0 storage precheck (see
[docs/harness.md](harness.md)) verifies at least this much free space
before any download may start.

## Fit gate (binding)

**No download has occurred.** Before any download is allowed:

1. License gate passed (Applicable Territory confirmed for the deployment
   jurisdiction — the harness must not assume it).
2. Every `externally reported` pin re-verified against source; exact
   per-file byte counts recorded for Lane A (replacing the 137.71 GiB
   rounded figure).
3. Unpinned components for the chosen lane identified and sized.
4. The numbers in this document refreshed, and the 85%-of-VRAM working
   threshold checked.
5. Storage precheck passed (phase-0 in [docs/harness.md](harness.md)):
   `/library` mounted read-write, ≥ 190 GB free, and the system root disk
   > 10% free.

Until all five hold, the fit gate remains **closed**.
