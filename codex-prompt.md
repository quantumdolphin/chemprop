You are Codex (junior dev). We need to audit the EXACT Chemprop version installed in this environment and enumerate ALL supported CLI flags and relevant capabilities (loss functions, task weighting, config/toml/yaml support, multitask behavior, metrics, scaling, uncertainty, etc.).

Goal: produce a reproducible “chemprop_capabilities_report” folder containing:
1) chemprop_version.txt
2) chemprop_path.txt
3) chemprop_help.txt
4) chemprop_train_help.txt
5) chemprop_predict_help.txt
6) chemprop_fingerprint_help.txt (if command exists)
7) chemprop_args_grep.txt (grep of likely flags: loss, huber, weights, task_weights, target_weights, metric, scaler, normalize, uncertainty, ensemble, config, toml, yaml)
8) python_introspection.txt (python-side inspection of chemprop modules/classes to infer whether task weighting and huber exist)
9) pip_freeze_chemprop.txt (pip show + pinned deps)
10) a short summary.md that lists:
   - Chemprop version
   - Which subcommands exist
   - Whether config files are supported and in what format
   - Whether weighted loss / task weights exist and how to enable
   - What loss functions are available and how to select them

Constraints:
- Do NOT assume any flags.
- Prefer “ground truth” from `chemprop ... --help` and from installed python package inspection.
- Save all outputs to disk and also print a short console summary at the end.

Implementation instructions:
- Run in bash from repo root (or any folder), using the active conda env where `chemprop` runs.
- Use set -euo pipefail.
- Make the script robust: if a subcommand doesn’t exist, capture that fact instead of failing.
- Use both CLI help and Python introspection.

Now do it:

1) Create a script: tools/inspect_chemprop_install.sh
2) Execute it.
3) Show me the final console summary and the tree of the output folder.

Script spec:

- Output folder: chemprop_capabilities_report/YYYYMMDD_HHMMSS/
- Capture stdout+stderr for each command into files.
- Commands to run (each should not crash the whole script; record failure):
    which chemprop
    chemprop --version (and/or chemprop -V)
    chemprop --help
    chemprop train --help
    chemprop predict --help
    chemprop fingerprint --help (if exists)
    chemprop hpopt --help (if exists)
    pip show chemprop
    python -c "import chemprop; print(chemprop.__version__); print(chemprop.__file__)"
    python -c "import inspect, chemprop; import chemprop.cli; print(dir(chemprop.cli))"
    python - <<'PY'
        import pkgutil, chemprop, re, inspect, sys
        from pathlib import Path
        import chemprop as cp

        root = Path(cp.__file__).resolve().parent
        print("chemprop_root:", root)

        # Search source for common strings that indicate support for weighting/losses/config
        needles = [
            "huber", "Huber", "loss", "loss_function", "task_weight", "task_weights",
            "target_weights", "weighting", "weights", "inv_sqrt", "uncertainty",
            "config", ".toml", "toml", "yaml", "OmegaConf", "hydra",
            "StandardScaler", "scaler", "normalize", "mask", "NaN"
        ]

        # Walk .py files and count hits
        hits = {n: [] for n in needles}
        for p in root.rglob("*.py"):
            try:
                txt = p.read_text(errors="ignore")
            except Exception:
                continue
            for n in needles:
                if n in txt:
                    hits[n].append(str(p.relative_to(root)))

        for n in needles:
            print("\n==", n, "==")
            print("count_files:", len(set(hits[n])))
            for f in sorted(set(hits[n]))[:50]:
                print(" ", f)

        # Try to locate argparse definitions of train/predict
        try:
            import chemprop.cli.train as t
            print("\ntrain_module:", t.__file__)
        except Exception as e:
            print("\ntrain_module_import_error:", repr(e))

        try:
            import chemprop.cli.predict as p
            print("predict_module:", p.__file__)
        except Exception as e:
            print("predict_module_import_error:", repr(e))
    PY

- After running, generate summary.md by parsing:
    - version
    - list of subcommands (from chemprop --help)
    - any weighting/loss/config evidence (from grep + python search)

Finally:
- Print:
    - Chemprop version
    - Subcommands
    - Whether you found any “huber” support
    - Whether you found any “task weight/weighting” support
    - Whether you found any “toml/yaml/config” support
and show:
    tree chemprop_capabilities_report/<latest_timestamp> -L 2

Proceed.
