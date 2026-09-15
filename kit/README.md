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
`RUBRIC_PATH` to either installed skill's current `rubric_v6.md`. Collect one v3
JSONL verdict per source id, including unresolved records as HOLD. Validate each
record with the skill helper before importing or aggregating it.

## Decisions and historical comparisons

Use PASS/approve, MINOR/revise, FAIL/reject, and HOLD/hold. MINOR requires
`revision_needed`; HOLD requires `hold_reason` and may use null for unknown H/S
findings. Do not evaluate or correct behaviour-family labels. Historical v1/v2
records remain unchanged; the new approval rate counts PASS alone, so it is not
directly comparable to a historical keep rate pooling PASS and MINOR.

The review app's v3 import compatibility remains unverified. Never silently map
Revise/Hold into an older binary field.

## Evidence in slices

Slices contain original prompt text, anchor metadata, and H/S planner diagnostics.
They omit family labels and family-selection rationale. `pairs.tsv` and
`funnel.json` retain family metadata for provenance; they are not audit inputs.
Plans and similarity/Jaccard scores are diagnostic claims, never verdict thresholds.
Verify all five roles, even in runs that regenerated only S.

These run slices include construction metadata and are not a blinded evaluation
packet. For an independent performance test, follow the skill's separate protocol
for prompt-only inputs and saving predictions before joining human labels.
