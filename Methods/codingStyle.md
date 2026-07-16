# Airtable JavaScript Coding Style

Write like a high-performance production engineer: understand the job, choose the simplest efficient data flow, make correctness visible, and remove everything that does not earn its place.

## 1. Think before writing

- Define the required outcome, execution environment, inputs, outputs, invariants, side effects, expected record volume, and realistic failure modes.
- Read the authoritative Context files before choosing an implementation.
- Identify every table, view, and field the script will touch and confirm that each is supplied through `input.config()` before coding.
- Trace the data from input to final write. Identify expensive reads, repeated searches, external requests, mutations, and possible duplicate work.
- Choose the simplest algorithm and data structures that comfortably fit the expected scale.
- Resolve correctness and data-flow questions before polishing syntax.
- State only material assumptions; do not narrate obvious reasoning.

## 2. Design the execution path

- Make the primary flow readable from top to bottom: acquire, validate, load, transform, write, report.
- Validate script-wide requirements before expensive queries, external requests, or mutations.
- Separate pure transformation from Airtable reads, writes, and external side effects when doing so clarifies the flow.
- Prefer one deliberate pass over records. Combine related work when that reduces queries, scans, allocations, or mutations without obscuring intent.
- Decide duplicate, rerun, partial-failure, and timeout behaviour before mutations begin when those risks exist.
- Avoid speculative flexibility and hypothetical future requirements.
- If an Automation script becomes too large to understand, test, or operate safely, split it into focused script actions.
- Treat every script action as an isolated program. Do not assume variables, objects, caches, or other in-memory state carry into the next script.
- Pass inter-script values explicitly through Automation outputs and inputs, or persist state in Airtable.

## 3. Keep code direct and compact

- Prefer direct, linear code for short one-off operations.
- Extract a function only when it names a meaningful operation, contains substantial logic, isolates a side-effect or failure boundary, or is genuinely reused.
- Do not wrap two or three obvious lines merely to make the script look structured.
- Use the fewest lines that remain easy to scan, debug, and change safely.
- Prefer guard clauses and `continue` over deep nesting.
- Avoid clever one-liners, nested ternaries, dense chaining, and compressed control flow.
- Keep related declarations and operations close together.

## 4. Make intent obvious

- Use precise names that express business meaning and units: `recordsByEmail`, `updates`, `elapsedMs`, `cutoffDate`.
- Use one consistent representation for IDs, statuses, dates, blanks, and field values.
- Distinguish raw records, derived values, and mutation payloads through naming.
- Comment decisions, constraints, and non-obvious Airtable behaviour—not syntax.
- Remove comments that merely repeat the code.
- Use formatting to expose structure; do not rely on blank lines or headings to compensate for tangled logic.

## 5. Optimise the expensive work first

- Optimise algorithm shape, Airtable I/O, external requests, and mutation count before micro-optimising JavaScript syntax.
- Query only the fields required by the script.
- Use a filtered view when it materially reduces the records loaded.
- Use `selectRecordAsync` when only one known record is needed.
- Build a `Map` or `Set` once for repeated matching, membership checks, grouping, or deduplication.
- Do not use `.find()` or `.filter()` repeatedly inside large loops when an index can make the work linear.
- Avoid unnecessary passes, intermediate arrays, object copies, and repeated cell-value conversions on large datasets.
- Compute stable values once outside loops.
- Do not parallelise Airtable mutations blindly; respect platform limits and preserve predictable failure behaviour.
- Measure or estimate the dominant cost before adding complexity for performance.

## 6. Read and write Airtable efficiently

- Destructure every table, view, and field reference from `input.config()` at the start of the script.
- Never hard-code table names, table IDs, view names, view IDs, field names, or field IDs anywhere in the script.
- Use configured Airtable models directly when supplied by the execution environment.
- When configured inputs are identifiers, resolve each model once after destructuring and reuse the resolved reference.
- Batch create, update, and delete payloads in groups of no more than 50.
- Do not write a field when its effective value has not changed.
- Avoid `await` inside record loops unless operations must be sequential or batching is unavailable.
- Prepare mutation payloads completely before writing when practical.
- Prefer the smallest mutation that achieves the outcome.
- Keep configured table, view, and field references consistent with the authoritative schema.
- Do not create schema constants, fallback literals, or placeholders in the code.

## 7. Use JavaScript deliberately

- Prefer `const`; use `let` only when reassignment is part of the logic.
- Use destructuring when the source and meaning of values remain obvious.
- Use optional chaining and nullish coalescing when they model valid missing values clearly.
- Use `map`, `filter`, `find`, and `some` when each expresses one clear transformation or question.
- Prefer `for...of` when processing includes multiple conditions, accumulated outputs, logging, early exits, or error handling.
- Avoid spread syntax on large collections when it creates unnecessary copies.
- Avoid mutation of shared data unless it makes ownership and performance clearer.
- Do not add classes, generic frameworks, configuration layers, or utility libraries without task-specific value.

## 8. Handle errors at the correct boundary

- Fail early with a plain-English message for missing configuration, invalid schema, or conditions that make the entire run unsafe.
- Include useful identifiers and operation context in errors without exposing secrets or sensitive payloads.
- Catch an error only when the script can add context, recover, retry safely, skip deliberately, or produce a better outcome.
- Do not use broad `try/catch` blocks that hide the failing operation.
- Treat record-level failures separately only when partial completion is useful and safe.
- Preserve a clear distinction between skipped, unchanged, failed, and completed work when reporting matters.
- Rely on guarantees established by the environment and validated Context; do not defend against impossible states.

## 9. Keep operational behaviour proportionate

- Add logging, counters, dry-run behaviour, checkpoints, retries, and resume logic only when the task or risk justifies them.
- Log decisions and summaries that help diagnose or operate the script; avoid noisy progress narration.
- Never log secrets, tokens, passwords, or unnecessary sensitive data.
- Make reruns idempotent when duplicate writes or external side effects are plausible.
- Check elapsed time only when expected scale approaches the runtime limit.
- Prefer safe termination with resumable state over rushing incomplete writes near a timeout.

## 10. Final performance pass

Before finalising, verify:

- The script produces the requested result and nothing extra.
- Every schema and configuration reference exists in the authoritative Context.
- Every table, view, and field reference originates from the destructured input configuration.
- No table name, table ID, view name, view ID, field name, or field ID is hard-coded.
- Each numbered script is independently executable with its own verified input configuration.
- The dominant operations have appropriate algorithmic complexity.
- Queries request only necessary data.
- Repeated lookups use suitable indexes.
- Mutation payloads contain only real changes and are batched correctly.
- Failure boundaries protect data consistency.
- Names and control flow make the script understandable in one read.
- Every function, branch, variable, comment, allocation, log, query, and write earns its place.

Prefer boring, obvious, measurable efficiency over code that merely looks sophisticated.
