# MANIFEST — pinned artifacts

Every pin below is labeled either **externally reported** (taken from a
model card, issue, or other secondary source and not independently checked
against the upstream source) or **verified against source at fetch time**
(checked against the upstream source listing / API on 2026-08-30). Pins
that are only *externally reported* **must be re-verified against source
before being used as a gate condition**. No credentials, tokens, or
private endpoints appear in this file.

## Lane A — BF16 baseline checkpoint

| Item | Pin | Label |
|---|---|---|
| HF repo | `FastVideo/FastVideo-FastH3-4-step-Preview-v1-VSA-DataFree` | verified against source at fetch time |
| Revision | `b65818d41939b5085451074fe8ca8b799f8d4921` | verified against source at fetch time (resolved as repo HEAD at fetch) |
| Total size | 137.71 GiB = 147,870,258,925 B summed from the revision's file listing | verified against source at fetch time — recompute per-file at download time |
| Training-side contract | fastvideo commit `48a047c05ff4138f20cfa33351499c6ec5945f5d` | verified: commit exists upstream (2026-08-30); contract values read from the revision's own `fastvideo_inference.json` |
| Sigma schedule | 5-point sigma grid = 4 transformer forwards | externally reported |
| Attention config | VSA-H3, tile 64, sparsity 0.9 | externally reported |
| Training kernel | sm100a VSA kernel, on B200 | externally reported |
| Sequence parallel | SP=4 | externally reported |

## Lane B — Kijai INT8 convrot checkpoint

Repo: `Kijai/MiniMax-H3-experimental` (externally reported)

| File | Bytes | sha256 | Label |
|---|---|---|---|
| `minimax_h3_fastvideo_vsa_datafree_1300step_4step_int8_convrot.safetensors` | 22,898,594,920 | `7221ae65d78780354d51e5048d29728d9f1f8fb9baf50b1dd3df85f5101413d3` | verified against source at fetch time |
| `minimax_h3_video_vae_int8_convrot.safetensors` | 3,171,670,912 | `9bb2d96f218c76babd85e0611b85ca8fb330a90546c01a0005e8a58a59593410` | verified against source at fetch time |
| Repo revision | `f4cac997f880e93cf6940af61ee8d58ef31ff7f3` | — | verified against source at fetch time (resolved as repo HEAD at fetch) |

Note: the pinned byte counts cover the two named files only. Any additional
components a runtime needs (e.g. a text encoder) are **not pinned** and
must be identified and pinned before the fit gate can be considered final.

## Runtime pins

| Item | Pin | Label |
|---|---|---|
| FastVideo (Lane A) | commit `48a047c05ff4138f20cfa33351499c6ec5945f5d` | verified: commit exists upstream |
| ComfyUI (Lane B) | version `0.31.0` | externally reported |
| ComfyUI PR #15958 | **draft** PR; pinned head `10febb01d7be73d1491cf5e5347b5ab8b6c2c09e` | verified against source at fetch time (API state: open draft) |
| comfy-kitchen PR #117 | merged 2026-08-29; merge commit `dae00a13d458876570804523ae045a487fd92961` | verified against source at fetch time (API state: merged) |

## License

| Item | Pin | Label |
|---|---|---|
| License (model weights, both lanes) | MiniMax H3 Community License | verified: full text read from the pinned baseline revision's LICENSE file |
| Applicable Territory | Excludes the EU, the UK, the Republic of Korea, and the United States | verified: full text read from the pinned baseline revision's LICENSE file |
| Output marking | Outputs carry an AI-generation identifier per license Section III.3(b) | verified: full text read from the pinned baseline revision's LICENSE file |

## Verification policy

1. Before any download: re-verify every `externally reported` pin against
   the upstream source (HF repo/revision exists, license text unchanged,
   PR/commit heads resolve) and upgrade its label.
2. After any download: recompute sha256 + byte counts for every file and
   compare against this manifest. Any mismatch is a stop condition.
3. This manifest is append-only for pins; changes to a pin require a note
   explaining what changed upstream.
