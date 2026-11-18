# Mand evals

Run from the repo root:

```
claude plugin eval . --trust-plugin -j 6
```

Each case runs with and without the skill, so the report shows what Mand adds (`Δ`).

- `t-*`: cases that should trigger Mand: "Mand." at the end, "Mand." alone, and `/mand:mand` on and off. Graded on length, correctness and kept caveats.
- `f-*`: false triggers. "mand" as an ordinary Dutch word, "korter", "be brief", "TL;DR" for a text. Mand must not fire and the answer must be normal.

Use `--tag trigger` or `--tag false-trigger` to run one group, and `--ablation none` to skip the baseline arm.

## History files

Multi-turn cases continue from a `history.jsonl` in the case folder. They were recorded with `claude -p` and stripped to the user and assistant messages. The `/mand:mand` ones contain a copy of SKILL.md as it was when recorded, so re-record them after big changes to the skill.

In those cases the baseline arm also sees that copy, so their `Δ` says little.
