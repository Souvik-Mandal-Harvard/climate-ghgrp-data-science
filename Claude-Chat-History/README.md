# Provenance — raw Claude session transcripts

This folder ships the raw `.jsonl` session transcripts from the Claude / Cowork
sessions used to execute this project, in compliance with the take-home rubric
("a log of every LLM prompt used during the project").

Two complementary deliverables together provide the prompt history:

- **`deliverables/doc-03_prompt_log_HBS-assignment_SouvikMandal.md`** — a curated, human-readable log of the substantive prompts that shaped scope, methodology and deliverables. *Lead with this for a reviewer who wants a quick read.*
- **`deliverables/provenance/*.jsonl`** *(this folder)* — the raw API event stream from each Cowork session. *Use this for full reproducibility / replay-the-collaboration auditing.*

## What's in this folder

| File | Session | Period | Size | Records | Notes |
|---|---|---|---:|---:|---|
| `session_2026-05-22_current_429a7672.jsonl` | Day 2–Day 3 build + finalization | 2026-05-22 → 2026-05-25 | 11.2 MB | 3,087 | 763 user prompts, 1,305 assistant turns. Includes NB 03 polish, NB 06 build, NB 07 dashboard build, all final deliverables. |

## What's missing (and how to add it)

The Day 0 framing session (`local_1d2aa1be-a28b-467c-a4f1-353d5ebd106d`,
"Senior Data Scientist Take-Home Assignment", 1,983 transcript lines) lives in a
Claude application-private directory that the sandbox running this project
cannot reach. The file is on your laptop — to add it to this submission, run
the following from your shell (one line):

```bash
SRC="$HOME/Library/Application Support/Claude/local-agent-mode-sessions"
find "$SRC" -name "*.jsonl" -type f -exec cp -n {} \
  "$(pwd)/deliverables/provenance/" \;
```

That will sweep every Claude session `.jsonl` on your machine into this folder.
Rename each file with a descriptive prefix (date + session-title slug) before
committing to GitHub so the reviewer can scan the chronology at a glance.

Alternatively, just the Day 0 session:

```bash
find "$HOME/Library/Application Support/Claude/local-agent-mode-sessions" \
  -path "*local_1d2aa1be-a28b-467c-a4f1-353d5ebd106d*" -name "*.jsonl" \
  -exec cp {} "$(pwd)/deliverables/provenance/session_2026-05-18_day0_framing.jsonl" \;
```

## File format

Each `.jsonl` file is one JSON record per line, conforming to the Claude
Cowork session event schema. Common record types you'll see:

| `type` | Meaning |
|---|---|
| `user` | A user message (the prompts) |
| `assistant` | A model turn — either text, a tool call, or a tool result |
| `ai-title` | Cowork's auto-generated title for the session |
| `last-prompt` | A pointer to the most-recently-sent prompt (housekeeping) |
| `queue-operation` | Internal enqueue/dequeue events for streaming |
| `attachment` | A file the user uploaded |
| `system` | A system-level event (e.g. compaction, settings change) |

To extract just the user prompts from a `.jsonl`, for example:

```bash
python3 -c "
import json, sys
for ln in open(sys.argv[1]):
    r = json.loads(ln)
    if r.get('type') == 'user' and 'content' in r:
        c = r['content']
        if isinstance(c, str) and c.strip() and not c.startswith('<'):
            print('---'); print(c)
" deliverables/provenance/session_2026-05-22_current_429a7672.jsonl
```

## What the reviewer can do with this

1. **Replay the framing → execution arc.** Read prompts in chronological order;
   verify that scope decisions (problem statement, methodology choices, the
   NB 04 drop, citation corrections) are anchored to explicit human direction
   rather than emergent from the model.
2. **Audit the human-AI division of labor.** Every push-back, every test for
   sycophancy, every constraint (4-day budget, 250-word section limits,
   under-100-word limitations bullet) is visible verbatim in the user turns.
3. **Reproduce.** With the raw transcripts plus the code/data in the rest of
   the repo, any reviewer can re-run the entire pipeline (notebooks 00 → 07)
   and compare outputs against what's checked into `data/processed/`.

## Privacy / redaction note

These transcripts contain absolute filesystem paths that include
`/Users/souvikmandal/...`. They do **not** contain API keys (those are loaded
from `.env`, which is gitignored), and they do not contain personally
identifying information beyond the author's name and project paths. If you
prefer to publish a redacted version, the one-liner below sed-strips the
home directory:

```bash
sed 's|/Users/souvikmandal|/Users/REDACTED|g' \
  deliverables/provenance/session_*.jsonl \
  > deliverables/provenance/session_redacted.jsonl
```
