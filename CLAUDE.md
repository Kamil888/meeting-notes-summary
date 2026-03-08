# Meeting Notes Summary — Project Context

## What this project is

A Claude skill (`meeting-notes-summary`) that transforms meeting transcripts into structured Word (.docx) documents with a full executive summary.

## Skill location

- **Skill definition**: `C:\Users\Kamil\.claude\skills\meeting-notes-summary\SKILL.md`
- **Docx script (reference)**: `C:\Users\Kamil\.claude\skills\meeting-notes-summary\scripts\create_docx.py`
- **Eval test cases**: `C:\Users\Kamil\.claude\skills\meeting-notes-summary\evals\evals.json`
- **Eval workspace**: `C:\Users\Kamil\.claude\skills\meeting-notes-summary-workspace\`

## What the skill does

**Input**: A meeting transcript as plain text or JSON (e.g., Teams/Zoom export with `speaker`/`text` fields).

**Output**: A `.docx` file named `meeting-notes-YYYY-MM-DD.docx` with these sections:
1. **Attendees** — everyone present, plus anyone mentioned but not on the call (marked "not present")
2. **Meeting Purpose** — one or two sentence summary
3. **Key Decisions** — only explicitly decided things, not discussions
4. **Discussion Summary** — concise narrative for someone who wasn't there
5. **Outcomes** — what changed as a result of the meeting
6. **Action Items** — flat list with owner, action description, and deadline (only if stated in transcript)

## Key design decisions

- **Deadlines**: Only included when explicitly mentioned in the transcript. Never inferred.
- **Non-attendees**: People mentioned but not on the call still appear in the attendee list as "(not present)" and can own action items.
- **Docx generation**: The skill instructs Claude to write and run the docx code inline (not call a bundled script), to avoid Bash permission friction in restricted contexts.
- **Input flexibility**: Works with both raw text transcripts and structured JSON formats.

## Iteration history

### Iteration 1 (2026-03-08)
- Built initial skill and ran 3 evals (with skill vs. without skill)
- **Results**: With skill 100% pass rate, without skill 61% pass rate (+39% delta)
- **Issue found**: Skill originally pointed to a bundled script (`scripts/create_docx.py`) via Bash — this caused friction in subagent contexts where Bash wasn't pre-approved
- **Fix applied**: Skill now tells Claude to write and run the docx code inline

## Dependencies

- `python-docx` — install with `pip install python-docx`

## GitHub

Repository: https://github.com/Kamill888/meeting-notes-summary
