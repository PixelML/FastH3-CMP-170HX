# Harness spec — one command per lane, fully gated

Status: **spec only** — nothing here is implemented or run yet (scaffold is
repo-only preparation; tracking issue: the WAITING_RESOURCE state described
in the README). The harness, when built, must honor every gate and stop
condition below.

## One-command contract

```bash
./harness/run.sh lane-a            # Lane A: FastVideo BF16 baseline
./harness/run.sh lane-b            # Lane B: ComfyUI INT8 convrot
```

One invocation per lane; no interactive steps except the license-gate
confirmation (below). All other parameters come from pinned files in the
repo, never from the operator's environment.

## Pinned environment

| Lane | Pin |
|---|---|
| A | FastVideo @ commit `48a047c05ff4138f20cfa33351499c6ec5945f5d`; checkpoint `FastVideo/FastVideo-FastH3-4-step-Preview-v1-VSA-DataFree` @ `b65818d41939b5085451074fe8ca8b799f8d4921` |
| B | ComfyUI @ `0.31.0` + PR #15958 head `10febb01d7be73d1491cf5e5347b5ab8b6c2c09e`; comfy-kitchen @ merge `dae00a13d458876570804523ae045a487fd92961`; checkpoint files + sha256 from [MANIFEST.md](../MANIFEST.md) |

The harness records the resolved environment (component versions, commit
heads, GPU name/count/driver as reported by standard tooling) into the run
record. Python/package versions are to be pinned in a lock file at
implementation time; until then the environment is **not** considered
reproducible and no benchmark numbers may be published from it.

## Exact flags

### Lane A (FastVideo, 4 GPUs)

Documented non-Blackwell path — all three flags mandatory:

```
--no-replicated-dit
--vsa-kernel triton
--no-fa4
```

Plus: `num_gpus=4` (satisfies the 56-head divisibility rule, 56/4 = 14).
Inference config mirrors the training contract: 5-point sigma grid (4
transformer forwards), VSA-H3 tile 64 sparsity 0.9. Resolution/steps
settings to be pinned in the run config at implementation time and then
frozen for all runs in the suite.

### Lane B (ComfyUI)

ComfyUI started at the pinned version + PR head; comfy-kitchen at the
pinned merge commit. Workflow graph serialized and stored in the repo so
the exact graph — INT8 sol-attention loading, ConvRot VAE, sampler
settings — is reproduced bit-for-bit per run. No ad-hoc graph edits during
a benchmark suite.

## Benchmark suite outline

Fixed prompt + seed list, identical across lanes so outputs are
comparable. Seven categories; exact prompt strings and seeds are frozen in
`harness/benchcases.*` at implementation time and never edited afterward:

1. motion (large coherent movement),
2. fine detail (texture/textures close-up),
3. multi-subject (≥2 subjects interacting),
4. camera movement (pan/track/dolly),
5. prompt adherence (unusual attribute binding / counting),
6. difficult audio (dense or acoustically complex scene),
7. AV sync (events that must align sound and picture).

Protocol per case, per lane:

- 1 warmup run (discarded, warms caches/kernels),
- ≥ 3 measured repeats, same prompt/seed, media saved every repeat,
- identical settings across lanes; nothing tuned per-lane after the first
  suite starts.

## Captured metrics (per repeat)

- end-to-end wall-clock and per-stage latency (text encode / forwards /
  VAE decode),
- peak VRAM per card,
- power draw per card (checked against the 180 W/card cap) and thermals,
- forward-pass count (expect 4 transformer forwards per sample),
- output media file + sha256,
- full console log, sanitized per [publication-plan.md](publication-plan.md)
  before leaving the node.

## Storage plan

- Weights live on the **shared model storage volume**; never the system
  root, never inside this repo working tree. `*.safetensors` is git-ignored
  here as a second line of defense.
- Every downloaded file is verified against the MANIFEST sha256 before
  first use; verification state is recorded once, not per-run.
- Outputs land in `outputs/` (git-ignored); only sanitized summaries,
  metrics tables, and explicitly cleared sample media may be committed.

## Checkpoint boundary (phased execution)

The harness runs in phases; each phase writes a checkpoint record
(timestamp, resolved pins, pass/fail) before the next may start:

1. **pin-verify** — every `externally reported` pin re-verified against
   source; any drift = stop.
2. **license gate** — Applicable Territory of the MiniMax H3 Community
   License confirmed satisfied **for the deployment jurisdiction**, with
   an explicit operator confirmation recorded. The harness must not
   assume this; there is no flag that bypasses it silently.
3. **fit gate** — [static-fit.md](static-fit.md) refreshed and its
   threshold checked for the chosen lane.
4. **download + hash** — fetch to shared model storage; sha256 every file.
5. **smoke run** — single low-cost generation; only after success may the
   benchmark suite start.

## Stop conditions (any one halts the lane, no partial publishing)

1. License Applicable Territory unconfirmed for the deployment
   jurisdiction (gate 2 refused or unanswerable).
2. Any pin drift: upstream revision/PR head/merge commit changed, or a
   re-read license text differs from the MANIFEST summary.
3. Any downloaded file fails sha256/byte-count verification.
4. Fit-gate numbers exceeded at runtime: OOM, or per-card VRAM demand over
   the 85% working threshold in a way the fit doc did not anticipate.
5. Per-card power sustained above the 180 W cap or thermals outside the
   node's operating envelope.
6. Repeated nondeterministic failures of the lane's runtime (e.g. the
   draft ComfyUI PR head breaking) — record, stop, re-pin.
7. Any evidence that output media would be published without the
   AI-generation identifier required by license Section III.3(b).
8. Any log/output that cannot be sanitized confidently is treated as
   private and stops publication, not the run.
