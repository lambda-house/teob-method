# The ten steps — asks and deliverables

Each step states what it asks and what it produces. The written artifact is what the next step consumes.

---

## Phase I · Outside

### 01 · Outside
**Asks** Who calls this, at what volume, and what can they tolerate — and what is this organisation actually able to operate, host and afford?

**Demand.** Per client class: identity and origin (first-party app, partner system, internal service, machine, scheduled job) · trust zone · volume and shape (read/write mix, average and peak, burst profile) · tolerances, one line each (latency percentile, ordering scoped to a key, durability, accuracy) · devices and networks · geography and regime · adversaries · your own dependencies, which hand you a quota rather than accept one.

**Every client-visible promise is declared here, and nowhere else.** Two of them are routinely deferred and must not be:

- **The views this class reads, each with its own freshness bound** — one bound per view, not one per client. A buyer tolerates a five-minute-old recommendation and a two-second-old basket. Flattening those is how every read ends up on the strict store. The bound is the client's to state; step 03 decides only what it implies.
- **What this class is still handed when a dependency is down** — chosen from six client-visible outcomes in rising severity: correct but slower · correct but less · stale but complete · pending but signalled · absent but signalled · **wrong, which is never permitted.** Say which are accepted, how often, and what each costs them. Step 07 prices and proves these; it does not author them.

Every tolerance names the conditions it holds under. "p99 200 ms" and "p99 200 ms during the nightly partner window" are different requirements, and only the second is testable.

**Supply.** Team capability and size · hosting posture (cloud, on-prem, hybrid, regulated enclave) · contracts already held (CDN, archive tier, managed services) · cost envelope · expected lifecycle (spike · one-off product · prototype becoming platform · platform).

**Deliverable** A client-class table carrying tolerances, the view list with a freshness bound each, and the accepted degraded outcomes — plus an operating-envelope table. These are the **joint-signature artifacts**: everything the business half of the organisation can read, check and correct lives here, and nothing client-visible is decided after this step.

**Exit condition — no figure leaves this step unconsumed.** Beside each volume, name the later step that will divide, bound or decide with it. A figure with no named consumer is a question that should not have been asked: drop it, or find the decision it was standing in for. Checked again at 02, reported at 10.

### 02 · Load
**Asks** What does the stated demand actually amount to?

Requests/second (average and peak, with the peak factor stated) · bytes/day and bytes/year · working-set size · fan-out · growth over the horizon you are designing for.

**Deliverable** The demand arithmetic, shown as division, with every input labelled measured / contractual / assumed. **No technology named yet.**

Every figure elicited at 01 is either divided here or explicitly carried to a named later step. One still unconsumed at 10 is a finding against 01, not against this step.

---

## Phase II · Shape

### 03 · Data
**Asks** What is true, what is derived from it, and what must never break?

Take the step-01 view list and its bounds — they are promises and are not revised here. Resolve each into truth or projection: a view whose bound is looser than the write path is a projection. Then, per data class: which acknowledgement kind writes need (receipt / reservation / completion) · invariants that no view can tell you about · conflict policy · read-your-writes seams · commitment reads (views a client commits against, which cannot be arbitrarily stale).

**Deliverable** A data-class table. This is where most of the design is actually decided.

### 04 · Storage & transport
**Asks** Which store families and channel guarantees do the data classes require — and how many runnable units?

Match store family to access pattern. Declare all six channel properties per edge. Then granularity: runtime units, module seams, repo topology — each split citing a force.

**Deliverable** The labelled graph: every node typed, every edge annotated.

**Record the ratio beside the citation.** Per choice: what the citing requirement demands against what the mechanism supplies. Near 1 is sized to the job; tens are headroom and must name the factor — growth, peak, failover; hundreds mean the requirement cannot discriminate the choice, and the choice came from somewhere other than this derivation.

### 05 · Trust
**Asks** Where are the boundaries, and what is enforced at each?

Authn/authz per surface and at which hop · data classification · delegated authority · what is enforced versus merely checked · quota as policy.

**Deliverable** Trust boundaries marked on the graph, with the enforcement point named per invariant.

---

## Phase III · Behaviour

### 06 · Derivation
**Asks** What computes what, and how do partial results compose?

Per processor: stateless / stateful / batch · inputs and outputs · merge semantics when combining across shards, hosts or windows · error bounds if it approximates · model processors (inference) sized like any other.

**Deliverable** Processor table with composition rules. Merge semantics are where correctness quietly dies — combining per-host top-K does not give global top-K.

### 07 · Failure
**Asks** What does each client class still get when a dependency is down?

1. Bring the promises forward from step 01 unchanged — the tolerances and the accepted degraded outcomes are already on the table. Do not re-ask.
2. Prove each promised rung is **reachable on purpose** — a flag, a breaker, a shed rule — and say when it was last exercised. A ladder never exercised is fiction.
3. Mechanics per edge — timeouts, retries with jitter, circuit breakers, bulkheads, fallbacks, dead-letter handling. Every mechanic traces to a rung a client asked for, or it goes.

**Deliverable** A reachability proof per promised rung, and the per-edge failure spec. The ladder is carried forward, not authored here.

### 08 · Runtime & placement
**Asks** How many of what, where?

Instances, regions, placement, data residency, and the **second arithmetic pass**: capacity, given the stores chosen at 04.

**Deliverable** Capacity arithmetic with headroom stated.

---

## Phase IV · Carry

### 09 · Operations
**Asks** How do we know it is working, how does it change, and what does it cost?

Detection proportioned to consequence — how badly a client is hurt, and where on the ladder that lands. Outcome KPIs in business terms (successful vs abandoned checkouts, not CPU). **Zero-tolerance counts**: promises with no acceptable rate, tracked as counts that must be zero. Release and delivery shape sized by lifecycle. Cost envelope carried from step 01.

**Deliverable** The operations table: what is watched, at what threshold, who is woken, and what it costs.

### 10 · Trace
**Asks** Does every mechanism cite a requirement, and every requirement map to a mechanism?

**Deliverable** The traceability map. Unmapped requirement → a gap; design for it. Unconsumed figure → a number collected at 01 that nothing here consumes; drop it, or name the decision it should have informed.

**A mechanism citing no client constraint takes the first disposition that applies — a search, not a menu.** ① **Delete** — nothing sits above it. ② **Surface a missing requirement** — tracing it lands on a client constraint that was always true and never written down; the common case, and often a *different* constraint from the one that bought the parent. ③ **Declare self-derived, naming the parent decision** — and the parent now faces the same test. Naming a parent moves the obligation up, it does not discharge it. Citations chain; the recursion ends at a client constraint or the finding sits at the root, and **a five-deep chain on an ungrounded adjective is one finding, not five.** Record the chain's root, depth and the link at which it stops tracing.

**Re-run trigger.** A zero-tolerance count that keeps firing is not an ops problem — it is a signal to re-run the derivation from step 03, because the source of truth was modelled wrong.
