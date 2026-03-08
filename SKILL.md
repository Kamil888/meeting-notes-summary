---
name: meeting-notes-summary
description: Summarizes meeting transcripts into a structured Word (.docx) document with a full executive summary. Use this skill whenever the user provides a meeting transcript (plain text or JSON) and wants it summarized, organized, or turned into meeting notes. Trigger on phrases like "summarize this meeting", "create meeting notes from this transcript", "extract action items", "turn this transcript into a document", "make a summary of this call", or any time a transcript is shared alongside a request for organization, summary, or documentation. Even if the user just pastes a transcript and says "summarize this" — use this skill.
---

# Meeting Notes Summarizer

Transform a meeting transcript into a clean, structured Word document (.docx) with a full executive summary.

## Input formats

The transcript can be provided as:
- **Plain text**: Raw transcript, ideally with speaker labels (`John: "..."`) but the skill handles unlabeled text too
- **JSON**: Structured format (e.g., from Teams, Zoom exports) with fields like `speaker`, `text`, `timestamp`

If the user doesn't specify, infer the format from what they paste or provide.

## Document structure

Generate a `.docx` file named `meeting-notes-<YYYY-MM-DD>.docx` (use the meeting date if found in the transcript, otherwise today's date).

The document should contain these sections in order:

### 1. Attendees
List every person who spoke or was explicitly mentioned in the transcript. For people mentioned but not on the call, note them as "(not present)".

### 2. Meeting Purpose
One to two sentences explaining why this meeting was held. Infer from context if not stated explicitly.

### 3. Key Decisions
Bullet list of decisions that were explicitly made during the meeting. Only include things that were clearly decided — not things that were discussed or floated as ideas.

### 4. Discussion Summary
A concise narrative of the main topics discussed, debates, and context. Capture the substance without replaying the transcript. Aim for clarity over completeness — this is an exec summary.

### 5. Outcomes
What changed or was concluded as a result of this meeting? What is the state of things now that this meeting has happened?

### 6. Action Items
A single flat list of all action items, so everyone can see what others need to do. Each item includes:
- **Owner**: The person responsible for this action
- **Action**: A clear, specific description of what needs to be done
- **Deadline**: Only include if explicitly mentioned in the transcript. Do not infer, guess, or add a deadline that wasn't stated. If no deadline was mentioned, omit the field entirely.

Anyone mentioned in the transcript can be an owner — even someone not on the call, if they were assigned work.

## How to proceed

1. Read the transcript carefully. Identify all speakers and mentioned parties.
2. Extract all structured information for each section above.
3. Format the data as JSON matching this schema:

```json
{
  "meeting_date": "YYYY-MM-DD",
  "attendees": [
    {"name": "Jane Smith", "present": true},
    {"name": "Bob Jones", "present": false}
  ],
  "meeting_purpose": "...",
  "key_decisions": ["...", "..."],
  "discussion_summary": "...",
  "outcomes": ["...", "..."],
  "action_items": [
    {
      "owner": "Jane Smith",
      "action": "Send updated proposal to the client",
      "deadline": "by Friday March 14"
    },
    {
      "owner": "Bob Jones",
      "action": "Review the Q1 budget figures"
    }
  ]
}
```

4. Write and run a short Python script to generate the Word document. Use `pip install python-docx` first if needed. Here is the script to use — fill in the `DATA` dict with the extracted meeting data:

```python
import json
from docx import Document
from docx.shared import Pt
from docx.enum.text import WD_ALIGN_PARAGRAPH
from pathlib import Path

DATA = { ... }  # paste your extracted JSON here

def add_heading(doc, text, level=1):
    h = doc.add_heading(text, level=level)
    h.alignment = WD_ALIGN_PARAGRAPH.LEFT
    return h

def add_bullet(doc, text):
    doc.add_paragraph(text, style="List Bullet")

doc = Document()
title = doc.add_heading(f"Meeting Notes — {DATA.get('meeting_date', '')}", level=0)
title.alignment = WD_ALIGN_PARAGRAPH.CENTER
doc.add_paragraph()

add_heading(doc, "Attendees")
for p in DATA.get("attendees", []):
    label = p["name"] if p.get("present", True) else f"{p['name']} (not present)"
    add_bullet(doc, label)
doc.add_paragraph()

add_heading(doc, "Meeting Purpose")
doc.add_paragraph(DATA.get("meeting_purpose", ""))
doc.add_paragraph()

add_heading(doc, "Key Decisions")
decisions = DATA.get("key_decisions", [])
if decisions:
    for d in decisions: add_bullet(doc, d)
else:
    doc.add_paragraph("No explicit decisions recorded.")
doc.add_paragraph()

add_heading(doc, "Discussion Summary")
doc.add_paragraph(DATA.get("discussion_summary", ""))
doc.add_paragraph()

add_heading(doc, "Outcomes")
for o in DATA.get("outcomes", []): add_bullet(doc, o)
doc.add_paragraph()

add_heading(doc, "Action Items")
for item in DATA.get("action_items", []):
    para = doc.add_paragraph(style="List Bullet")
    run = para.add_run(f"[{item['owner']}] ")
    run.bold = True
    para.add_run(item["action"])
    if item.get("deadline"):
        para.add_run(f" — by: {item['deadline']}").italic = True

output = f"meeting-notes-{DATA.get('meeting_date', 'unknown')}.docx"
doc.save(output)
print(f"Saved: {Path(output).resolve()}")
```

Run this script from the user's working directory so the file is saved somewhere convenient.

5. Tell the user the file has been saved and its full path.

## Tips

- If the transcript has no speaker labels, do your best to attribute statements from context clues
- Be conservative with key decisions — if it was discussed but not settled, it belongs in Discussion Summary, not Key Decisions
- Keep the Discussion Summary readable for someone who wasn't on the call
