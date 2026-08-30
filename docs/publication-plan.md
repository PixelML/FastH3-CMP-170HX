# Publication plan — two-PR rule

## The rule

Work in this repo produces exactly two kinds of outbound artifacts:

1. **Evidence lives here.** This repository holds the full trail: run
   records, sanitized logs, metrics tables, rubric scores, sample outputs,
   and the analysis that led to conclusions. Anything needed to audit a
   claim belongs here.
2. **Findings go to PixelML/club-170hx.** A separate, concise PR distills
   the reusable findings (what worked, what the flags must be, fit
   numbers, gotchas) for people evaluating the same hardware. It links
   here for evidence and does not duplicate it.

One finding, two artifacts: a detailed evidence trail and a short reusable
summary. Never merge the two — a club PR should be readable in minutes,
and this repo should be checkable claim by claim.

## What belongs where

| Content | This repo (evidence PR) | club-170hx PR (findings) |
|---|---|---|
| Sanitized run logs, metrics CSVs | ✅ | summary table only |
| Rubric scores + protocol notes | ✅ | one-line quality verdict |
| Sample outputs (cleared, identifier intact) | ✅ | at most 1–2 illustrative links |
| Full flag/config repro steps | ✅ | exact flags, link here |
| Analysis, verdicts, claim labels | ✅ (per-claim) | conclusions only |
| Manifest/pin changes | ✅ | mention pin set briefly |

## Sanitization checklist (run before committing or commenting anything)

- [ ] No private hostnames or node names.
- [ ] No private IP addresses (including overlay/VPN addresses).
- [ ] No UUIDs, MAC addresses, serial numbers, or account identifiers.
- [ ] No exact PCI bus maps or device paths that fingerprint the node
      (generic "4× CMP 170HX" labeling only).
- [ ] No unredacted logs — logs pass through the sanitizer, and the
      sanitized result is spot-checked before commit.
- [ ] No credentials, tokens, or endpoints of any kind.
- [ ] Sample media checked: metadata stripped or verified clean, and the
      license-required AI-generation identifier still present
      (Section III.3(b)) — stripping metadata must not remove it.
- [ ] File paths in logs generalized (no usernames or mount topology that
      identifies infrastructure).

If an artifact cannot be sanitized confidently, it stays private and only
a described summary is published.

## Claim labels

Every published claim is labeled, matching
[compatibility.md](compatibility.md) / [quality-eval.md](quality-eval.md):

- **measured** — produced by a run recorded in this repo under the pinned
  harness;
- **inferred** — derived reasoning, stated with its premises;
- **externally reported** — attributed to its source, not endorsed;
- **untested** — explicitly flagged as having no data.

No claim may be published without a label; "obviously" and "should work"
are not labels.

## Release gate (all must hold before either PR is filed)

1. **License gate:** Applicable Territory of the MiniMax H3 Community
   License confirmed satisfied for the deployment jurisdiction, and the
   full license text re-read at gate time (see
   [harness.md](harness.md) checkpoint 2).
2. **Verification gate:** all pins re-verified; every downloaded file
   sha256-matched against [MANIFEST.md](../MANIFEST.md).
3. **Evidence gate:** every claim in the PR is labeled and traceable to a
   sanitized artifact in this repo.
4. **Sanitization gate:** checklist above run on the entire diff, with a
   second pass by someone other than the author.
5. **Identifier gate:** all shared outputs carry the AI-generation
   identifier.
6. **Status gate:** README status section and the tracking issue updated
   to reflect the new state before the PR goes out.
