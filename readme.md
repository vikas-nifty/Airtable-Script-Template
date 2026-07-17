# Airtable Scripting Workspace

A reusable, evidence-gated workspace for designing, building, testing, and
reviewing Airtable scripts.

## Repository contract

| File | Owner | Purpose |
| --- | --- | --- |
| `context/requirements.md` | User only | Source requirements and acceptance criteria |
| `context/baseSchema.md` | User refreshes | Current target-base schema captured from Airtable |
| `context/inputConfig.md` | User refreshes | Current script inputs and masked secret previews |
| `output/plan.md` | Codex | Required interpretation, decisions, dependencies, risks, and test matrix |
| `output/script.js` | Codex | Single completed script |
| `output/script1.js`, etc. | Codex | Separate Automation script actions |
| `methods/prompt.md` | Immutable template standard | High-level workflow and Airtable constraints |
| `methods/codingStyle.md` | Template standard | Production Airtable JavaScript standards |
| `AGENTS.md` | Template standard | Operating instructions for Codex |

Only the user may modify `context/requirements.md`. If requirements are unclear
or contradictory, Codex records the ambiguity in `output/plan.md` and asks the
user to resolve it.

## Workflow

1. The user creates or updates `context/requirements.md`.
2. The user runs the extractor in `context/baseSchema.md` in the target Airtable
   base and replaces the previous output.
3. Codex verifies the schema and creates `output/plan.md`, covering:
   - execution type and trigger;
   - record-selection strategy;
   - dependencies and field mapping;
   - reads, writes, APIs, secrets, and handoffs;
   - ownership, idempotency, duplicate detection, and concurrency;
   - failure, retry, checkpoint, and timeout behaviour;
   - acceptance tests traced to the requirements.
4. The user configures the Airtable script action and refreshes
   `context/inputConfig.md`.
5. Codex verifies every required input against the schema and plan.
6. Codex writes the complete script.
7. The user tests it in Airtable and supplies console log results in chat.
8. Codex marks only demonstrated behaviour as verified and makes the smallest
   evidence-backed revisions.

If the schema or dependencies change, repeat the relevant capture and
verification steps before changing code.

## Core rules

- Never hard-code Airtable table, view, or field names or IDs.
- Keep one input configuration object and avoid duplicate field declarations;
  resolve only Airtable objects needed for method calls.
- Never print or store an unmasked secret. Only the input-capture helper may log
  the documented masked preview.
- Do not assume an Automation supplies a triggering record ID.
- Define record selection explicitly using a configured record ID, view, or
  eligibility/checkpoint fields.
- Define source-of-truth and create/update/delete ownership before coding.
- Detect multiple Airtable records claiming the same external ID or business key.
- Treat ambiguous ownership as a visible failure before external mutation.
- Make mutating scripts safe to rerun after partial completion.
- Query only required fields and respect Airtable runtime and mutation limits.
- Treat each Automation script action as an isolated runtime.

## Starting a project

1. Copy this template.
2. Have the user replace `context/requirements.md`.
3. Refresh `context/baseSchema.md` in the target base.
4. Ask Codex to interpret the requirements and create `output/plan.md`.
5. Continue through the gated workflow.

Never reuse schema output, input mappings, IDs, secrets, or assumptions from a
previous base or project.
