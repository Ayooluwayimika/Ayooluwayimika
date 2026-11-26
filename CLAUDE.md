# CLAUDE.md

## What this repo is

A **public** GitHub repo that holds a vendored copy of the LinkedIn agent
skills (`li-*`) from `postzen-dev/linkedin-agent-skill`, under
`.claude/skills/` (MIT, see `.claude/skills/LICENSE-linkedin-agent-skill`).
They live here so every Claude Code session on this repo loads them.
There is no app, no package manager and no CI.

- `li-human/humanize.py`, `li-human/detect.py`: the only code. Python 3,
  standard library only. Both read `li-human/slop.json`.
- `li-post/hooks.json`, `li-profile/rubric.json`, `li-human/slop.json`: data
  files the skills read.
- Every other skill is a `SKILL.md` prompt.

## Commands

There is no build step and no test suite. To check a change, run from the
repo root:

```bash
# Run the humanizer and detector on a sample (put scratch files outside the repo)
python3 .claude/skills/li-human/humanize.py draft.txt --report
python3 .claude/skills/li-human/detect.py draft.txt
python3 .claude/skills/li-human/detect.py before.txt after.txt   # compare
python3 .claude/skills/li-human/detect.py draft.txt --json       # exits non-zero unless PASS

# Check the JSON data files still parse
for f in .claude/skills/*/*.json; do python3 -m json.tool "$f" >/dev/null || echo "BAD: $f"; done

# Byte-compile the scripts
python3 -m py_compile .claude/skills/li-human/*.py
```

After editing `humanize.py`, `detect.py` or `slop.json`, run both scripts on a
sample that has an em dash, a zero-width space, a URL, a slop word and a line
break. Confirm the URL, the line breaks and the indentation come through
unchanged. A past code review fixed bugs in exactly those areas (see commit
`f719610`).

## Code style

- Python: standard library only, no third-party dependencies. Use 4-space
  indents, keep the `#!/usr/bin/env python3` shebang and the usage docstring
  at the top, and use argparse for the CLI. Paths go through
  `os.path.dirname(os.path.abspath(__file__))` so the scripts work from any
  directory.
- Skill docs: YAML frontmatter with `name` and `description`, then plain
  Markdown. Commands inside a `SKILL.md` use repo-relative paths
  (`python3 .claude/skills/li-human/...`).
- Keep `slop.json` valid JSON, and keep each entry with its replacement in
  the right category. If the user has a word they always use, remove it from
  the lexicon instead of working around it.
- Don't commit `__pycache__/` or `*.pyc` (they're in `.gitignore`).

## Working with the skills

- Before you show the user any LinkedIn post, comment, reply or DM, run it
  through `li-human` (`humanize.py`, then `detect.py`).
- Never publish, schedule or queue anything on LinkedIn (`li-publish`, the
  PostZen tools) without the user's explicit yes on the final text and time.
  Every time you show the user includes a timezone.
- This copy has no `.mcp.json`. PostZen works only when the session has the
  PostZen connector attached.
- If you change a vendored skill, say so in the commit message so it's clear
  the copy has moved away from upstream.

## Voice, plan and log files (cloud sessions)

The skills keep state in `~/.linkedin-agent/` (`voice.md`, `plan.md`,
`log.md`). In a Claude Code cloud session the home directory is wiped when
the container is reclaimed, so those files are gone next session and the
skills will ask for past posts and themes all over again.

This repo is public, so don't commit the voice profile, target-people list or
post log here without asking the user first. If the user fills in the
profile section below, use it to seed `~/.linkedin-agent/voice.md` at the start
of a session instead of asking again.

## User profile (filled in by the user)

<!-- Fill these in once so skills don't ask every session. -->
- What I sell, and to whom:
- Themes I want to be known for:
- Audience timezone (IANA name, e.g. Africa/Lagos):
- Words or phrases I always use (keep them even if they're in slop.json):
- Words or phrases I never use:

## Git

- Work on the branch the session names. Don't push to `main` directly.
- Don't open pull requests unless asked.
