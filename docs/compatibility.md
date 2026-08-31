# Compatibility — both lanes on CMP 170HX

Claim labels used throughout: **measured** (run on target hardware),
**inferred** (derived by us from reported facts), **externally reported**
(stated in a model card / PR / issue), **untested** (no data on target
hardware).

Target hardware, generic labeling: 4× NVIDIA CMP 170HX, GA100, SM 8.0,
64 GiB per card, 180 W per card. No private hostnames, IPs, UUIDs, or PCI
maps are recorded in this repo.

Verification dates: fetch-time checks 2026-08-30; refreshed against
upstream sources **as of 2026-08-31** (see MANIFEST.md re-verification
block). No claim below is measured on target hardware.

## Lane A — FastVideo BF16 baseline

| # | Claim | Label |
|---|---|---|
| A1 | CMP 170HX is a GA100 part with compute capability 8.0 (SM 8.0) | measured (standard tooling on the target node, preparation time 2026-08-30) |
| A2 | The pinned checkpoint was trained/served with an **sm100a** VSA kernel on B200; sm100a cubins cannot run on SM 8.0 | externally reported for the first half; inferred for the second half (PTX/SASS incompatibility is standard CUDA behavior) |
| A3 | The model card documents a non-Blackwell route via `--vsa-kernel triton` (VSA-H3, tile 64, sparsity 0.9 computed by a Triton kernel instead of the sm100a kernel) | externally reported |
| A4 | FA4 attention must be disabled on non-Blackwell: `--no-fa4` | externally reported |
| A5 | Replicated DiT must be disabled for this topology: `--no-replicated-dit` | externally reported (the flag combination is documented; the internal reason for this checkpoint is inferred) |
| A6 | The model has 56 attention heads and the GPU count must divide 56 | externally reported (rule + head count) |
| A7 | 4 divides 56 (56/4 = 14 heads/card) → the 4-card node satisfies the rule; 3 GPUs would not | inferred (arithmetic on A6) |
| A8 | Training contract SP=4 matches a 4-card inference topology | inferred |
| A9 | 5-point sigma grid → exactly 4 transformer forwards per sample (relevant for latency accounting) | externally reported |
| A10 | The triton VSA route actually runs correctly, and at usable speed, on SM 8.0 | **untested** — this is the single largest Lane A risk |
| A11 | FastVideo main HEAD as of 2026-08-31 is `620bc36dc44fab81e6c266445b61c45d6a8b18ad`; the non-Blackwell flags (`--vsa-kernel`, `--fa4`, `--replicated-dit`) are still present in `examples/inference/basic/basic_fasth3.py` at that main | verified against source (2026-08-31) |
| A12 | In `fastvideo-kernel`'s `build.sh` at current main, the ThunderKittens kernels compile **only for SM90 (Hopper)**; all other archs — including SM 8.0/Ampere — take the non-TK build path, which the script explicitly notes is fine for non-SM90/ROCm targets. Arch detection uses `TORCH_CUDA_ARCH_LIST` or a torch probe; SM 8.0 maps to `CMAKE_CUDA_ARCHITECTURES=80` | verified against source (2026-08-31) |
| A13 | The `fasth3` extra in FastVideo's `pyproject.toml` at current main is `["flash-attn-4", "fastvideo-kernel==0.3.5; sys_platform == 'linux'"]`. FA4 is explicitly disabled on this hardware via `--no-fa4`; whether `flash-attn-4` supports SM 8.0 at all remains **unverified/untested** | verified against source for the pin (2026-08-31); **untested** for SM 8.0 support |
| A14 | A code search over the FastVideo repository finds exactly one `sm80` reference: `fastvideo-kernel/csrc/turbodiffusion/gemm/kernel.hpp`, unrelated to VSA. The Triton VSA kernel is Python/Triton, so no SM 8.0 CUDA kernel is required for the A3 route | verified against source for the search result (2026-08-31); inferred for the "no SM 8.0 CUDA kernel required" conclusion |

Reading: Lane A is *compatible on paper* — every known blocker has a
documented flag — but nothing above A10 has been demonstrated on CMP
hardware. Triton generates PTX for the local arch at run time, so A10 is
plausible but unproven (inferred, not measured). The 2026-08-31 refresh
(A11–A14) narrows the build-path question — the kernels build routes SM 8.0
away from ThunderKittens by design, and no VSA-related SM 8.0 CUDA kernel
exists to fail — but does not change A10's untested status.

## Lane B — ComfyUI INT8 convrot

| # | Claim | Label |
|---|---|---|
| B1 | The INT8 checkpoint + ConvRot VAE require ComfyUI **0.31.0** (INT8 sol-attention and ConvRot VAE support) | externally reported |
| B2 | ComfyUI PR #15958 is a **draft** PR; behavior may change; harness pins head `10febb01d7be73d1491cf5e5347b5ab8b6c2c09e` | externally reported |
| B3 | comfy-kitchen PR #117 is **merged** (2026-08-29, merge commit `dae00a13d458876570804523ae045a487fd92961`) | externally reported |
| B4 | Draft status means B2's head can be invalidated upstream at any time; the pin is a snapshot, not a guarantee | inferred |
| B5 | The checkpoint's sm_80+ support claim is community-reported | externally reported for the claim |
| B6 | The INT8 path runs correctly on SM 8.0 / CMP 170HX | **untested** — the only evidence is B5 |

Reading: Lane B's tooling risk is *pin stability* (draft PR), and its
hardware risk is *unverified sm_80 support*. Both are gated in the harness.

## Verdict table

| Question | Lane A (FastVideo BF16) | Lane B (ComfyUI INT8) |
|---|---|---|
| Native sm100a path works? | No (A2) | n/a (different runtime) |
| Documented non-native path? | Yes — triton VSA + no-FA4 + no-replicated-DiT (A3–A5) | ComfyUI 0.31.0 + pinned PR heads (B1–B3) |
| Head-divisibility satisfied at 4 GPUs? | Yes (A6–A7) | n/a (single/2-card hypothesis) |
| sm_80 support evidenced? | Documented non-Blackwell route exists; execution untested (A10) | Community report only (B5–B6) |
| Pin stability | Commit-pinned, merged code (stable) | Depends on a **draft** PR (fragile) (B4) |
| On-paper verdict | Compatible pending A10 | Compatible pending B6 + pin re-verification |
| Both lanes | **Untested on CMP 170HX until a run occurs.** License gate applies to the weights in both lanes. | |

Neither lane may be declared "compatible" in any published material until
a measured run exists; until then the correct label is *expected to work
on paper, untested*.
