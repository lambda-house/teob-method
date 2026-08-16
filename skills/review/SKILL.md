---
name: review
description: "Audit an existing system, design document or architecture diagram against the Outside-In Method: type every component, run the deletion test, divide to check each mechanism is proportional to the requirement it cites, build the traceability map, and label every figure's provenance. Use when reviewing a design, preparing an architecture review, inheriting an unfamiliar system, auditing a set of designs at once, or asked why a component exists."
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

Work in this order. Do not skip to step 5; the map is worthless if the outside was never established.

### 0 · State what you can see

Record the coverage before recording a finding: what form the artifact arrived in, what fraction of it was legible, and what you did not read.

Diagrams are the usual casualty — raster images with no extractable text, a photographed whiteboard, a slide exported flat. Prose normally describes the same components well enough to type them, but a diagram can carry an edge the prose omits. So mark every finding a legible figure could overturn: *"this component has no declared owner"* is provisional until somebody has looked at the picture, while a finding resting on arithmetic in the text is not.

### 1 · Recover the outside

Find, or reconstruct, what is outside the system boundary. Two kinds, both required:

- **Demand** — who calls this, at what volume, with what tolerances (latency as a percentile, staleness in units of time, ordering scoped to a key, durability, accuracy). Adversaries are clients too. Your own dependencies are client relationships reversed.
- **Supply** — the organisation that has to run and pay for it: hosting posture, contracts already held, team capability, on-call reality, cost envelope.

If the artifact under review never states these, **that is the first finding** and it usually explains most of the others. Say so plainly rather than inventing them.

**Supply gets its own table in the output** — hosting posture, contracts held, team capability, cost envelope, lifecycle. Fill in what the artifact states and leave the rest blank. An empty cell is a finding; an empty table is a loud one. Design documents are demand-side by habit, and the half that says who pays for this and who is woken by it is the half that goes missing. A cost figure does not on its own close the gap: *"$150 000 a day"* measured against *"low infrastructure cost"* is a comparison that cannot resolve, because only one side carries a number.

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
- **Answers with an earlier design decision** → self-derived. Legitimate, but it must name its parent decision, and the chain gets recorded — see below.
- **No answer** → uncited mechanism.

**Decide the posture before you run the test.** It changes what an uncited mechanism means, never whether you report it.

| Posture | The artifact is | The test is | Report an uncited mechanism as |
|---|---|---|---|
| **Greenfield** | A design not yet built | A razor | Delete it, or surface the requirement it implies |
| **Brownfield** | A running system | Chesterton's fence — the reason may exist undocumented, or have expired | *"No recorded reason — confirm before removing."* Recommend an incremental cutover, never a deletion |
| **Didactic** | A worked example whose purpose is to demonstrate the mechanism | Inverted | *"The stated requirement cannot discriminate this mechanism."* The finding is against the requirement, not the mechanism |

Oversizing is the point of a teaching artifact, so *"delete this"* is the wrong verdict and *"uncited"* is a slander. What survives is the real defect: a requirement so weak that it cannot tell the demonstrated design from one a hundredth its size.

**Record the chain; do not rule on it.** When the answer is an earlier design decision, follow it — which decision produced *that* one, and so on, until you reach a client constraint or run out. A chain is only as cited as its weakest link, and a link that fails at depth 3 is invisible while each component is checked alone.

| Root | Depth | Weakest link | Root is a client constraint? |
|---|---|---|---|
| "High availability" — no figure, no client named | 5 | The root itself | No |
| A 100 ms latency budget | 4 | Link 3 — *"the index grows too large for one server"*, never computed | Yes |

The second is the instructive shape: two properly derived links, then an unquantified premise, and everything after it inherits the gap.

Record root, depth, weakest link, and whether the root is a client constraint. **Whether a given chain is acceptable is the human's call, not this skill's** — do not write a rule for when a self-derived chain is allowed.

### 4 · Divide — is the citation proportional?

A mechanism that passes the deletion test can still be wrong by two orders of magnitude. Step 3 asks *whether* a requirement produced this. It never asks *how much* requirement.

So for every mechanism that survives step 3, do the division:

> **What the cited requirement demands ÷ what the mechanism supplies. Record the ratio.**

