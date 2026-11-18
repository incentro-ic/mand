# Mand.

Short, straight answers for your AI.

Mand is a Claude skill that makes answers brief, plain and direct: the answer first, only what was asked, usually in one or two sentences. It picks one option instead of listing five, and says so when your plan is a bad idea. One mode, no settings. Named after the Dutch "Mand." meme, a basket that was never for sale.

Site: https://mand.incentro.dev

## Install

**1. Claude Code plugin**

```
/plugin marketplace add incentro-ic/mand
/plugin install mand@mand
```

**2. Zip**

Download [mand.zip](https://github.com/incentro-ic/mand/releases/latest/download/mand.zip) and unzip it into `~/.claude/skills/`, or upload it as a skill in Claude.ai.

**3. Manual**

```
git clone https://github.com/incentro-ic/mand.git
cp -r mand/skills/mand ~/.claude/skills/mand
```

## Other agents

Mand follows the open [Agent Skills](https://agentskills.io) format, so the same `mand/` folder from the zip works in other agents that read `SKILL.md` files. Put it in that agent's skills folder:

| Agent | Folder |
|---|---|
| Claude Code | `~/.claude/skills/` |
| OpenAI Codex | `~/.codex/skills/` |
| GitHub Copilot | `~/.copilot/skills/` |
| Cursor | reads `~/.claude/skills/` and `~/.codex/skills/` |
| Gemini CLI | `.gemini/skills/` in your project |
| Most others | `.agents/skills/` in your project |

## Use

End a question with "Mand." to get a short answer to that question only. Send just "Mand." after a long answer to get the TL;DR of it. To use Mand for the rest of the conversation, type `/mand:mand` (plugin) or `/mand` (skill). Type it again to turn it off.

"Mand" inside a sentence, like "ik zoek een picknickmand", doesn't trigger it.

## The rules

1. Answer first. No intro, no restating the question.
2. Answer only what was asked, then stop. No history, extra tips or recap.
3. Plain words, short sentences. No jargon unless you used it first.
4. No filler, no stacked hedges, no asides in parentheses.
5. Prose by default. Lists only for scanning or comparing.
6. One example at most.
7. Short question, short answer: "Yes, since Python 3.7."
8. Never wrong for the sake of short. Caveats about safety, security, data, money and side effects stay.
9. Code stays complete and runnable.
10. Answers in your language.
11. Ask for more detail and you get it. Then it's short again.

Be direct:

12. Pick one. One recommendation, not a menu of options.
13. Say it when the plan is a bad idea. The better way first, never the risky one.
14. Fix wrong assumptions first. "Why is Python faster than C?" It isn't.
15. Commit. Yes, no, or how sure. "Nobody knows" when nobody does.
16. Blunt about the work, never about you. No sarcasm, no lecturing.
17. No made-up problems. A good plan gets a plain yes.

## Does it work?

Word counts from the [examples](skills/mand/examples.md), same model, with and without the skill:

| Question | Without | Mand. |
|---|---:|---:|
| Is Python's dict ordered? | 137 | 9 |
| Undo last git commit, keep changes | 113 | 24 |
| Palindrome function | 209 | 46 |
| React infinite re-render | 213 | 48 |
| What is a race condition? | 344 | 44 |
| Delete all Docker data | 315 | 39 |
| Excel cel vastzetten (Dutch) | 158 | 51 |

Being direct (examples 9 to 12). Without Mand, Claude often reaches the same advice, but it does the risky thing first or buries the advice under a full setup:

| Question | Without | Mand. |
|---|---|---|
| Turn off SSL verification in Python requests | 288 words, starts with `verify=False` | 108 words, starts with "Don't turn it off in production" and the proper fix |
| PostgreSQL or MongoDB for a web shop? | 355 words, picks PostgreSQL, then when to use MongoDB and tips | 33 words, picks PostgreSQL with one reason |
| node_modules in .gitignore, right? | 99 words, yes plus a checklist | 43 words, yes plus the two notes that matter |
| Kubernetes for a blog with 50 visitors (Dutch) | 449 words, says it's overkill, then gives the full cluster setup | 89 words, "Doe het niet", then static hosting |

## Releasing

If `skills/mand/SKILL.md` changed, run the evals first:

```
claude plugin eval . --trust-plugin --ablation none -j 6 --threshold 0.8
```

Exit code 0 means every case scored at least 0.8. Still check the report: any failed run in a false-trigger case (`f-*`) blocks the release. For a failing trigger case (`t-*`), look at why: one run going a few words over is noise, a wrong or missing caveat is not. See [evals/README.md](evals/README.md) for details. README or site changes don't need a run.

Then set `version` in `.claude-plugin/plugin.json`, commit, and push a matching tag (`v1.0.0` for `1.0.0`). The release workflow fails if they differ, otherwise it builds `mand.zip` and attaches it to a GitHub Release.

## Licence

MIT. See [LICENSE](LICENSE).
