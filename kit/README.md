# Audit kit for v6 contrast sets

Batch preparation for the same construction audit used by both installed skills.
Use [the shared rubric](../codex/audit-contrast-set/rubric_v6.md); the Claude copy
is identical. The kit does not contain a separate rubric.

| File | What |
|---|---|
| `slice_auditor_prompt_v6.md` | Per-slice wrapper. Fill `{RUBRIC_PATH}`, `{SLICE_PATH}`, `{OUT_PATH}`, `{N}`. |
| `audit_prep_v6.py` | Renders a run root into `sets.md`, `pairs.tsv`, `funnel.json` and `slices/slice_N.md` (24 sets each). |

## Run

From the repository root, with explicit run and output paths:

```bash
python3 kit/audit_prep_v6.py /path/to/run /path/to/audit/prep
```

For style-refresh runs without `sampling/`, pass `--parent /path/to/parent/run`
to fill the language field. Preparation reads source runs and writes the requested
output directory; it does not call a model. Paid model audits require explicit
authorization.

For each slice, provide the wrapper with all placeholders filled. Set
`RUBRIC_PATH` to either installed skill's current `rubric_v6.md`. Collect one v4
JSONL verdict per source id, including unresolved records as deny with evidence-gap explanations. Validate each
record with the skill helper before importing or aggregating it.

## Decisions and historical comparisons

Use only `decision: approve|deny`; do not emit a separate verdict category.
Approve requires a set to meet the criteria as-is. Required repairs, failed
criteria, and insufficient evidence all require deny, with the reason explained.
Use `revision_needed` for repair instructions and `evidence_gap` for uncertainty;
a denial with an explained gap may keep unassessable H/S findings null. Do not
convert missing evidence into a demonstrated semantic defect. Behaviour labels
remain out of scope. Historical v1/v2/v3 records are preserved without rewriting.
The approval rate counts approve only, and status separates new binary decisions
from historical verdicts. Review-app v4 import compatibility remains unverified.

## Evidence in slices

Slices contain original prompt text, anchor metadata, and H/S planner diagnostics.
They omit family labels and family-selection rationale. `pairs.tsv` and
`funnel.json` retain family metadata for provenance; they are not audit inputs.
Plans and similarity/Jaccard scores are diagnostic claims, never verdict thresholds.
Verify all five roles, even in runs that regenerated only S.

These run slices include construction metadata and are not a blinded evaluation
packet. For an independent performance test, follow the skill's separate protocol
for prompt-only inputs and saving predictions before joining human labels.