A ratio near 1 is a mechanism sized to its job. A ratio in the tens is headroom, and the artifact has to say which factor it is — growth, peak, or failover. A ratio in the hundreds or above means the stated requirement cannot discriminate the mechanism, and the finding is recorded against the requirement, exactly as in the didactic case above.

A requirement of 10 000 IDs/s against a design capacity of 4.19 billion IDs/s is a ratio of 419 430×, and four sequence bits on one machine meet the requirement. Queues, workers and horizontal autoscaling appear for a load that divides out at 185 requests per second. Sharding, a shard-map manager and a rebalancing story are introduced for a dataset growing at 146 GB a year.

If the division cannot be done because the mechanism's capacity is nowhere stated, **that is the finding.** A mechanism nobody sized cannot be shown to be the right size.

### 5 · Build the traceability map

| Requirement (testable) | Mechanism | Verified by |
|---|---|---|
| Consumer reads, p99 ≤ 200 ms, from three regions | Derived view replicated per region | Per-region latency SLO with error budget |

Every requirement must be **testable** — a number, a bound, or a condition. "Highly available" is not a requirement; "99.9% of writes acknowledged within 200 ms during the nightly partner window" is. Every tolerance names the conditions it holds under.

Read the map in both directions. Then read it a third way: **a mechanism serving three requirements is a coupling finding** — when it breaks, three promises break at once.

### 6 · Label every figure's provenance

| Label | Means |
|---|---|
| **measured** | Came from production telemetry or a load test. Name the source |
| **contractual** | Came from an SLA, a contract, or a written commitment |
| **assumed** | Somebody made it up — including you |
| **unconsumed** | Stated, and nothing divides it, bounds anything by it, or decides with it. Write *nothing* in the Supports column |
| **missing** | Required by an arithmetic step and never stated. Name the input, the step that needed it, and the sensitivity band across plausible values |

The first three say where a figure came from; the last two say whether anything uses it. They are different axes and they combine — *measured, unconsumed* and *assumed — declared* are both real labels.

Every **assumed** figure becomes a risk-register entry with the decision it supports. This is the single most useful thing this skill produces, because assumed figures are load-bearing far more often than anyone admits.

**Unconsumed is the label most reviews miss**, because a labelled figure looks like diligence. A scale figure elicited in the first five minutes and then divided by nothing is a question that should not have been asked, and the design is resting on whatever was reached for instead.

**Never invent a number to fill a gap.** A missing figure is a finding and now has a row. If arithmetic requires an input that does not exist, state the input, mark it **missing**, show the sensitivity, and continue — a retention promise of *forever* with no message rate behind it spans two orders of magnitude across plausible rates, and that span is the finding.

### 7 · Check the arithmetic — the quantity first, then the sum

Two passes, and the order is the whole trick:

1. **Demand arithmetic, before any technology is named** — requests/second from stated volumes, bytes/day, working-set size, fan-out. Round out loud: a day is ~10⁵ seconds.
2. **Capacity arithmetic, after the store is chosen** — node counts, replica counts, headroom.

A design that names a technology before pass 1 has decided by reference architecture. Compute pass 1 yourself and compare: *"the assertion is N nodes; the division gives 3."* Assertions and derivations disagree, and only one of them can be checked.

Then ask two questions of every figure, in this order:

1. **Is this the quantity the decision needs?** An entitlement — users × quota — is not a storage requirement; the requirement is what they actually store per day. One design computes 500 PB of entitlement exactly, against a real consumption of 10 TB/day that takes 137 years to reach it, and sizes its storage against the wrong one.
2. **Does it compute?** Recompute independently before reading the stated answer.

**A correct answer to the wrong division is more dangerous than an arithmetic error, because it survives checking.** The error gets caught by the next reader. The wrong quantity gets quoted.

### 8 · Enumerate the zero-tolerance promises

By name, and separately from everything above: which promises have an acceptable failure count of **zero** rather than a percentile? Consent honoured. Data not lost. A prohibited item never served. Money never taken twice.

For each — the promise, the mechanism carrying it, that mechanism's failure mode, and the counter that proves it. Three things to look for, none of which the earlier steps will surface on their own:

