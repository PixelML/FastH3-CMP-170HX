# FastH3 on CMP 170HX — preparation scaffold

Repo-only preparation for running FastH3 video generation on a four-card
NVIDIA CMP 170HX node (SM 8.0, 64 GiB per card, 180 W per card — generic,
public hardware labeling only; no private infrastructure identifiers appear
in this repository).

**Tracking issue:** <https://github.com/seanphan/pixelml/issues/54>
(status: **WAITING_LICENSE + WAITING_RESOURCE**)

**Current status: WAITING_LICENSE + WAITING_RESOURCE.** This repository is
documentation and planning only until both gates clear: written
jurisdiction confirmation for the MiniMax H3 Community License territory
closure, and node access. Pins re-verified against upstream sources on
2026-08-31 (see [MANIFEST.md](MANIFEST.md)).

## Why this repo exists

We want to know whether FastH3 (a 4-step distilled MiniMax-H3 video model
with VSA attention) is usable on a CMP 170HX node. CMP 170HX is a GA100
part: compute capability 8.0 (SM 8.0), not Blackwell, so the checkpoint's
native sm100a kernels cannot run and the documented non-Blackwell route
must be used. Two candidate lanes are prepared; nothing has been executed.

## Candidate lanes

| | Lane A — BF16 baseline | Lane B — Kijai INT8 "convrot" |
|---|---|---|
| Checkpoint | `FastVideo/FastVideo-FastH3-4-step-Preview-v1-VSA-DataFree` @ `b65818d41939b5085451074fe8ca8b799f8d4921` | `Kijai/MiniMax-H3-experimental` @ `f4cac997f880e93cf6940af61ee8d58ef31ff7f3` |
| Size | 147,870,258,925 B = 137.71 GiB total (verified from the revision's file listing) | 22,898,594,920 B (DiT) + 3,171,670,912 B (VAE), sha256 pinned in [MANIFEST.md](MANIFEST.md) |
| Runtime | FastVideo @ commit `48a047c05ff4138f20cfa33351499c6ec5945f5d` | ComfyUI 0.31.0 + ComfyUI PR #15958 (draft, pinned head) + comfy-kitchen PR #117 (merged) |
| Non-default flags | `--no-replicated-dit --vsa-kernel triton --no-fa4` (documented non-Blackwell path) | pinned ComfyUI / PR heads |
| Static fit (arithmetic only) | ≈ 34.4 GiB weights/card across 4 cards vs 64 GiB VRAM | ≈ 24.28 GiB total → single-card possible; 2-card topology hypothesis |
| Untested on CMP 170HX? | Yes — untested | Yes — untested (sm_80+ support is community-reported only) |

Lane B's checkpoint appears to be an INT8 repack of the same
`fastvideo_vsa_datafree` baseline (inferred from the file name; to be
confirmed before any comparative claims are made).

## License gate (read before any run)

The pinned model weights are licensed under the **MiniMax H3 Community
License**, whose Applicable Territory **excludes the EU, the UK, the
Republic of Korea, and the United States**. Use in any jurisdiction
requires confirming the deployment location is inside the Applicable
Territory **before any download or run**. The harness explicitly treats
this as a gate it must not assume away — see
[docs/harness.md](docs/harness.md) (license gate stop condition) and
[MANIFEST.md](MANIFEST.md). This repository's code/docs are Apache-2.0
(see [LICENSE](LICENSE)); that does not change the weights' own license.

## Documents

| File | Contents |
|---|---|
| [MANIFEST.md](MANIFEST.md) | Pinned revisions, byte counts, sha256, license clause summary |
| [docs/compatibility.md](docs/compatibility.md) | SM 8.0 vs sm100a, triton VSA route, head-divisibility rule, ComfyUI lane status — with claim labels |
| [docs/static-fit.md](docs/static-fit.md) | Paper arithmetic for both lanes vs 64 GiB/card; exact vs estimated numbers marked |
| [docs/harness.md](docs/harness.md) | One-command harness spec: pinned env, exact flags, benchmark suite, storage plan, gates, stop conditions |
| [docs/quality-eval.md](docs/quality-eval.md) | Blinded 1–5 rubric protocol, AI-generation identifier requirement, publication labeling |
| [docs/publication-plan.md](docs/publication-plan.md) | Two-PR rule, sanitization checklist, release gate |

## Status

**This scaffold is published for repo-only preparation. No weights have
been downloaded, no GPU run has been performed, and no packages have been
installed.** Everything here is static analysis of externally reported
facts. The next physical step (download + smoke run) is blocked on
resource availability (tracking issue above) and on the license/fit gates
in [docs/harness.md](docs/harness.md).
