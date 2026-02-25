# Vera – TOOLS.md

## FlowJot Client Intake Chain (Section 2.6)

- **True intake path**:
  - Softr client portal → Airtable logging → ClickUp as the execution queue.
- **Morning brief**:
  - Must query ClickUp (via the `clickup` skill) for tasks, not generic Notion/Sheets polling.
- **Execution pattern**:
  - Dequeue from ClickUp → execute task → update status and add notes/links back to the ClickUp task.

---

## Skill Vetting Workflow (Section 16.2)

Vera must **never** blindly install skills from ClawHub or other sources.

### Step 1 – Discovery

- Encounter skill via `web_search`, GitHub, or ClawHub listing.
- Capture its repository and version.

### Step 2 – Self-Diagnosis (Code Review)

Before installation:

1. Fetch the skill’s source (GitHub raw or ClawHub API).
2. Scan for red flags:
   - Outbound network calls to unvetted domains (`curl`, `wget`, `fetch`, arbitrary webhooks).
   - Credential exfiltration: reading `~/.ssh`, environment dumps, or broad filesystem reads.
   - Obfuscated commands (`base64`, `eval`, large inline heredocs with shell).
   - Writes outside the intended workspace (e.g., `/etc`, `/tmp` persistence, home directory).
   - Hard-coded Discord/Slack/webhook URLs or suspicious analytics endpoints.

3. Produce a safety report:
   - **SAFE** – No red flags; functionality clear.
   - **UNSAFE** – Clear exfiltration or dangerous patterns.
   - **REVIEW REQUIRED** – Ambiguous behaviours that need human review.

### Step 3 – Approval Gate

- **SAFE + useful** → Propose install via TUI approval, summarizing behaviour and permissions.
- **UNSAFE** → Recommend rejection and suggest safer alternatives.
- **REVIEW REQUIRED** → Present concerns and pause until a human audits the code.

### Step 4 – Clone & Adapt When Needed

If functionality is valuable but code is questionable:

- Use `skill-creator` to generate a **clean, local** implementation based solely on the concept.
- Treat third‑party code as reference, not something to run directly.
- Prefer owning the code and security surface for critical workflows.

---

## File Type Handlers (Section 15.3)

These are the default processing strategies for uploaded files in `~/openclaw/media/`.

| Type   | Commands / Flow                                                                                          | Output                                           |
|--------|----------------------------------------------------------------------------------------------------------|--------------------------------------------------|
| **PDF**   | `pdftotext input.pdf - \| summarize`                                                                   | Markdown summary, key insights into memory       |
| **CSV**   | `csvkit` or `python3` (pandas) to pivot/filter → write `out.csv`                                      | Cleaned CSV + stats table for Slack              |
| **Image** | `tesseract img.png -` for OCR or `exiftool img.jpg` for metadata                                      | Extracted text/metadata + optional annotations   |
| **Video** | `ffmpeg -i vid.mp4 -vf \"scale=640:360\" thumb.jpg` and frame extraction, optionally via `video-frames` | Thumbnails, frame snapshots, or transcripts      |
| **DOCX/XLSX** | `pandoc doc.docx -o doc.md` or `xlsx2csv sheet.xlsx`                                              | Markdown/CSV representation for editing/analyis  |

All commands that go through `exec` remain approval-gated, and processing must respect the 50MB `mediaMaxMb` cap and media quarantine/cleanup rules.

