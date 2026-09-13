# 29. End-to-End Implementation Workflow

Use this as the default sequence for a non-trivial change.

### Phase 1 - Understand

- Read requirement and acceptance criteria.
- Inspect current implementation and callers.
- Inspect related tests.
- Identify domain vocabulary.
- Identify invariants.
- Identify trust/security boundaries.
- Identify persistence and external effects.
- Identify concurrency/retry behavior.
- Record any assumptions that materially affect design.
### Phase 2 - Design the smallest correct change

- Decide the module that owns the behavior.
- Define/adjust types first when they clarify valid states.
- Define public function/interface around domain intent.
- Keep vendor/framework specifics behind existing boundaries.
- Decide expected errors and exceptional failures.
- Decide transaction/resource lifetime.
- Decide test level.
- Do not design extension points for requirements that do not exist.
### Phase 3 - Protect behavior

- Add a failing behavioral test or characterization test.
- Include the most important edge/failure case.
- Make time/network/random dependencies controllable.
- Keep test setup readable.
### Phase 4 - Implement

- Validate boundary input.
- Authorize sensitive action.
- Express high-level operation in domain language.
- Keep calculations separate from effects where useful.
- Use atomic/transactional operations for invariants.
- Call dependencies through owned boundaries.
- Map dependency failures appropriately.
- Bound time/retries/resources.
- Return truthful outcome.
### Phase 5 - Refactor

- Rename ambiguous concepts.
- Split mixed abstraction levels.
- Remove real duplication.
- Reduce unnecessary nesting.
- Reduce public surface.
- Delete obsolete comments/code.
- Ensure comments explain intent/constraints, not syntax.
- Align source layout with reading flow.
### Phase 6 - Verify

Run relevant:

- format
- lint
- typecheck
- unit tests
- integration tests
- build
- security/static checks

Then inspect the diff.
### Phase 7 - Production sanity check

Ask:

- What happens on timeout?
- What happens if this runs twice?
- What happens if two workers run it concurrently?
- What happens if the process dies halfway through?
- What happens with malformed/hostile input?
- What happens when the dependency returns an unexpected response?
- What would I see in logs/metrics/traces if it fails?
- Can any secret or private data leak?
- Can a caller perform this on another user's/tenant's resource?

For business-critical code, these questions are not optional.
