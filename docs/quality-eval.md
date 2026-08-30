# Quality evaluation — blinded fixed-rubric protocol

Companion to [harness.md](harness.md): the harness produces the media;
this document defines how it is judged.

## Rubric template

Every output is scored 1–5 on six dimensions. Anchors are fixed before any
viewing and are not renegotiated mid-evaluation.

| Dimension | 1 (poor) | 3 (usable) | 5 (excellent) |
|---|---|---|---|
| Motion | strobing, melting, physics violations | mostly coherent, occasional artifacts | smooth, physically plausible throughout |
| Fine detail | smeared/boiling textures | readable at normal viewing | crisp, stable under scrubbing |
| Prompt adherence | ignores key prompt elements | main elements present, details wrong | all elements, correct relations/counts |
| Stability (temporal) | flicker, identity drift | minor drift, ignorable | stable identity and scene across frames |
| Audio quality | artifacts, wrong or absent where required | acceptable, minor issues | clean, appropriate to scene |
| Sync (AV) | clearly desynchronized | loose alignment | tight alignment of sound and picture |

Scores are integers; a dimension that cannot be assessed (e.g. a prompt
with no audio element) is marked `n/a` and excluded from that case's
average — never scored as a middle value.

## Blinded human inspection protocol

1. **Blinding.** Outputs from Lane A and Lane B (and repeats) are renamed
   to neutral, randomized IDs by a script; the evaluator never sees lane
   names, file names, or run metadata. The key mapping is kept outside the
   evaluation session and revealed only after all scores are locked.
2. **Fixed rubric.** The table above, same anchors, same viewing
   conditions (same player, same display, headphones for audio) for every
   session.
3. **Coverage.** All 7 benchmark categories × all repeats (≥3) are shown;
   per-case score = per-dimension median across repeats.
4. **Side-by-side vs isolated.** Primary protocol is isolated viewing
   (score each clip alone) to avoid anchoring; a secondary side-by-side
   preference pick per category may be recorded and is labeled as a
   separate, weaker signal.
5. **Recording.** Scores go into a fixed CSV schema
   (`clip_id, dimension, score, n/a flag, evaluator, session`) with no
   free-form notes that could leak lane identity; commentary is collected
   separately, blinded.

Results are only meaningful together with the harness's measured metrics;
a subjective win without matching latency/VRAM data is reported as
subjective only.

## AI-generation identifier (license requirement)

Per Section III.3(b) of the MiniMax H3 Community License (as summarized in
[MANIFEST.md](../MANIFEST.md); re-read the full text at the license
gate), outputs carry an AI-generation identifier. The harness must verify
the identifier survives in every saved/shared output file before any
output leaves the node. A missing identifier is a stop condition
(harness.md, stop condition 7).

## Publication labeling rules

- Every published score carries the protocol version, evaluator count, and
  date.
- Claims are labeled **measured** / **inferred** / **externally reported**
  / **untested**, consistent with [compatibility.md](compatibility.md).
  Human evaluation results are **measured** only if the protocol above was
  followed verbatim; deviations downgrade the claim and must be stated.
- Never present a single-run impression as a benchmark result; only
  suite-aggregated numbers with repeat counts may be called benchmarks.
- Comparative lane statements require both lanes run on the same suite
  version; otherwise publish per-lane results without ranking language.
