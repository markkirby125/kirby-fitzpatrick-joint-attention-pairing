# Joint Attention Pairing — Technical Operational Dispatcher

**Framework Author**: William Fitzpatrick (*Writer Science*)  
**Source Lecture**: [Write 10X Clearer: Do These 3 Writing Exercises to Connect with Readers](https://www.youtube.com/watch?v=UqiEjaCiWmQ)  
**Parent Collection**: [Master Collection](../../kirby-fitzpatrick-writers-collection/SKILL.md) | [Global Help](../../../kirby-help/SKILL.md)  

---

## 1. Cognitive Foundation: Two Private Simulations and the Object That Is Not in the Room

Debugging is not a solitary act of reasoning. It is a dyadic act of *looking at the same thing*. Human communication of concrete fact — as opposed to abstraction or poetry — runs on **triadic attention**: two agents, one object, and mutual knowledge that both parties are attending to that object. Developmental psychology calls this a **joint attentional frame**, and its defining property is not that two people happen to face the same thing. It is that each party knows the other is facing it too. Once the frame exists, language has almost no work left to do: a handful of words plus a point transfers a fully specified state. When the frame is missing, language must *manufacture* the object from scratch, and every word in the sentence becomes simultaneously ambiguous. Clarity is not a property of the sentence. It is a property of the *triad*.

Software engineering is the domain where the missing vertex costs the most, for three structural reasons:

1. **The objects are addresses, not concepts.** The referent of a debugging claim is never "the cache" — it is a specific read at a specific offset of a specific revision of a mutable tree. Conceptual reference is insufficient here; the claim is only true or false *at a revision*.
2. **The pair is never colocated.** There is no shared screen, no cursor to stab at, no terminal the other party can see. The transcript *is* the perceptual field.
3. **One member of the pair may have no perception at all.** When the listener is an agent, model, indexer, or CI job, there is no repair loop: no hesitation, no confused silence, no "wait, which one?" — just a silent substitution of the nearest plausible object.

That third property is what makes this a hard engineering constraint rather than a style preference. A human reader who cannot resolve a referent *feels* the friction and asks. A machine reader resolves it to the statistically nearest shape and proceeds. The failure is not a lost afternoon; it is a wrong mutation executed confidently against the wrong object.

```text
[ANTI-PATTERN: two private simulations, zero shared objects]

   author                                    reader
     │                                          │
     │  ┌────────────────────────┐              │  ┌────────────────────────┐
     │  │ "the cache is stale"   │              │  │ "the cache" → ?        │
     │  │ "this retries too much"│  ── words ──▶│  │ "this"     → ?         │
     │  │ "the bug in the parser"│   only       │  │ "the bug"  → nearest   │
     │  └────────────────────────┘              │  └────────────────────────┘
     │            ▲                             │            ▲
     └────────────┼─────────────────────────────┴────────────┼──────────────
                  │                                          │
            [ no object ]                              [ no object ]
   The sentence is the entire channel. Nothing in the exchange can be checked
   by either party, so a divergence is invisible until the fix fails.

   Boundary behaviour of the anti-pattern
     A0  same screen, author's cursor present  → works; the pointer is doing the work
     A1  async thread, "the retry logic"       → reader guesses; guess is usually near
     A2  agent session, "fix the parser bug"   → agent invents a parser and edits it
     A3  reader six months later, tree mutated → referents resolve to the wrong lines
     A4  machine reader (CI, indexer, model)   → no repair loop; silent substitution
```

```text
[PROTOCOL: one triadic frame, one addressable object]

   author                     the object                     reader
     │                   ┌─────────────────────────┐           │
     └───── point ───────▶│ src/reconciler.go       │◀─── point ─┘
                         │   :212  (lastSync read) │
                         │ @ 9f21c0d               │
                         │ $ go test -run Reconcile│
                         │   --- FAIL (3.140s)     │
                         └─────────────────────────┘
                                    ▲
                        mutual knowledge of shared focus
                    (both parties say which object they hold)

   Every claim in the conversation is now decidable by either party, at any
   time, without asking the other one. Gaze converges before the argument starts.

   The triad, and what happens when a vertex is removed
     author ── object ── reader      joint attention   ✔ claims are checkable
     author ── reader                mere co-presence  ✘ politeness, no resolution
     reader ── object                unpaired gaze     ✘ reader explores alone
     author ── object                monologue         ✘ the log nobody reads
```

The lecture's teaching target — three exercises that force connection with the reader rather than performance for an imaginary audience — translates into exactly one engineering obligation: **manufacture the shared object explicitly, because the shared perceptual field that would have supplied it for free does not exist here.** Fitzpatrick's exercises all push the same muscle: stop inventing a reader who already holds your mental model, and start doing the work of placing the object where both of you can see it. Operationalized for engineering, those three exercises are the *point*, the *frame*, and the *confirmation* (§2.4).

The mechanism has one invariant:

> **The Joint Attention Invariant**: every claim about system behaviour in a collaborative debugging exchange must resolve, for *both* parties, to a verifiable address at a known revision — or it must be explicitly marked unresolvable and treated as a defect in the exchange, never as a claim.

Seven load-bearing definitions:

1. **Joint Attentional Frame (JAF)** — the triadic state in which two agents attend to one object and each knows the other does. Not "both looked at the file": both *know* they are looking at the same thing, and can say so.
2. **Referent Address** — a resolvable, revision-pinned locator for the object: `path:line`, `sha:path:line`, a test node ID, a symbol signature, a log line with its timestamp, a trace span, a fixture path.
3. **Deictic Anchor** — the token in prose that performs the point: the line number, the SHA, the command, the pasted output with its exit code. It is not decoration or citation etiquette; it *is* the channel. Delete the anchor and the remaining words carry no information.
4. **Fictive Pointing** — pointing at an object that does not yet exist (a proposed API, an unimplemented branch, a "we should"). Legitimate, and dangerous: an unmarked fictive anchor is resolved by the reader to the nearest existing thing with a similar name.
5. **Anchor Drift** — the anchor outliving its object. Line numbers shift under any edit above them; SHAs disappear under rebase; pasted output loses its revision. Joint attention is a *moment*, not a durable state, and the anchor is mutated by the very change it justified.
6. **Gaze Convergence Time (GCT)** — seconds until both parties resolve the same address. The one measurable quantity in this dispatcher. Unresolved referents make it unbounded.
7. **Common Ground Ledger** — the enumerated set of objects the pair has jointly established. Each turn may add anchors or reuse ones already on the ledger; an anchor that is neither added nor reused is an orphan claim.

Why this outranks ordinary documentation hygiene: unresolved referents are *cheap to write and expensive to hold*. Each one forces the reader to build a private simulation, and two private simulations diverge. The divergence cost is not paid at comprehension time — it is paid at the second failure, under incident pressure, by a reader who now believes they understand the system and does not.

---

## 2. Core Transformation Protocols

1. **Point before you predicate.** A claim about system behaviour is preceded by its address, or it is not written. *"The cache is stale"* is not a finding; *"`9f21c0d` `cache.go:88`: `lastSync` is read before `mutex.Lock()`, so a concurrent writer can repopulate the entry"* is a finding.
2. **One anchor per assertion, not per paragraph.** A sentence containing three claims needs three addresses. Anchoring the paragraph licenses the reader to attach one object to all three clauses.
3. **Anchor to the most stable address available.** Preference order: `sha:symbol@path:line` → `path:symbol` → `path:line @ sha` → `path` → nothing (forbidden). Line numbers are the *least* stable anchor class, so never ship one alone.
4. **Ship the command with the object.** An anchor tells the reader *where* to look; the command tells them *how to reproduce the look*. `go test -run TestReconcileFlushWatermark ./internal/reconcile -count=1` converts an assertion into a re-runnable instrument.
5. **Pin or quarantine pasted output.** Every pasted log, stack trace, or benchmark carries its revision, command, and platform, or is explicitly labelled `unpinned`. Unpinned output is the most authoritative-looking unresolvable referent in existence.
6. **Ban the bare demonstrative for technical referents.** *this*, *that*, *it*, *the above*, *the previous behaviour*, *the old config* → the symbol, the path, or the commit. A pronoun in a debugging walkthrough is a promise that the referent lives in the reader's head.
7. **Use canonical identifiers, not roles.** `batcher.flush()` not "the flush logic"; `TestRetryCapBoundsCallerDeadline` not "the retry test"; `internal/reconcile.Policy` not "the policy thing". Roles are how referents become multiple.
8. **Narrow the frame before walking through it.** Joint attention has a capacity: establish *one* entry point (one file, one function, one failing test), keep the gaze there for the step, then sequence to the next object. A walkthrough that touches nine files in one paragraph has no object at all.
9. **Mark fictive anchors explicitly.** `[proposed]`, `[not yet implemented]`, `[to be created]`, paired with the prototype, fixture, or interface stub that makes the future object inspectable *today*. Unmarked fictive pointing is the single largest source of wrong-object edits.
10. **Re-point after every mutation.** The anchor is invalidated by the change it justified. When line numbers move, state the new anchor and the revision it was valid at: *anchors valid at `9f21c0d`; after this diff, `reconciler.go:214`.*
11. **Acknowledge the point when you receive one (the mutuality clause).** Joint attention requires each party to know the other is looking at the same object. In an agent or async session, echo the resolved anchor before acting: *"anchored to `reconciler.go:212` at `9f21c0d`, confirmed by the failing assertion on line 214."* An unacknowledged point is a monologue wearing a diagram.
12. **Fail closed on unresolvable anchors.** If an anchor does not resolve, say so and stop; never substitute a plausible object. *"`reconciler.go:212` is not present at HEAD; at HEAD, line 212 is the metrics emit — I cannot reproduce the claim"* is a correct and complete response. A silent pass is a defect.
13. **Convert pronouns at every turn boundary.** Drift accumulates per sentence, not per paragraph, and each new author turn resets the reader's gaze. Re-point at the start of each reply, not once at the top of the thread.
14. **Sequence objects; do not stack them.** One shared object per step. Two anchors competing inside one sentence split the frame, and the reader resolves to whichever they checked out last.
15. **Measure the pairing.** Record GCT (time from anchor published to both parties confirming the same object) and repair count (referents the reader had to ask about or guess). Both are gate metrics for critical-path walkthroughs; zero repairs is the required budget for runbook-adjacent writing.

### 2.1 Anchor classes and their decay behaviour

| Anchor class | Example | Resolves for | Degrades when | Failure mode |
|---|---|---|---|---|
| Deictic (line) | `reconciler.go:212` | anyone with the tree, same revision | any edit above the line | silently points at the wrong statement |
| Revision-pinned | `9f21c0d:reconciler.go:212` | humans and tools, durably | force-push, GC after history rewrite | loud failure: resolves to nothing |
| Symbolic | `reconcile.Policy.Apply()` | searches, IDEs, indexers | symbol renamed, or duplicated per package | ambiguous anchor; multiple objects |
| Executable | `go test -run TestReconcileFlush -count=1` | anyone with the toolchain | flaky test, drifting fixtures | two parties get different outputs |
| Observational | pasted stderr + exit code 1 | anyone | unpinned revision or command; truncated paste | looks authoritative; unfalsifiable |
| Artifact | `trace.json`, `cpu.pprof`, heap dump | tooling and humans | artifact deleted, link expires | dead pointer, usually noticed late |
| Fictive | `[proposed] RetryPolicy.Budget()` | nobody, yet | left unmarked | reader resolves to nearest existing shape |

### 2.2 Transformation table: anti-patterns and clean replacements

| Anti-pattern | Why joint attention breaks | Clean replacement |
|---|---|---|
| *"The cache is stale."* | Names a concept, not a read; the reader picks their own cache | `` `9f21c0d` `cache.go:88` — `lastSync` is read before `mutex.Lock()`; concurrent writer repopulates between read and use `` |
| *"This retries too much."* | *this* has no referent; "too much" has no budget | `retry.go:41–57`: 3 attempts × 400 ms backoff = 1.6 s worst case, exceeding the caller's 2 s deadline at `gateway/timeouts.go:14` |
| *"The test fails intermittently."* | No test ID, no run count, no revision | `TestReconcileFlushWatermark` fails ~1 in 20: `go test -run TestReconcileFlushWatermark ./internal/reconcile -count=20` @ `9f21c0d` → 2 failures, both with `watermark=0` |
| *"See the logs."* | No stream, no filter, no notion of normal | `kubectl -n payments logs -l app=reconciler --since=10m` → filter `reconcile:`; expect one `scanned=<n>` line/min |
| *"Fixed the bug we discussed."* | The object lives in a conversation, not the artifact | `ISSUE-1043`; mechanism at `batcher.go:88`; verified by `make bench-batcher WORKLOAD=burst-4k RUNS=5` |
| *"It's faster now."* | No metric, no baseline, no revision | p99 write lag `3,140 ms → 610 ms`, workload `bench-recon-100k`, baseline `main @ 9f21c0d`, 5 runs, ±6 ms |
| *"Follow the usual setup."* | The procedure is habit, not an address | Numbered steps + `scripts/dev-setup.sh:1–40` + expected output per step |
| *"There's probably a race here."* | Speculation presented as an anchor | `[hypothesis] reconciler.go:212`: read-before-lock; test: `-race -count=50`; if no failure in 50 runs, hypothesis is unsupported |
| *"We'll add a retry budget."* | Unmarked fictive point; reader edits the nearest retry code | `[proposed] RetryPolicy.Budget()` — stub at `policy_stub.go:12`; existing `retry.go:41` is *not* the object and must not be edited |
| `// see above` | The referent is a scroll position | The symbol, the path, or the commit SHA |
| A 400-line pasted log | Shared *text*, not shared attention; no landing point | Quote the 3 lines that carry the claim, with command, revision, exit code |
| *"Ask Dave about the watermark."* | Human oracle; unresolvable at 03:00 and for machines | The invariant in the comment, plus the test that enforces it at `batcher_test.go:88` |
| *"LGTM 👍"* on an anchored review | Confirms nothing; mutuality was never established | Echo the resolved object: *confirmed `batcher.go:88` @ PR head; budget assertion at `batcher_test.go:88` holds* |

### 2.3 Failure diagnostics

| Symptom | Diagnosis | Fix |
|---|---|---|
| Agent edits a file nobody named | Fictive anchor resolved to the nearest shape | Mark `[proposed]`/`[not yet implemented]`; supply stub or fixture |
| Reviewer asks a question the comment already "answered" | Deictic pointer with no address | Replace the pronoun with `path:line @ sha` and the reproducing command |
| Fix works on the author's branch, not on yours | Anchor implied a revision that was never stated | Pin the anchor: `sha:path:line`; state the revision in the claim |
| Two engineers describe the same bug differently | Neither anchor resolves for the other; two private objects | Reduce to one address, one command, one output; agree on the object first |
| Line number points at the wrong statement | Anchor drift after an edit above it | Re-derive anchors post-change; pair line numbers with symbol names |
| Pasted log is authoritative but unverifiable | Unpinned observational anchor | Add command, revision, environment, exit code, or label `unpinned` |
| Agent "fixes" a passing test | Object was a test *name*, not a test *node ID* | Anchor to `Package/TestName` and pin the revision |
| Walkthrough reads clearly but nothing is actionable | No frame was established; gaze scattered across many objects | Narrow to one entry point, sequence objects, re-point at each step |
| Review thread converges in comments, never in the diff | Mutuality established in conversation, not in the artifact | Land the confirmed anchor in the artifact (comment, test name, PR body) |

### 2.4 The three exercises, operationalized as probes

Fitzpatrick's three exercises share one target: forcing the writer to establish a shared object *before* making claims, instead of performing for a reader who already holds the writer's mental model. In engineering, they run as three probes over any draft — a comment, a PR body, a debug walkthrough, an RFC.

| Exercise | Probe | Pass condition |
|---|---|---|
| **1. Point** | Delete every pronoun and demonstrative; require an address in each slot. Count the slots you cannot fill. | Zero unfillable slots; every claim carries an anchor class from §2.1 |
| **2. Frame** | Name the single object of the walkthrough. Check each sentence's gaze resolves to it or to a sequenced next object. | Exactly one object per step; no orphan objects; explicit sequencing between steps |
| **3. Confirm** | Hand the anchored text to a reader or agent with no context; require them to state the object back, unprompted. | The stated object matches the intended one, and the reproducing command runs |

Three orthogonal probes run alongside them:

- **Mutation probe** — apply an unrelated edit above the anchor. Does it still resolve to the intended statement, or does it now point somewhere plausible and wrong? Any anchor that survives a silent mutation needs a symbolic co-anchor.
- **Machine-reader probe** — strip the page from the reader (feed only the text). Does the claim still resolve? A machine reader will not ask.
- **Latency probe** — time-to-shared-object for a cold reader with repository access but no author contact. Unbounded GCT means the anchor count is zero in practice, whatever the prose says.

**Related dispatchers.** Audit the anchored text as a zero-context stranger with the [Cold Reader PR Auditor](../../kirby-fitzpatrick-cold-reader-pr-auditor/SKILL.md); ensure the artifact outlives its author with the [Silent Author Test](../../kirby-fitzpatrick-silent-author-test/SKILL.md); strip pronouns, filler, and restated code with the [Lexical Anti-Bloat Filter](../../kirby-fitzpatrick-lexical-anti-bloat-filter/SKILL.md) and the [Anti-Author-Splain Commenter](../../kirby-fitzpatrick-anti-author-splain-commenter/SKILL.md); make the anchored structure discoverable at skim speed with [Cathedral Taxonomy](../../kirby-fitzpatrick-cathedral-taxonomy/SKILL.md) and the [Skim-Test Outliner](../../kirby-fitzpatrick-skim-test-outliner/SKILL.md); establish the entry point before narrowing with [Bilbo Simple-to-Complex](../../kirby-fitzpatrick-bilbo-simple-to-complex/SKILL.md) and the [Uneven U Explainer](../../kirby-fitzpatrick-uneven-u-explainer/SKILL.md); lead with the decision the reader must make using [Here's Why Inversion](../../kirby-fitzpatrick-heres-why-inversion/SKILL.md); keep the anchor as the relay baton across sentences with [Linear Relay Linking](../../kirby-fitzpatrick-linear-relay-linking/SKILL.md); and convert the anchored exchange into a durable disposition with the [Letter of Response Reviewer](../../kirby-fitzpatrick-letter-of-response-reviewer/SKILL.md).

---

## 3. Engineering Application Scenarios

### 3.1 Code Reviews — one comment, one object

A review comment that names a concept rather than a line is a request for the author to reconstruct the reviewer's private object. It reads as polite and costs both parties a round trip. The anchored form is shorter and settles the claim on first read.

```text
BEFORE — concept-level, unresolvable, invites a thread:

  "I think the retry logic here is going to blow the caller's budget in
   the worst case. Can we take another look?"

  Reader of this comment holds: "the retry logic" (which one?), "here"
  (which hunk?), "the budget" (whose?), "worst case" (measured?).

AFTER — one triadic frame, three anchors, zero round trips:

  "At retry.go:41–57 @ 9f21c0d the attempt cap is 3 with 400 ms backoff
   and 2 × 200 ms jitter = 1.6 s worst case, which exceeds the caller's
   deadline set at gateway/timeouts.go:14 (2 s) before the first sleep.

   Evidence: go test -run TestRetryCapBoundsCallerDeadline ./internal/retry -count=1
     → --- FAIL: TestRetryCapBoundsCallerDeadline (0.42s)
          budget_test.go:88: worst case 1.6s >= caller budget 2s? false ... 1.6s < 2s passes; crossing at attempt 4

   Claim: raising maxAttempts converts a fast failure into a caller-visible
   timeout rather than raising the success rate.

   Confirm: I hold retry.go:41–57 @ 9f21c0d. Reply with the object you hold
   if it differs."
```

Three rules make this work, and each maps to a protocol above:

1. **Anchor every clause.** Three claims (cap value, arithmetic, deadline source) need three addresses: `retry.go:41–57`, the computed sum, `gateway/timeouts.go:14`. One anchor for all three licenses the reader to assume one object.
2. **Ship the instrument.** The failing test node ID plus its `-run` command is what turns a reviewer's opinion into a checkable proposition. Reviewers who can run the claim do not argue about it.
3. **Ask for the object back.** The closing line is the mutuality clause (§2, rule 11). It converts a monologue into a pairing, and it surfaces a divergence *before* the author starts editing.

When the finding is a *pattern* rather than a line — a naming convention, a repeated misuse — anchor to the best instance, then state the scope with a command rather than an adjective:

```text
  "Instance: reconciler.go:212 @ 9f21c0d (read-before-lock on lastSync).
   Scope: rg -n 'lastSync' internal/ → 7 sites; 3 of them read before
   acquiring Policy.mu (list at reconciler.go:212, worker.go:88, worker.go:141).
   The other 4 are inside the lock and are not the object."
```

*"This pattern is everywhere"* is unresolvable. `rg -n 'lastSync' internal/` → 7 sites, 3 in scope is a set both parties can count.

### 3.2 PR Descriptions — the failing output is the shared object

A PR body is read at rollback time, by a stranger, with no access to the conversation that produced it. The prime move is therefore to establish the frame at the top: **the one test that was failing and now passes**, pinned to revisions, with its output reproduced. Everything downstream — rationale, blast radius, rollback — hangs off that object.

```markdown
# Watermark-bounded write batching

## The object
`TestReconcileFlushWatermark` @ `9f21c0d` — failing on `main`, passing at PR head.
Command: `go test -run TestReconcileFlushWatermark ./internal/reconcile -count=1`

before (main @ 9f21c0d)              after (PR head, 3a77e02)
  --- FAIL: (4.81s)                    ok (0.63s)
  watermark_test.go:41:                watermark_test.go:41:
    peak rows buffered = 12,480         peak rows buffered = 512
    want <= 512                         want <= 512

## What changed
`batcher.go:88` — flush triggers on `min(interval, 512 records)` instead of the
ticker alone. The flush trigger IS the durability contract: a time-only trigger
bounds latency in seconds and leaves record count unbounded.

## Why this object and not the throughput numbers
The failing assertion is the invariant. Throughput figures are support, below.

## Evidence
`make bench-batcher WORKLOAD=burst-4k RUNS=5` · main @ 9f21c0d → PR head 3a77e02

| Metric | main @ 9f21c0d | PR head 3a77e02 | Δ |
|---|---|---|---|
| p99 write lag | 3,140 ms | 610 ms | −81% |
| peak rows buffered | 12,480 | 512 | −96% |
| write txn / 10k rows | 42 | 59 | +40% (expected) |

## Blast radius and rollback
Write path of `reconciler` only; readers see smaller batches, never partial ones.
Rollback: `git revert 3a77e02`. Startup banner `batch_flush_watermark=512` disappears
when the revert is live.

## Does not cover
Compaction scheduling and the read path. The watermark is not a throughput dial.

## Anchors valid at
3a77e02. Line numbers in `batcher.go` shift under the compaction change tracked in
ISSUE-2211; re-derive the anchor after that lands.
```

The two clauses that stop the PR from decaying: **"Anchors valid at"** (rule 10 — the anchor is invalidated by the change it justified) and **"Does not cover"** (bounded scope, so a reader does not extend the decision past what was examined). A PR body that reproduces the diff has produced shared *text* without shared *attention*: the reader still has no landing point for their gaze.

### 3.3 Architecture RFCs / ADRs — pointing at objects that do not exist yet

This is the hard case. In a review, the object exists and can be pointed at. In an RFC, the object of the argument is *the system after the change*, which has no address. The failure mode is structural: fictive anchors get resolved by readers to the nearest existing shapes with similar names, so the RFC is debated as though it described the current code — and the eventual implementation lands somewhere nobody agreed on.

The protocol for this case, in order:

1. **Anchor the motivation in the present tense.** Every future claim is grounded in a *present* object the reader can inspect today: the line that does the wrong thing, the trace that shows the failure, the measured baseline, the incident. *"Reads happen without the lock at `reconciler.go:212`"* is a present-tense address; *"the design is slow"* is a judgment with no object.
2. **Mark every fictive anchor explicitly.** `[proposed]`, `[not yet implemented]`, `[to be created]`, each paired with the stub, interface, fixture, or prototype that makes the future object inspectable *today*. Marking is not stylistic — it is the only thing preventing a wrong-object edit.
3. **Publish the falsification command with the decision.** A decision without a command that would prove it wrong cannot be re-evaluated by a silent maintainer. State the signal, its threshold, and the role that watches it.
4. **Bind future claims to a scale band.** *Valid for datasets ≤ 10M rows per tenant, single-writer partition assignment.* Outside the band the decision is not wrong — it is unexamined, and the RFC must say so.

| ADR section | Joint-attention obligation |
|---|---|
| Context | The present-tense object: the invariant at stake, the line or trace that violates it, with addresses |
| Decision | One falsifiable commitment a reader can match against the code |
| Preconditions | Scale band, topology, versions, flag state — the frame inside which the decision holds |
| Evidence | Commands, baselines, revisions, output — the object as *executable* anchor |
| Fictive inventory | Every not-yet-existing object named, marked, and paired with its stub or fixture |
| Reversal trigger | The signal that proves the decision wrong, its threshold, and the watching role |
| Re-evaluation | Interval, owner role, and the artifact that forces the check (upgrade, incident review, load test) |
| Does not cover | Explicit outer bounds, so no reader extends the decision into unexamined territory |
| Anchors valid at | Revision of the present-tense anchors, plus the trigger that invalidates them |

```markdown
# ADR-021 — Watermark-bounded write batching

## Context (present tense, anchored)
`reconciler.go:212` @ 9f21c0d flushes on the ticker alone. A flush trigger IS the
durability contract: a time-only trigger bounds latency in seconds and leaves
record count unbounded. Observed at `9f21c0d`: peak 12,480 rows buffered (trace
`flaky-flush-4k.json`, 41 s window).

## Decision (falsifiable)
Flush on `min(interval, 512 records)` in `batcher.go:88` (PR 3a77e02). Match: the
startup banner reports `batch_flush_watermark=512`.

## Preconditions
Single-writer partition assignment; datasets ≤ 10M rows per tenant. NOT valid under
multi-writer assignment — the watermark then bounds one writer's buffer, not the
dataset's in-flight records. [not yet implemented] multi-writer support is tracked
in ISSUE-2211; this ADR must be revised before that lands.

## Evidence (executable anchor)
`make bench-batcher WORKLOAD=burst-4k RUNS=5` · 9f21c0d → 3a77e02 ·
p99 lag 3,140 → 610 ms · peak rows 12,480 → 512 · txn/row +40% (expected price).

## Fictive inventory
[proposed] `BatchPolicy.Watermark()` — stub `policy_stub.go:12` (compiles, panics on call).
[proposed] `reconciler` per-tenant watermark override — no object exists; not covered here.

## Reversal trigger
`write_txn_per_row` > 1.5× baseline for one week. Watched by the storage on-call
(rotation `storage-primary`); reviewed quarterly by the platform role, not the author.

## Does not cover
Compaction scheduling and the read path. The watermark is not a throughput dial.

## Anchors valid at
9f21c0d (context, evidence). `reconciler.go` line numbers shift under ISSUE-2211.
```

The `[not yet implemented]` marker on multi-writer support is doing real work: without it, a reader implementing multi-writer will assume the watermark already covers their case and will edit the single-writer code — the nearest existing shape. Anchored, the same reader stops and opens ISSUE-2211.

---

## 4. Verification Checklist

- [ ] **Every behavioral claim carries an address.** Each assertion about system behaviour resolves to an anchor from §2.1 — `sha:path:line`, symbol, test node ID, command, or pinned output — with no bare demonstrative (`this`, `that`, `it`, `the above`, `the old`) standing in for a technical referent anywhere in the text; unfillable slots were counted and eliminated, not softened.
- [ ] **One object per step, explicitly sequenced.** The walkthrough names its single entry point, keeps the gaze there for the step, sequences to the next object by name, and never leaves an anchor orphaned by a paragraph-level anchor that licenses several claims.
- [ ] **Anchors are revision-pinned and mutation-surviving.** Line anchors ship with their symbol and commit; pasted output carries its command, revision, and exit code (or is labelled `unpinned`); and an unrelated edit above any anchor leaves it resolving to the intended statement rather than to a plausible neighbour.
- [ ] **Fictive pointers are marked and made inspectable.** Every not-yet-existing object is prefixed `[proposed]` / `[not yet implemented]` / `[to be created]` and paired with a stub, fixture, prototype, or interface that lets a reader inspect it today — so no reader resolves it to the nearest existing shape and edits that instead.
- [ ] **The pairing was confirmed and measured.** A reader or agent with repository access and no author contact stated the object back unprompted and reproduced it from the shipped command; gaze convergence time and repair count were recorded (required budget for critical-path walkthroughs: zero repairs), and any mismatch found was fixed in the anchor, never in the reader.