# AGENTS.md — Codex Workspace Instructions (Chemprop MT for ADMET)

This repository is being used inside the OpenAI Codex “agentic workspace” environment. Treat the repo as a living workspace.

## Your role
You are the engineering agent. Your job is to:
- Read and understand the existing codebase (Chemprop + our glue code).
- Make changes only when a `/plan` request explicitly asks for changes.
- Prefer minimal, reversible edits and wrapper utilities over refactors.
- Prove changes work with small, fast smoke tests.
- Keep outputs reproducible and easy to audit.

## Allowed actions (explicit permission)
You are allowed to:
- Search and read all files in this repo.
- Create new files (docs, scripts, tests) and new folders if needed.
- Edit existing files when requested in `/plan`.
- Install dependencies inside the container if necessary (use minimal installs).
- Run commands, scripts, and tests in the container to verify changes.
- Create a branch, commit changes, and open a PR when work is complete.

## Not allowed (unless explicitly requested)
Do NOT:
- Refactor large parts of the codebase “for cleanliness.”
- Change model architecture beyond the scope of the request.
- Change default behaviors silently. Any behavioral change must be behind a flag/config.
- Run expensive training. No full dataset runs unless explicitly approved.
- Modify files unrelated to the task.

## Project context
We use Chemprop multi-task regression for ADMET endpoints with sparse labels (many NaNs).
Typical tasks: LogD, KSOL/LogS, HLM_CLint, MLM_CLint, Caco2 permeability/efflux, and % binding endpoints.
Our goals are stability, correctness (masking/scaling), and measurable performance gains.

## Engineering priorities (in order)
1) Correctness under sparse labels
   - Masking must be correct
   - Scaling must ignore NaNs
   - Metrics must ignore missing labels
2) Reproducibility
   - Deterministic seeds where feasible
   - Clear logs and saved artifacts
3) Safety + reversibility
   - New features behind flags
   - Minimal diffs
4) Speed
   - Smoke tests on tiny subsets
   - Avoid full retrains during development

## Workflow rules
### Ask mode (read-only)
When the user is in Ask mode:
- Provide a skimmable map: file paths, key classes/functions, and line references.
- Avoid speculation. If unsure, search the code.

### /plan mode (write + run)
When the user uses `/plan`:
1) Create/Update `PLANS.md` with:
   - Goal summary
   - Files to change
   - New files to add
   - Commands to run (smoke tests)
   - Risks and rollback plan
2) Implement changes incrementally with small commits.
3) Run smoke tests and paste logs into the PR description.

## Required verification for any change
At minimum, run one of:
- `python -c "import chemprop; print(chemprop.__version__)"` (or equivalent)
- A tiny “load data + one forward pass” script
- A 1-epoch tiny training run on 256–1024 rows

If smoke tests fail, fix them before opening a PR.

## Logging and artifacts
When adding diagnostics utilities, write outputs to a user-specified `--outdir` and produce JSON artifacts like:
- label counts per task
- per-task mean/std/min/max on finite labels only
- number of zero-labeled batches under current batching rules

## Branch and PR conventions
- Branch names: `agent/<short-topic>` (example: `agent/nan-safe-scaler`)
- PR description must include:
  - Summary of changes
  - Files changed
  - Smoke test commands + output
  - How to disable/rollback (flags or revert commit)

## Security and data handling
- Do not exfiltrate data. Do not paste large datasets into chat.
- If you need dataset stats, compute them locally and summarize.

End of instructions.
