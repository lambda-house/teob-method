# Review checklist

## Per-primitive question set

**Surface** — Is authn/authz stated, and at which hop? Is there a request contract with a versioning policy? Is there a quota per client class, and is it policy rather than capacity? Which invariants are enforced here rather than merely checked? What happens on partial results — is there a documented behaviour, or does it silently truncate?

**Channel** — Which delivery guarantee? What is the ordering key, and is ordering scoped to it? What is the failure mode when the far side is down? What is the retry policy, and does it have jitter? **Who owns the payload schema, and what is the compatibility mode?** Is TTL stated, and is it TTL on the message rather than retention on a store?

**Processor** — Stateless, stateful or batch? If it combines partial results, how do they compose — and is that composition correct? If it approximates, what is the error bound, and who agreed to it?

**Store** — Is this the truth of something, or derived from something? If derived: what is the staleness bound, and is it stated in units of time? What is the access pattern, and does the store family match it? What is the retention, and who decided it? Is there a resharding path?

## Anti-patterns to look for

| Pattern | What it looks like |
|---|---|
| **Reference architecture in disguise** | Technologies appear before any number. The requirements section reads as justification written afterwards |
| **The undeclared API** | An async channel with no schema owner, no compatibility mode and invisible consumers |
| **Synchrony mistaken for reliability** | Every write goes to the strict store because "it must be reliable" |
| **Capacity before demand** | Node counts and coordination protocols exist before requests/second was ever derived |
| **Metrics of convenience** | Dashboards of what was easy to emit; nothing traces to a promise made to a client |
| **The silent range** | "Between 1 and 128 hosts" — a range nobody narrowed, with machinery built for the top of it |
| **Granularity as identity** | "We're microservices" as an answer, with no force cited per split |
| **Wrong as a degradation rung** | Serving stale data as if fresh, with no as-of marker |
| **Unlabelled figures** | Numbers with no provenance, load-bearing under a decision |

## Severity ordering

Rank findings by cost, not by how easy they were to spot:

1. **Correctness** — a mechanism that produces wrong answers under a stated condition (bad merge semantics, an invariant enforced nowhere, an ordering assumption that does not hold).
2. **Unmapped requirement** — a promise with no mechanism. Discovered in production by definition.
3. **Arithmetic that does not survive checking** — an assertion contradicted by its own inputs.
4. **Uncited mechanism** — recurring cost with no recorded reason.
5. **Unlabelled provenance** — a load-bearing figure nobody can source.
6. **Adjacent-area gaps** — observability, operations, delivery, lifecycle left to "later".

## What not to do

- Do not propose a redesign. Report the findings; the human decides the response.
- Do not fill a gap from a reference architecture — the gap is the finding.
- Do not report a finding you cannot demonstrate with a quote or an arithmetic step.
- Do not treat an uncited mechanism in a **running** system as a deletion instruction. Chesterton's fence: report it as "no recorded reason — confirm before removing".
