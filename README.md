# HARLEY gold-set helper

Tooling for independently auditing HARLEY v6 WildChat behaviour-contrast sets
(five roles: misaligned target T, aligned control A, hard negative H, style
control S, paraphrase target P). Reviewers can inspect run roots or supplied
records and optionally save verdicts. Skill v0.10.1 uses the v4 verdict schema with revised acceptance rules;
see the compatibility notes below before importing those records into a review app.

## Layout

| Path | What |
|---|---|
| `claude/audit-contrast-set/` | Skill v0.10.1 for Claude Code. Install into `~/.claude/skills/`. |
| `codex/audit-contrast-set/` | Identical skill v0.10.1 for Codex. Install into `~/.codex/skills/`. |
| `kit/` | Batch alternative: `audit_prep_v6.py` renders a run root into 24-set slices, `slice_auditor_prompt_v6.md` is the per-slice wrapper. See `kit/README.md`. |
| `install.sh` | Copies the skill into one or both skill directories. |

Both skill directories contain:

- `SKILL.md`: the procedure the reviewing agent follows.
- `rubric_v6.md`: the audit rubric for that variant, including T/A/H/S/P definitions and issue tags.
- `audit_set.py`: helper with subcommands `rubric`, `render`, `record`, `next`, `status`. Standard library only.

Both directories also contain `test_audit_set.py`. Keep both skill copies identical.
The batch wrapper uses the same rubric and verdict mapping.

## Skill v0.10.1

The skill audits construction quality without scoring behaviour-family
labels. Generic persona blocks with dialogue lines and an interchangeable tail
request are ineligible under the rubric. Task-specific roleplay with a concrete
conflict is not automatically excluded. A/H must not retain active harmful
permissions outside the neutralizing change; risky wording alone is not failure. Near-duplicate
removal is outside the skill and belongs to a separate collection process.
It accepts supplied files and pasted records, as well as HARLEY run roots.
Advice is read-only; recording requires an explicit request. The default response
is a short decision and per-role findings. Structured output is optional unless
recording.

New records use `harley_set_audit_verdict_v4`:

| Decision | Meaning |
|---|---|
| approve | Defining relations hold; minor revision needs are accepted and explained. |
| deny | A defining relation fails, the set is ineligible, or material evidence is insufficient. |

`decision` is the only outcome field; new records have no separate `verdict`.
Use `revision_needed` for a required repair and `evidence_gap` for unresolved
material evidence. A denial with an explained evidence gap may preserve unknown
H/S findings as null; it does not claim those findings are defects. Behaviour
labels remain optional provenance and are not scored.
See `codex/audit-contrast-set/rubric_v6.md` for the schema and acceptance gate.
Historical v1/v2/v3 records stay unchanged and are reported separately by `status`.
Freeze the skill version with predictions: the v4 record shape is unchanged, but
v0.10.1 accepts minor revisions. For human agreement, map approve + revise to
approve and reject to deny; report disputed/unsure separately from the resolved
agreement denominator. Keep all resolved primary decisions in the headline score,
including label-only or duplicate-related rejections, and explain those separately.
When explicitly requested, credit either prediction on disputed cases and label
that adjusted score clearly; keep the resolved confusion matrix separate and
exclude unsure. Do not claim duplicate detection was tested.
Review-app import compatibility remains unverified for v4; an older consumer
needs an explicit adapter, not a silent change to saved decisions.

The file reader supports JSON, JSONL, CSV, and TSV. Other readable attachments
require an appropriate reader. Select a collection explicitly when a JSON file
contains multiple collections:

```bash
python3 codex/audit-contrast-set/audit_set.py render --file input.json --collection records
python3 codex/audit-contrast-set/audit_set.py record SOURCE_ID --reviewer NAME --output verdict.json --verdict-json verdict-input.json
python3 -B -m unittest discover -s codex/audit-contrast-set -p 'test_*.py'
python3 -B -m unittest discover -s claude/audit-contrast-set -p 'test_*.py'
python3 -B -m unittest discover -s kit -p 'test_*.py'
```

File rendering preserves prompt text and omits verdicts, notes, and planner keys.
Standalone recording writes only to `--output`. The optional run adapter uses
`HARLEY_RUNS_DIR`, defaulting to `./wildchat_candidate_mining_2/runs` relative to
the working directory, or accepts an explicit `--run` path. The bundled rubric
takes precedence over older run rubrics.

## Install

```bash
git clone git@github.com:TyBrassington/harley-gold-set-helper.git
cd harley-gold-set-helper
./install.sh            # or: ./install.sh claude   /   ./install.sh codex
```

Then in Claude Code or Codex: `/audit-contrast-set wildchat_03c3f36c981b387b`, or ask
for "the next set in harley_msv1_forward_v1_wave4_v6".

## Paste mode (no lab access)

The review app's copy button emits one JSON record per set. Paste it into Claude
Code or Codex. The agent reads the bundled rubric and replies with Approve / Deny and per-role findings. Structured output is provided only when
requested. It does not run `render`, `record`, or `next`, and nothing is written.
The human enters the recommendation in the app. Revision needs and uncertainty
appear in the explanation, not as additional final decisions. Minor repairs accompany approval; material evidence gaps require denial.

## Where the data lives

The run adapter reads finished v6 run roots (`variants/final/approved_contrast_sets.jsonl`,
`excluded_contrast_sets.jsonl`, `human_verify.jsonl`) and writes requested verdicts to
`<run_root>/audit/verdicts/<candidate_id>.<reviewer>.json`. The default runs directory
is `./wildchat_candidate_mining_2/runs` relative to the current working directory.
Set `HARLEY_RUNS_DIR` or pass an explicit `--run` path for another location.
Only run roots whose `variants/final/manifest.json` declares `pipeline_version: v6`
are searched automatically.

`python audit_set.py status --run RUN --csv out.csv` flattens all verdicts for a run.