- **A percentile attached to a zero-tolerance promise is already a finding.** "99.9% of opt-outs honoured" is a commitment to violate consent a thousand times per million.
- **A single unguarded mechanism.** One filter, one flag check, one callback. Ask what happens when it is down, skipped, or raced. A consent check that runs *before* a queue and never again does not survive an opt-out during the retry window; a status flipped by a callback that never arrives stays pending forever.
- **No counter.** A promise whose count must be zero and which nothing counts is unfalsifiable in production. That is a specific observability finding with a named owner, not a general plea for more metrics.

### 9 · Check the adjacent areas

Most reviews stop at the runtime picture. Carry it further — each is derived from the same outside:

- **Observability** — is detection proportioned to consequence, or to what was easy to emit? Zero-tolerance promises have their own step above; here, check that what *is* watched traces to a promise made to a client.
- **Operations** — can this organisation actually staff, host and afford this shape?
- **Delivery** — does each boundary buy something (independent deployment, fault isolation, a genuinely different scaling axis, an existing team boundary)? "It'll scale better" is not a force.
- **Codebase lifecycle** — spike, one-off, prototype-becoming-platform, or platform? Does the repo topology match?

## When the artifact declares its own process

Some artifacts carry their own method — a design template, a house checklist, a prescribed order of steps, a framing chapter every later chapter obeys. **Review the process before the design it produced**, and review it as a first-class component: which constraint produced this ordering?

A process that says *draw the components, then check whether the arithmetic supports them* has made the arithmetic a validation step, and validation steps are the ones that get skipped. A process that fixes the inventory of components before any problem is named has decided the design in advance.

A finding about the process is the **parent** of the design findings it produced — cite it that way. The design finding is then not a lapse by its author but a faithful execution of the instruction, which changes both who fixes it and what fixing it costs.

## More than one artifact at a time

Reviewing a set — a family of services, a documentation corpus, a team's last twelve designs — changes two things.

- **Add a recurrence column** to every output table: in how many of the N artifacts does this finding appear? A finding at 1 of 12 is a lapse. The same finding at 9 of 12 is a property of something they share.
- **Run a common-cause pass. Mandatory when N > 1.** For each recurring finding ask: *does one shared source produce this in k artifacts?* A template, a house reference architecture, a platform default, a framing document. Report the common cause once as the parent and the k instances as its children.

The most valuable findings in a corpus are invisible from inside any single artifact. Twelve designs reaching for the same uncited component are not making one mistake twelve times; they are inheriting one decision made somewhere else.

## Output format

Report in this order, most costly first:

```
## Scope
What was reviewed, in what form, what fraction was legible, what was not checked,
and which findings would change if a figure contradicted the prose.

## Uncited mechanisms
| Component | Type | Deletion test | Ratio | Disposition |

## Chains
| Root | Depth | Weakest link | Root is a client constraint? |

## Unmapped requirements
| Requirement | Stated where | No mechanism found for |

## Zero-tolerance promises
| Promise | Carrying mechanism | Failure mode | Counter that proves it |

## Supply
| Hosting posture | Contracts held | Team capability | Cost envelope | Lifecycle |

## Provenance
| Figure | Value | Label | Supports | Risk if wrong |

## Arithmetic
| Claim in the document | Independent derivation | Right quantity? | Agrees? |

## Adjacent areas
| Area | Derived, or assumed? | Finding |
```

Reviewing more than one artifact adds a **Recur** column — *k of N* — to every table above.

Close with **the three most expensive findings**, in one sentence each.

## Rules

- **Do not rewrite the design.** This skill reports; the human decides.
- **Do not soften a finding to be agreeable.** An uncited mechanism is uncited even if it is a component everybody uses.
- **Do not report a finding you cannot demonstrate.** Show the arithmetic, quote the document, or drop it.
- **Say what you could not check.** A review that silently skipped the storage layer reads as a review that cleared it.
- Where the artifact is thin, the finding is the thinness — not an excuse to fill it in from a reference architecture.
- **Record a self-derived chain; do not adjudicate it.** Root, depth, weakest link, whether the root is a client constraint. What the chain is worth is the human's decision.
- **Do not report a ratio you have not divided.** "Oversized" without the division is taste, and taste arguments get settled by seniority.

## References

| File | Load when |
|---|---|
| `references/checklist.md` | Running the review — the per-primitive and supply question sets, the anti-pattern list, the severity order |
