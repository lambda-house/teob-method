---
name: review
description: "Audit an existing system, design document or architecture diagram against the Outside-In Method: type every component, run the deletion test, build the traceability map, and label every figure's provenance. Use when reviewing a design, preparing an architecture review, inheriting an unfamiliar system, or asked why a component exists."
---

# Outside-In Review

Audit a system that already exists — code, a design document, a diagram, or a conversation — against the discipline that every mechanism must cite a requirement and every requirement must map to a mechanism.

The output is **a table someone can take to a review**, not an essay. Findings are ranked by what they would cost, and every number carries where it came from.

Full method: <https://teob.cc/method>

## The two rules this skill enforces

| Rule | Failure it detects |
|---|---|
| Every mechanism cites a requirement | **Uncited mechanism** — speculation, or something that entered by habit. It is a licence, a pipeline, a line in the bill and an on-call weekend |
| Every requirement maps to a mechanism | **Unmapped requirement** — a gap nobody designed for, usually discovered in production |

Both checks are mechanical. Neither is a matter of taste — which is the point, because taste arguments get settled by seniority.

## Procedure

Work in this order. Do not skip to step 4; the map is worthless if the outside was never established.

### 1 · Recover the outside

Find, or reconstruct, what is outside the system boundary. Two kinds, both required:

- **Demand** — who calls this, at what volume, with what tolerances (latency as a percentile, staleness in units of time, ordering scoped to a key, durability, accuracy). Adversaries are clients too. Your own dependencies are client relationships reversed.
- **Supply** — the organisation that has to run and pay for it: hosting posture, contracts already held, team capability, on-call reality, cost envelope.

If the artifact under review never states these, **that is the first finding** and it usually explains most of the others. Say so plainly rather than inventing them.

### 2 · Type every component

Assign each component exactly one primitive. Types apply to **roles, not products** — one product routinely occupies several nodes.

| Primitive | Is | Must declare |
|---|---|---|
| **Surface** | Where a client touches the system | Authn/authz and at which hop · request contract and versioning · quota per client class · which invariants are enforced here · pagination and partial-result behaviour |
| **Channel** | A transfer between two nodes | Guarantee (at-most/at-least/effectively-once) · ordering key · failure mode · retry policy · payload schema and compatibility mode · owner |
| **Processor** | Derives | Stateless / stateful / batch · how partial results compose · error bounds if it approximates |
| **Store** | Holds state | What it is the truth of, or what it is derived from · access pattern · retention · staleness bound if derived |

Decomposition is where most findings appear:

- A **log** is not one thing — it is a producer channel, a store with a retention window, and one consumer channel per reader. Three primitives, three owners.
- A **cache** is a derived store with an invalidation contract, not a fifth primitive.
- A **database with CDC** is a store *and* a producer channel.
- **Message TTL is not store retention.** They are different properties on different primitives, and TTL is vacuous on a synchronous channel.

Flag any component whose declared properties are missing. An async channel with no stated schema owner has **published an API by accident** and promised nothing about it.

### 3 · Run the deletion test

For each component: *which constraint's removal would delete this?*

- **Answers with a client constraint** → keep, record the citation.
- **Answers with an earlier design decision** → self-derived. Legitimate, but it must name its parent decision, and a chain of self-derived requirements justifying a component is itself a finding.
- **No answer** → uncited mechanism.

**Brownfield asymmetry — apply this.** In a design under review the deletion test is a razor. In a running system it is Chesterton's fence: the requirement may exist and be undocumented, or have expired. Report an uncited mechanism in production as *"no recorded reason — confirm before removing"*, never as *"delete this"*. Recommend an incremental cutover, not a deletion.

### 4 · Build the traceability map

| Requirement (testable) | Mechanism | Verified by |
|---|---|---|
| Consumer reads, p99 ≤ 200 ms, from three regions | Derived view replicated per region | Per-region latency SLO with error budget |

Every requirement must be **testable** — a number, a bound, or a condition. "Highly available" is not a requirement; "99.9% of writes acknowledged within 200 ms during the nightly partner window" is. Every tolerance names the conditions it holds under.

Read the map in both directions. Then read it a third way: **a mechanism serving three requirements is a coupling finding** — when it breaks, three promises break at once.

### 5 · Label every figure's provenance

| Label | Means |
|---|---|
| **measured** | Came from production telemetry or a load test. Name the source |
| **contractual** | Came from an SLA, a contract, or a written commitment |
| **assumed** | Somebody made it up — including you |

Every **assumed** figure becomes a risk-register entry with the decision it supports. This is the single most useful thing this skill produces, because assumed figures are load-bearing far more often than anyone admits.

**Never invent a number to fill a gap.** A missing figure is a finding. If arithmetic requires an input that does not exist, state the input, mark it assumed, show the sensitivity, and continue.

### 6 · Check the arithmetic runs in the right order

Two passes, and the order is the whole trick:

1. **Demand arithmetic, before any technology is named** — requests/second from stated volumes, bytes/day, working-set size, fan-out. Round out loud: a day is ~10⁵ seconds.
2. **Capacity arithmetic, after the store is chosen** — node counts, replica counts, headroom.

A design that names a technology before pass 1 has decided by reference architecture. Compute pass 1 yourself and compare: *"the assertion is N nodes; the division gives 3."* Assertions and derivations disagree, and only one of them can be checked.

### 7 · Check the adjacent areas

Most reviews stop at the runtime picture. Carry it further — each is derived from the same outside:

- **Observability** — is detection proportioned to consequence, or to what was easy to emit? Which promises have **zero tolerance** (a count that must be zero, not a percentile)?
- **Operations** — can this organisation actually staff, host and afford this shape?
- **Delivery** — does each boundary buy something (independent deployment, fault isolation, a genuinely different scaling axis, an existing team boundary)? "It'll scale better" is not a force.
- **Codebase lifecycle** — spike, one-off, prototype-becoming-platform, or platform? Does the repo topology match?

## Output format

Report in this order, most costly first:

```
## Uncited mechanisms
| Component | Type | Deletion test | Disposition |

## Unmapped requirements
| Requirement | Stated where | No mechanism found for |

## Provenance
| Figure | Value | Label | Supports | Risk if wrong |

## Arithmetic
| Claim in the document | Independent derivation | Agrees? |

## Adjacent areas
| Area | Derived, or assumed? | Finding |
```

Close with **the three most expensive findings**, in one sentence each.

## Rules

- **Do not rewrite the design.** This skill reports; the human decides.
- **Do not soften a finding to be agreeable.** An uncited mechanism is uncited even if it is a component everybody uses.
- **Do not report a finding you cannot demonstrate.** Show the arithmetic, quote the document, or drop it.
- **Say what you could not check.** A review that silently skipped the storage layer reads as a review that cleared it.
- Where the artifact is thin, the finding is the thinness — not an excuse to fill it in from a reference architecture.

## References

| File | Load when |
|---|---|
| `references/checklist.md` | Running the review — the per-primitive question set and the anti-pattern list |
