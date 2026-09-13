# 3. AI Coding Agent Operating Procedure

An AI coding agent MUST treat coding as a verification loop rather than text generation.

## 3.1 Before editing

- Read the task literally. Identify the requested behavior, constraints, compatibility requirements, and acceptance criteria.

- Inspect relevant repository context. Read nearby code, types, tests, configuration, package manifests, and established patterns before inventing a design.

- Locate the ownership boundary. Determine which module should own the behavior. Do not place logic in a convenient file merely because it is easy to edit.

- Identify invariants. Ask what must remain true before and after the operation.

- Identify trust boundaries. Mark data arriving from users, networks, databases, files, queues, third-party services, environment variables, and other processes.

- Identify side effects. List database writes, file operations, network calls, queue messages, caches, logs, external payments, emails, and state mutations.

- Identify failure modes. Consider validation failure, missing data, authorization failure, timeout, dependency failure, partial completion, concurrency, duplicate delivery, cancellation, and resource exhaustion.

- Identify type guarantees. Determine which inputs are untrusted, which values may be absent, which states are finite, and where unchecked casts/dynamic values could bypass invariants.

- Identify scale assumptions. For performance- or architecture-sensitive work, determine expected load, bottlenecks, state ownership, idempotency, backpressure, and whether the current deployment unit can simply scale horizontally.

- Find existing tests. Prefer extending the existing test language and helpers over introducing a separate testing style.

- Plan the smallest coherent change. A correct local change is preferable to a broad cleanup unrelated to the task.

## 3.2 During implementation

- Add or update a test that captures the important behavior when practical.

- Implement the simplest correct path.

- Make names express domain intent.

- Keep state changes explicit.

- Keep high-level orchestration at one abstraction level.

- Isolate third-party APIs and infrastructure mechanics behind owned boundaries.

- Validate untrusted input at the trusted boundary.

- Preserve cleanup and transactional guarantees on failure.

- Add observability where a production failure would otherwise be opaque.

- Refactor only after the behavior is protected enough to change safely.

## 3.3 Before declaring completion

The agent MUST run the checks that are available and relevant:

- formatter;

- linter;

- type checker;

- unit tests;

- relevant integration tests;

- build;

- targeted security/static-analysis checks when configured.

Then review the actual diff. Check for accidental edits, dead code, debug output, stale comments, disabled tests, copied secrets, unhandled errors, widened API surface, and unnecessary dependencies.

Never claim that code compiles, tests pass, or a command succeeded unless that result was actually observed.
