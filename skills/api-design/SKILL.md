---
name: API Design
description: Use when designing or changing an endpoint, route, RPC method, webhook, event schema, or any payload two systems exchange — and before writing the handler. Triggers on "add an endpoint", "design the API", "REST", "GraphQL", "schema for", "response shape", "status code", "pagination", "rate limit", "how should clients call this", "breaking change", or a field added to or removed from something already in use. Also use when a client and server must agree on a contract.
---

# API Design

Design the contract before the handler. A contract that has shipped is expensive to change.

## Sequence

1. Define the resource and its identity.
2. Define the operations and their semantics.
3. Define the request and response shapes.
4. Define every error the caller can receive.
5. Define versioning and compatibility rules.
6. Only then write the handler.

## Resources and operations

- Name resources as nouns, operations as HTTP verbs. `POST /orders` not `/createOrder`.
- Plural collection names. `/users`, not `/user`.
- Nest only where the child cannot exist without the parent. Prefer flat paths for anything that moves: `GET /orders/{id}/items` reads better than `GET /users/{id}/orders`.
- Keep the identifier in the path and everything else in the body or query.
- `PUT` replaces, `PATCH` merges, `POST` creates or triggers. Do not use `POST` for reads.
- Keep action endpoints rare. If a verb appears in the path, consider whether it is a missing field or a missing resource.

## Request and response

- Consistent casing and naming conventions across every endpoint. Verify against existing endpoints rather than choosing fresh.
- Use the project's existing envelope, status conventions, and date format.
- Return the resource, not the ORM object. Never serialize a whole entity with its internal columns and relations.
- Optional fields omitted rather than sent as `null`, unless `null` carries meaning.
- Lists carry enough metadata to paginate without a second request where practical.
- No secrets, password hashes, internal IDs, or provider credentials in responses.

## Validation

- Validate at the boundary, once, before business logic.
- Reject unknown fields on strict endpoints; it turns a typo into an error instead of a silent no-op.
- Enforce bounds: string length, array size, numeric range, page size ceiling.
- Validate types and shapes, not just presence. `[]` is present and valid; `{}` is neither.
- Cross-field rules belong in one validator, not scattered across the handler.
- Return every validation error at once where the client can act on them.

## Errors

A consistent error shape is the single most valuable property of an API.

- Use one error shape everywhere: a stable machine-readable `code`, a human-readable `message`, optional structured `details`.
- Never change the `code` for an existing condition. Clients branch on it.
- Distinguish caller error (400, 401, 403, 404, 409, 422) from server error (500) from upstream failure (502, 503, 504).
- Validation → 400 or 422, consistently. Pick one and never mix.
- Unauthenticated → 401. Authenticated but not permitted → 403. Do not use 403 to hide existence; that is a deliberate choice, so make it consistently.
- Missing resource → 404. Conflicting state → 409. Never 500 a client error.
- Include a trace or correlation ID in 5xx responses. Log the detail server-side; do not leak internals to the caller.

## Pagination

- Offset pagination is fine for small, stable, admin-facing lists. It gets slow and can skip or duplicate rows as data changes.
- Cursor or keyset pagination for anything large, ordered, or frequently written.
- Default page size small, with a hard maximum. Never allow unbounded.
- Return the cursor or next page link, not just a bare list.

## Idempotency

- `POST` that creates, charges, sends, or schedules should accept an idempotency key. Retries are normal; double charges are not.
- `DELETE` should be idempotent: deleting an absent resource is a success.
- `PUT` is idempotent by definition. Do not put side effects in it.
- Anything that mutates money, sends a message, or calls a provider must be safe to retry. If it cannot be, say so explicitly in the contract.

## Concurrency

- Prefer optimistic concurrency: a version or ETag, and reject the write on mismatch.
- Never rely on check-then-act without a transaction or a constraint.
- State the consistency guarantee the caller should expect. "Strongly consistent" and "eventually consistent" are contracts.

## Versioning

- Decide how the contract changes before you need it, then write it down.
- Additive changes (new optional field, new endpoint, new enum value in a tolerant direction) are usually backward compatible.
- Breaking changes: removing or renaming a field, narrowing a type, adding a required field, changing default behavior, tightening validation.
- Never change the meaning of an existing field. That is the worst possible break because old clients keep working and return wrong data.
- Adding an enum value breaks clients with exhaustive matching. Treat it as breaking unless the type is documented as open.
- A contract decision like this routes to the `architect-decisions` skill.

## Webhooks and events

- Verify the signature on every delivery, and document the algorithm.
- Respond quickly and process asynchronously. A slow endpoint causes retries and duplicates.
- Assume duplicate and out-of-order delivery. Make handlers idempotent and versioned.
- Send a stable event identifier so receivers can deduplicate.
- Document the schema and the ordering guarantee, including that there is none unless you provide one.

## Output shape

Produce a contract before implementing:

```text
<METHOD> <path> — <one line>

Auth: <required|public>  Role: <role>  Scope: <object-level check>
Request:  <fields, types, required/optional, bounds>
Success:  <status, shape>
Errors:
  <code> <condition> → <status>
  <code> <condition> → <status>
Idempotent: <yes|no — key behavior>
Pagination: <scheme>
Rate limit: <limit, or "none">
Breaking: <yes/no — what existing callers break>

Open questions: <anything undecided>
```

## Rules

- Follow the existing API's conventions. Consistency inside one system beats theoretical correctness.
- Never invent product rules: prices, limits, permissions, or roles. If one is required and unknown, load `architect-decisions` or ask.
- Do not design for scale nobody asked for. Pagination, versioning, and idempotency are justified by real requirements.
- Do not break an existing contract without saying so explicitly and offering the migration path.
- Prefer fewer fields and stricter semantics over a wide, ambiguous surface.