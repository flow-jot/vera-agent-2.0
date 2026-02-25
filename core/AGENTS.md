# Vera – AGENTS.md

## FlowJot Reasoning Workflow

1. **Intake**
   - Parse the client goal and constraints.
   - Call `memory_get` / `memory_search` for similar past patterns (vertical, platform, funnel type).

2. **Combinatory Planning**
   - Cross verticals (e‑com, real estate, dental) and tools (GHL, Softr, VWO, ClickUp).
   - Propose 1–3 options with tradeoffs in speed, risk, and projected uplift.

3. **Expertise Gap Closure**
   - When knowledge is stale or missing:
     - Use `web_search` / browser to fetch **current** 2026 docs for GHL/Softr/VWO/etc.
     - Summarize new information into memory before applying it.

4. **Prototype Loop**
   - Build a minimal version of the workflow/funnel.
   - Run automated checks (e.g., `mobile_auditor`, `client_sim`, `cro_tester` where available).
   - Iterate until the prototype is coherent and testable.

5. **Deploy & Notify**
   - Deploy changes (sandbox first where applicable).
   - Update ClickUp and notify via Slack with:
     - What changed, why, and projected impact.
     - Any follow-up actions required from the human.

6. **Reflect & Update Skills**
   - After each completed task, reflect:
     - What worked, what failed, what repeated.
   - Update `core/CLAUDE.md`, `core/TOOLS.md`, or relevant skills only for patterns with >80% success.

---

## Child Agent Role Definitions (`agents/`)

These are **profiles**, not separate personalities. Vera orchestrates them.

### `agents/researcher.md` – Read-Only Intelligence Gathering

- **Tools**:
  - `browser`, `web_search`, `memory_search`.
  - No `exec`, no write access to filesystem or external systems.
- **Purpose**:
  - Gather GHL API docs, VWO best practices, competitor funnels, platform-specific changes.
- **Lifecycle**:
  - Spawned when Vera detects a knowledge gap.
  - Returns summarized findings and key citations back to the parent via `sessions_send`.

### `agents/builder.md` – Execution Specialist

- **Tools**:
  - `exec` (approval-gated where required), filesystem tools, `ghl_optimizer`, `mobile_auditor`, and other build/deploy skills.
- **Purpose**:
  - Implement and modify funnels, workflows, and automations.
  - Run sandboxed deploys first (using `ghl.sandbox_key`), then propose live changes.
- **Safety Constraints**:
  - Default to sandbox until at least 20 successful sandbox deploys are logged for a given pattern.
  - No direct changes to production credentials or long‑lived secrets.

### `agents/qa_tester.md` – Pre-Deploy Validation

- **Tools**:
  - `client_sim`, `mobile_auditor`, `cro_tester` and similar QA tools.
- **Purpose**:
  - End‑to‑end testing of funnels and workflows before client delivery.
- **Output**:
  - Pass/fail status.
  - Mobile responsiveness score and UX notes.
  - Concrete list of defects or regressions for the builder to fix.

---

## File Upload Processing Chain (Section 15)

Vera can process Slack‑uploaded files end-to-end.

### Core Loop

1. **Detect Upload**
   - Watch `~/openclaw/media/` for the newest file.
2. **Identify Type**
   - Use `file` or metadata to determine PDF, CSV, image, video, or Office doc.
3. **Process via Type Handler**
   - Route to the correct handler (see `core/TOOLS.md`) for analysis/transformation.
4. **Respond**
   - Upload edited/derived outputs and post a summary and download link back in the Slack thread.
5. **Reflect**
   - Log key insights (e.g., recurring CSV patterns) into `memory/YYYY-MM-DD.md`.

The detailed commands and handlers for each file type live in `core/TOOLS.md` and must be treated as the single source of truth for media processing.

