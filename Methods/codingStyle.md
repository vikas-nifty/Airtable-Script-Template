# Airtable JavaScript Coding Style

Write compact production code: correct first, efficient where it matters, and readable in one pass.

## Think and design

- Define the outcome, invariants, side effects, scale, rerun behaviour, and realistic failure modes before coding.
- Trace the flow: input, validation, reads, transformation, writes, output.
- Choose the simplest algorithm and data structures that fit the expected scale.
- Validate script-wide requirements before expensive work or mutations.
- Prefer one deliberate pass over records when it remains clear.
- Split an Automation into focused scripts when that improves clarity, reliability, or runtime management.
- Treat each script as isolated; pass values explicitly or persist them in Airtable.

## Keep code direct

- Make the main flow readable from top to bottom.
- Keep short one-off work inline.
- Extract a function only when it names meaningful logic, hides substantial complexity, isolates a side effect or failure boundary, or is reused.
- Prefer guard clauses and `continue` over deep nesting.
- Avoid clever one-liners, nested ternaries, dense chains, speculative abstractions, classes, and generic frameworks.
- Keep related declarations and operations together.

## Make intent visible

- Use precise names with business meaning and units.
- Distinguish source records, derived values, indexes, and mutation payloads.
- Use one representation for IDs, statuses, dates, blanks, and field values.
- Comment decisions, constraints, and non-obvious Airtable behaviour—not syntax.
- State only material assumptions.

## Use inputs and schema safely

- Destructure every table, view, and field from `input.config()` at the start.
- Never hard-code table, view, or field names or IDs.
- Use configured Airtable models directly; resolve configured identifiers once.
- Do not add schema constants, fallback literals, or placeholders.
- If code needs an unmapped dependency, stop and return to input verification.

## Optimise Airtable work

- Optimise queries, external requests, mutation count, and algorithm shape before syntax.
- Query only required fields and use a configured filtered view when it materially reduces records.
- Use `selectRecordAsync` for one known record.
- Build `Map` or `Set` indexes for repeated matching, grouping, membership, or deduplication.
- Avoid repeated `.find()` or `.filter()` calls inside large loops.
- Compute stable values once and avoid unnecessary passes, copies, and cell-value conversions.
- Prepare mutation payloads before writing.
- Write only effective changes and batch mutations in groups of no more than 50.
- Avoid `await` inside record loops unless work must be sequential or cannot be batched.
- Do not parallelise Airtable mutations blindly.

## Use JavaScript deliberately

- Prefer `const`; use `let` only for reassignment.
- Use destructuring, optional chaining, and nullish coalescing when they keep the source and meaning clear.
- Use array methods for one clear transformation or question; use `for...of` for branching, accumulation, logging, early exits, or error handling.
- Avoid large spread copies and unclear shared mutation.

## Handle operations proportionately

- Fail early with plain-English context when the whole run is unsafe.
- Catch errors only to add context, recover, retry safely, or skip deliberately.
- Keep error boundaries close to the failing operation.
- Separate record-level failures only when partial completion is useful and safe.
- Add logs, counters, dry runs, retries, checkpoints, and resume logic only when justified.
- Never log secrets or unnecessary sensitive data.
- Prefer safe, resumable termination over incomplete work near timeout.

## Final check

- The script does only the requested job.
- Every dependency exists in the verified Context and comes from `input.config()`.
- Each numbered script is independently executable with explicit handoffs.
- Queries are minimal; repeated lookups are indexed.
- Writes contain only changes and are batched correctly.
- Failure boundaries protect consistency.
- Every function, branch, variable, comment, allocation, log, query, and write earns its place.

Prefer boring, obvious, measurable efficiency over sophisticated-looking code.
