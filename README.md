# soundshuman

[![AgentShield Security](https://merx.help/badge/A+.svg)](https://merx.help/shield)

Keep AI slop out of your prose and your repo.

This is a merger of the best parts of the current anti-slop tooling: one skill that rewrites text like a careful human editor, one rule pack that machines can enforce, and one zero-dependency CLI that audits whole repos and gates CI. The goal is not beating AI detectors. It is a workflow you can inspect: visible rules, explainable scores, reviewable diffs.

## What was taken from where

| Source | What it contributed |
|--------|---------------------|
| [blader/humanizer](https://github.com/blader/humanizer) | The rewrite skill's backbone: the Wikipedia-derived pattern catalog, voice calibration from writing samples, the no-fabrication rule, false-positive guardrails, the draft, audit, final loop, and the plugin packaging. |
| [hardikpandya/stop-slop](https://github.com/hardikpandya/stop-slop) | Structural anti-patterns the others miss (false agency, narrator-from-a-distance, binary contrasts, negative listing, lazy extremes, adverb pile), the pre-delivery checklist, and the lean skill-plus-references layout. |
| [brandonwise/humanizer](https://github.com/brandonwise/humanizer) | The tooling ideas: tiered vocabulary, statistical tells (burstiness, type-token ratio, trigram repetition), explainable 0-100 scoring, repo-wide scanning with CI gates, and a test suite for the scanner. |
| Aider | The editing workflow: rewrites land as git diffs on a clean tree, you review them like code, and undo is `git checkout`, not an apology. |
| ast-grep / OpenRewrite | Rules as data. Detection lives in a versioned JSON rule pack, not in code, so a house rule is a reviewable one-line diff. |
| Biome / Semgrep | Guardrails after the edit: a pre-commit hook and a CI job that catch what the session missed. |

Everything here is MIT licensed; upstream copyrights are preserved in [LICENSE](LICENSE).

## The pieces

[SKILL.md](SKILL.md) is the humanizer skill: 41 patterns across content, language, style, communication, filler, and rhetoric, plus statistical tells, voice calibration, and four invocation modes (pasted text, file, repo audit, embedded). It is plain markdown, so it runs in any agent harness that reads skills.

[rules/slop-rules.json](rules/slop-rules.json) is the machine-readable rule pack: three vocabulary tiers, phrase fixes, and regex detectors with weights and notes. [bin/sloplint.js](bin/sloplint.js) is the scanner that reads it. Zero dependencies, Node 18+.

The long-form material lives in [references/](references/): [vocabulary](references/vocabulary.md), [phrases](references/phrases.md), [structures](references/structures.md), a [style guide](references/style-guide.md), a [checklist](references/checklist.md), and [examples](references/examples.md). [docs/WORKFLOW.md](docs/WORKFLOW.md) explains how the pieces fit together, stage by stage.

## Install

As a skill, for any harness that reads markdown skills:

```bash
cp -r . /path/to/your/skills/soundshuman
```

As a Claude Code plugin:

```
/plugin marketplace add aashaexo/soundshuman
/plugin install soundshuman@soundshuman
```

The CLI needs nothing but Node:

```bash
node bin/sloplint.js --help
```

Or link it globally:

```bash
npm install -g .
```

## Use

Score any text (0 to 100, higher means more AI-flavored, under 25 is the target):

```bash
echo "This serves as a testament to our seamless approach." | sloplint score
```

See exactly why, with line numbers and the rule that fired:

```bash
sloplint analyze draft.md
```

Apply the safe mechanical fixes (filler phrases, em dashes, curly quotes):

```bash
sloplint fix draft.md -w
```

Audit a whole docs folder, worst first:

```bash
sloplint scan docs/
```

Gate CI:

```bash
sloplint scan docs/ --fail-above 50
```

Install the pre-commit hook:

```bash
scripts/install-git-hook.sh 50
```

For the full rewrite treatment, invoke the skill in your agent ("humanize this", "humanize docs/launch-post.md", "audit our docs for slop"). The skill drafts, audits its own draft, and delivers a final rewrite with no invented facts. Give it two paragraphs of your own writing first and it will match your voice instead of producing generic clean output.

## Always-on mode

Prevention beats cleanup. To make an agent write like this by default, add the short version to your system prompt or CLAUDE.md:

```
Write like a person, not a press release. Use "is" and "has", not "serves as"
or "boasts". Never use: delve, tapestry, vibrant, crucial, robust, seamless,
groundbreaking, pivotal, testament, leverage, empower, realm. No em dashes.
No "It's not just X, it's Y". No "Great question!" or "I hope this helps".
One qualifier per claim. Vary sentence length. End on a concrete fact, not
"the future looks bright". Never invent facts, names, numbers, or citations.
```

Then verify with `sloplint score` on the agent's output.

## Scoring, briefly

Pattern density per 100 words on a log curve (70%), blended with statistical uniformity: burstiness, type-token ratio, and trigram repetition (30%). Bonuses for breadth and category diversity. Short samples get a confidence haircut instead of a confident wrong answer. Every point is traceable to a finding you can read, argue with, or delete from the rule pack.

## Development

```bash
npm test
```

The tests pin the scorer's behavior: AI samples score high, human samples score low, code blocks are ignored, fixes are safe. If you add a rule, add a test.
