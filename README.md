# HARLEY gold-set helper

Tooling for independently auditing HARLEY v6 WildChat behaviour-contrast sets
(five roles: misaligned target T, aligned control A, hard negative H, style
control S, paraphrase target P). Reviewers can inspect run roots or supplied
records and optionally save verdicts. Codex v0.7.0 uses a new verdict schema;
see the compatibility notes below before importing those records into a review app.

## Layout

| Path | What |
|---|---|
| `claude/audit-contrast-set/` | Skill for Claude Code. Install into `~/.claude/skills/`. |
| `codex/audit-contrast-set/` | Codex skill v0.7.0: portable file/run audits, construction-only judgments, and v3 verdicts. Install into `~/.codex/skills/`. |
| `kit/` | Batch alternative: `audit_prep_v6.py` renders a run root into 24-set slices, `slice_auditor_prompt_v6.md` is the per-slice wrapper. See `kit/README.md`. |
| `install.sh` | Copies the skill into one or both skill directories. |

Both skill directories contain:

- `SKILL.md`: the procedure the reviewing agent follows.
- `rubric_v6.md`: the audit rubric for that variant, including T/A/H/S/P definitions and issue tags.
- `audit_set.py`: helper with subcommands `rubric`, `render`, `record`, `next`, `status`. Standard library only.

The Codex directory also contains `test_audit_set.py`. The Claude skill and batch
kit retain the earlier audit protocol; they are not synchronized with Codex v0.7.0.

## Codex v0.7.0

The Codex skill audits construction quality without scoring behaviour-family
labels. It accepts supplied files and pasted records, as well as HARLEY run roots.
Advice is read-only; recording requires an explicit request. The default response
is a short decision and per-role findings. Structured output is optional unless
recording.

New records use `harley_set_audit_verdict_v3`:

| Verdict | Decision | Meaning |
|---|---|---|
| PASS | approve | Ready as-is. |
| MINOR | revise | A concrete repair is required; include `revision_needed`. |
| FAIL | reject | A defining relation fails. |
| HOLD | hold | Material evidence is unresolved; include `hold_reason`. |

Unknown H/S findings may be null for HOLD. Behaviour-label judgments and correction
tags are retired; `family` is optional source provenance. See the bundled
`codex/audit-contrast-set/rubric_v6.md` for the full schema and acceptance gate.
Historical v1/v2 records remain readable by `status` without being rewritten.
Consumers expecting `gold`/`reject` need an explicit adapter; do not silently
count `revise` or `hold` as approval. Compatibility with the review app's importer
has not been verified for v3.

The file reader supports JSON, JSONL, CSV, and TSV. Other readable attachments
require an appropriate reader. Select a collection explicitly when a JSON file
contains multiple collections:

```bash
python3 codex/audit-contrast-set/audit_set.py render --file input.json --collection records
python3 codex/audit-contrast-set/audit_set.py record SOURCE_ID --reviewer NAME --output verdict.json --verdict-json verdict-input.json
python3 -B -m unittest discover -s codex/audit-contrast-set -p 'test_*.py'
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

The following describes the legacy Claude workflow. For Codex, use the decisions
and output format described above and in its bundled skill.

The review app's copy button emits one JSON record per set (`candidate_id`,
`prompts.T/A/H/S/P`, `H_plan`, `S_declared_realised_dimensions`, ...). A human
reviewer pastes it into Claude Code while verifying the set by hand.
With the skill installed, the agent reads the bundled `rubric_v6.md`, judges
from the pasted prompts, and replies with the verdict JSON, per-role findings,
and a final `DECISION:` line (gold, gold with a family relabel, or reject). It does not run `render`, `record`, or `next`, and nothing is written.
The human enters the decision in the app.

## Where the data lives

The run adapter reads finished v6 run roots (`variants/final/approved_contrast_sets.jsonl`,
`excluded_contrast_sets.jsonl`, `human_verify.jsonl`) and writes verdicts to
`<run_root>/audit/verdicts/<candidate_id>.<reviewer>.json`. The default runs
directory for the legacy Claude helper is the lab share checkout:

```
/local-scratch/localhome/wgb/behavior-latent-library/wildchat_candidate_mining_2/runs
```

Set `HARLEY_RUNS_DIR` if the run roots are somewhere else. Only run roots whose
`variants/final/manifest.json` declares `pipeline_version: v6` are searched.

## Legacy Claude verdict file

Schema `harley_set_audit_verdict_v1`. Required fields: `candidate_id`, `verdict`
(`PASS|MINOR|FAIL`), `tags` (closed list in the rubric; empty iff PASS), `reason`,
`h_decisive_fact_ok`, `family_grounded`, `s_dimensions_realised`,
`s_dimensions_missing`, `s_topic_disjoint`, `s_cue_free`, `s_same_speaker`,
`decision` (`gold` for PASS/MINOR, `reject` for FAIL).
Optional: `notable`, `family`, `anchor_role`, `contract_source`,
`human_verify_verdict`, `human_verify_notes`, `family_override` (required with
the FAMILY_BETTER_FIT_ELSEWHERE tag: the better-fitting family; the set stays
gold and the reviewer changes the behaviour in the app).

`python audit_set.py status --run RUN --csv out.csv` flattens all verdicts for a run.
