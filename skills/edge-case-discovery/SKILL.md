---
name: edge-case-discovery
description: >-
  Systematically enumerates failure modes, boundary conditions, and edge cases
  for a design before implementation begins. Invoked automatically by the
  brainstorming skill after Phase 1 (architecture) is approved. Produces a
  categorized list of edge cases that feeds directly into Phase 2 (error handling).
---

# Edge Case Discovery

## Overview

Systematically enumerate what can go wrong before writing any code. The goal is to surface failure modes, boundary conditions, and adversarial inputs that the happy path misses — so Phase 2's error handling addresses *discovered* problems, not *imagined* ones.

**Core principle:** You cannot handle what you have not enumerated. Vague statements like "handle errors appropriately" produce vague error handling. Specific enumeration produces specific handling.

<HARD-GATE>
Do NOT proceed to Phase 2 of brainstorming until you have completed edge case enumeration and the user has reviewed the results. Every design has edge cases. "This is too simple" is a rationalization — simple designs fail on simple edge cases.
</HARD-GATE>

## When to Use

This skill is invoked automatically by the brainstorming skill between Phase 1 approval and Phase 2 presentation. You do not need to invoke it manually.

## Input

You need the approved Phase 1 design (architecture + components) as context. The enumeration analyzes that specific design — not abstract possibilities.

## The Enumeration Process

Work through each category below against the approved architecture. For each category, identify specific edge cases relevant to *this* design. Skip categories that genuinely do not apply — but state why you are skipping them so the user can correct you.

### 1. Inputs and Boundaries

Examine every input the system accepts — user input, API parameters, file contents, environment variables, configuration values.

For each input, ask:
- What happens when it is **empty**, **null**, or **missing**?
- What happens at **minimum** and **maximum** valid values?
- What happens **one past** the boundary (off-by-one)?
- What happens with **unexpected types** (string where number expected, nested object where flat expected)?
- What happens with **malformed** data (invalid JSON, truncated input, wrong encoding)?
- What happens with **adversarial** input (injection attempts, extremely large payloads, deeply nested structures)?

<example>
<good>
Feature: User profile update API

Edge cases identified:
- Display name: empty string, 1 character, exactly 255 characters, 256 characters (boundary), contains only whitespace, contains Unicode combining characters, contains null bytes
- Email: missing @ sign, multiple @ signs, exceeds 254-character RFC limit, contains internationalized domain names
- Avatar URL: empty, non-URL string, URL to non-image resource, URL exceeding 2048 characters, data: URI with embedded payload
</good>
<bad>
Edge cases identified:
- Invalid input
- Bad data
- Edge cases in user fields
</bad>
</example>

### 2. State and Transitions

Examine every state the system can be in and every transition between states.

For each stateful component, ask:
- What are the **valid states** and **valid transitions**?
- What happens on an **invalid transition** (e.g., canceling an already-completed order)?
- What happens when the system is in an **intermediate state** and a request arrives (partially initialized, mid-migration, shutting down)?
- What happens when **two operations conflict** (concurrent writes to the same resource, simultaneous state transitions)?
- What happens on **partial failure** (first of three writes succeeds, second fails — is state consistent)?

### 3. External Dependencies

Examine every external system the design touches — databases, APIs, filesystems, network services, hardware.

For each dependency, ask:
- What happens when it is **unavailable** (network down, service crashed, DNS failure)?
- What happens when it is **slow** (responds after timeout, 10x normal latency)?
- What happens when it returns **unexpected data** (schema changed, extra fields, missing fields, different version)?
- What happens when it returns an **error** (rate limit, auth failure, server error, malformed response)?
- What happens when **credentials expire** or are revoked mid-operation?

### 4. Timing and Ordering

Examine every operation where timing or order matters.

Ask:
- What happens if operations arrive **out of order** (event B before event A)?
- What happens if the **same operation runs twice** (duplicate requests, retry storms, idempotency)?
- What happens under **race conditions** (two users act on the same resource simultaneously)?
- What happens when an operation **takes longer than expected** (user navigates away, connection drops mid-request)?
- What happens at **time boundaries** (midnight, DST transitions, leap seconds, timezone differences)?

### 5. Resources and Limits

Examine every resource the system consumes — memory, disk, connections, file handles, API quotas.

Ask:
- What happens when a resource is **exhausted** (out of memory, disk full, connection pool empty)?
- What happens when a resource **leaks** (connections not closed, temporary files not cleaned up)?
- What happens at **scale** (10x, 100x, 1000x expected load)?
- What happens when **quotas or rate limits** are hit (API limits, storage limits, bandwidth limits)?

### 6. Domain-Specific Edge Cases

After completing the general categories, apply the domain-specific checklist that matches your project type. These are in companion files in this directory:

- **@cli-checklist.md** — Command-line tools and scripts
- **@web-api-checklist.md** — HTTP APIs and web services
- **@frontend-checklist.md** — User interfaces and client-side applications
- **@data-pipeline-checklist.md** — Data processing, ETL, and streaming systems
- **@infrastructure-checklist.md** — Infrastructure, deployment, and platform tooling

If multiple checklists apply (e.g., a CLI that manages infrastructure), use all relevant ones. If none apply, skip this step and note what kind of system you are working with — the general categories above still apply.

## Output Format

Present the enumerated edge cases as a categorized list, grouped by the categories above. For each edge case, include:

1. **What can happen** — the specific failure mode
2. **Severity** — how bad is it if unhandled? (data loss, crash, wrong result, poor UX, minor annoyance)
3. **Recommended handling** — brief suggestion (validate and reject, retry with backoff, degrade gracefully, log and alert, etc.)

The user reviews this list and may add, remove, or reprioritize edge cases. Once confirmed, these feed directly into Phase 2 of the brainstorming design.

## Anti-Patterns

| Pattern | Problem |
|---------|---------|
| "Handle errors appropriately" | Not an edge case. Name the specific error and the specific handling. |
| Listing only happy-path variations | Edge cases are about what goes *wrong*, not what goes *right in different ways*. |
| "Validate all inputs" | Which inputs? What validation? Against what rules? Be specific. |
| Skipping categories because "that won't happen" | If you know it won't happen, explain why. Otherwise, enumerate it. |
| Listing 50 edge cases with equal weight | Prioritize by severity. Data loss and crashes come before minor UX annoyances. |

## Verification

Before presenting the edge case list to the user, verify:

- [ ] Every category was considered (and skipped categories have stated reasons)
- [ ] Each edge case is specific to *this* design, not a generic best-practice platitude
- [ ] Each edge case includes severity and recommended handling
- [ ] The list covers both expected failures (user error, bad input) and unexpected failures (infrastructure, timing, resource exhaustion)
- [ ] At least one adversarial/security edge case was considered
- [ ] Domain-specific checklist was applied if applicable
