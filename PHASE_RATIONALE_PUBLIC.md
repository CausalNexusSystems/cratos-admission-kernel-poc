# Phase Rationale — Plain-Language "Why" (Public)

**This document is commentary, not evidence.** It is not sealed, it is not
part of any Merkle root, and it carries no cryptographic weight of its own.
Its only purpose is to explain, in plain language and without any CRATOS
source code, *why* each phase's `PASS` result is the correct, by-design
engineering outcome — not a workaround, not a lucky pass, and not an
omission. Every claim below is a restatement of language already present,
verbatim or near-verbatim, in each run's own `CRATOS_REPORT.md` — nothing
here is new sensitive information. To verify a phase's result yourself, use
that phase's own **native seal**, shown directly in `CRATOS_REPORT.md` /
`CRATOS_REPORT.txt` under each phase's `Result: PASS` line, and recompute it
per the instructions printed immediately below it. The two seals for each
phase (one per run) are reproduced below purely for convenient
cross-reference — the authoritative copy is always the one inside the
run's own report.

Eighteen phases run in every CRATOS ARP execution, grouped into three
categories: **fixed regression** (phases 0–7, deterministic, hardcoded
inputs, no randomness at all), **randomized** (phases 8–13, PRNG-varied
adversarial inputs at scale), and **declared boundary** (phases 14–17, new
in this release, PRNG-varied inputs probing boundaries the kernel states
explicitly rather than silently assumes). The PRNG in every randomized or
declared-boundary phase decides *which test cases* get tried; it is never
passed to the kernel and never touches the kernel's own decision logic.

Run legend used below:
- **R1** = `CRATOS_ARP_20260904T014323Z` (outbound network route reachable at run start)
- **R2** = `CRATOS_ARP_20260904T014512Z` (no outbound network route at run start)

---

## Fixed regression (phases 0–7)

### Phase 0 — SignedAuthorization provenance
**What was tested:** that the kernel only accepts an authorization whose
signature genuinely traces back to the claimed issuer, with fixed,
hardcoded valid and invalid cases.
**Why PASS is correct:** provenance checking is a precondition, not a
best-effort filter — an authorization that doesn't verify must never reach
the admission logic. A PASS here means that boundary held on every
hardcoded case tried, with no exceptions.
**Seals:** R1 `daec3de7e81b5776929c8e3bbca427ff544170f5809c5c4ca60f25ef668b8b59` · R2 identical input, seal recomputed independently in `CRATOS_REPORT.md` of R2 (fixed-input phases produce identical seals across runs since nothing PRNG-varied feeds them).

### Phase 1 — core admit/release/replay/overflow (signed path)
**What was tested:** the four core admission behaviors together — admitting
a valid request, releasing occupancy correctly, rejecting a replayed
sequence number, and rejecting a request that would overflow the budget —
all through the signed proposal path.
**Why PASS is correct:** these four behaviors are the kernel's baseline
contract. A PASS means each of the four fixed cases produced exactly the
outcome the contract specifies, with no case silently skipped.

### Phase 2 — widened effect taxonomy (8 abs + 6 occ)
**What was tested:** that every one of the 14 declared effect kinds (8
absolute-budget lanes, 6 occupancy lanes) is independently tracked and
independently enforced, not merged into a single generic counter.
**Why PASS is correct:** a taxonomy that silently collapsed two distinct
effect kinds into one shared budget would let one lane's spend starve or
mask another's. PASS means each of the 14 lanes held its own boundary
independently.

### Phase 3 — sealed snapshot / crash-restart durability
**What was tested:** that kernel state can be sealed to a snapshot,
restored after a simulated crash/restart, and that the restored kernel
enforces exactly the same boundaries as before the crash.
**Why PASS is correct:** a kernel whose guarantees reset or weaken across a
restart is not durable. PASS means the restored kernel's decisions matched
the pre-crash kernel's decisions bit for bit.

### Phase 4 — raw path: N threads racing a pre-anchored proposal
**What was tested:** many real OS threads simultaneously racing to submit
the same pre-anchored proposal through the kernel's lowest-level (raw)
entry point, with no serialization applied by the caller.
**Why PASS is correct:** exactly one thread should win the race and every
other thread should be correctly rejected as a replay or a resource
conflict — never both admitted, never both rejected, never a corrupted
intermediate state. PASS means the race resolved cleanly every time.

### Phase 5 — fix: same race via propose_and_evaluate
**What was tested:** the identical race from Phase 4, but through the
higher-level entry point intended for normal callers.
**Why PASS is correct:** this is the direct comparison case for Phase 4 —
it demonstrates the safe entry point resolves the same race with the same
guarantee, so callers using the intended API are not exposed to the raw
path's sharper edges.

### Phase 6 — no backdoor: forged proposal via raw path still fails
**What was tested:** whether a proposal forged to bypass normal
authorization can succeed just because it was submitted through the raw,
lower-level path instead of the normal one.
**Why PASS is correct:** a security boundary that only holds through one
entry point is not a boundary. PASS means the raw path enforces exactly the
same provenance and structural checks as the normal path — there is no
faster, less-checked route to admission.

### Phase 7 — v0.5: HighWaterMark stops a legitimate-but-stale snapshot replay
**What was tested:** whether a snapshot that was legitimately signed and
valid at an earlier point in time, but is now stale, can be replayed to
roll the kernel's state backward.
**Why PASS is correct:** a signature alone doesn't carry a notion of
"current" — an attacker who captures a valid-but-old snapshot must not be
able to reintroduce it later and roll back state. PASS means the
high-water-mark boundary rejected the stale replay.

---

## Randomized adversarial (phases 8–13)

### Phase 8 — causal-anchor race at scale: N real OS threads, randomized proposals, same captured anchor
**What was tested:** the Phase-4/5 race pattern but at scale, with many
real threads and PRNG-randomized proposal content, all anchored to the same
captured causal point.
**Why PASS is correct:** randomizing the proposal content (while keeping
the race structure) checks that the guarantee from phases 4/5 isn't an
artifact of one specific input — it must hold across a wide, unpredictable
input space, not just the hand-picked fixed case.
**Seals:** R1 `4995321322c2f79dfabff940bf3506d9edc173f4e645c1147293ed3f60f86865` · R2 recomputed independently, same structural outcome (see R2's `CRATOS_REPORT.md`).

### Phase 9 — fuzzing SignedAuthorization at scale: randomized valid/wrong-issuer/bit-flip/post-sign-tamper mix
**What was tested:** Phase 0's provenance boundary, fuzzed at scale — a
PRNG-controlled mix of genuinely valid authorizations, wrong-issuer
authorizations, single-bit-flip corruption, and post-signing tampering.
**Why PASS is correct:** a provenance check that only catches the exact
tamper patterns its author thought to test is fragile. PASS across a large
randomized mix is stronger evidence that the check is structural (verifying
the signature itself) rather than pattern-matching specific known-bad
inputs.
**Seals:** R1 `162be7ca588561f0a8b14348205d253c747ce481288540654692d352fdecdc4e` · R2 `77c17da6347d37be6c03792bf8783f3b3fdc8cbea90c393ca93c6945089d7bc0`

### Phase 10 — randomized cross-lane budget spillover: fund one absolute lane, attack a different one
**What was tested:** whether funding one effect lane (e.g. FileWrite) can
be exploited to admit spend against a different, unfunded lane (e.g.
FileDelete), with PRNG-chosen lane pairs and magnitudes each run.
**Why PASS is correct:** each lane's budget must be genuinely independent.
PASS across randomized lane pairs means no combination tried let budget
"leak" from one lane's ledger into another's.
**Seals:** R1 `3231ce5569bde7d98c94a8ba7fa9440584c0a1cc73d64d3e45439920c8b29742` · R2 `24523ec76ee8198b04f4dfd74ef8e38cc4c2bafb235b574a965a958a6df76b12`

### Phase 11 — randomized occupancy release: some legitimate, some attempting to release more than was held
**What was tested:** a PRNG-mixed sequence of legitimate occupancy releases
and hostile attempts to release more occupancy than was actually held.
**Why PASS is correct:** an occupancy ledger that can go negative or be
"over-released" no longer reflects reality and could let a later proposal
be wrongly admitted against phantom freed capacity. PASS means every
over-release attempt was rejected while every legitimate release succeeded.
**Seals:** R1 `808c8f3dc4ad01c5119e7000a11c4eeb859345b49f5129ba7a4d97941adba149` · R2 `b4d132403b32a6fdd7e482116f48354b2a47a8f1584a6a439be8ada08cf95c15`

### Phase 12 — insider-disclosure scenario: N concurrent agents across 10 independently-keyed illustrative sectors
**What was tested:** many concurrent agents — a mix of legitimate operators
and 5 distinct adversarial pressure techniques — acting simultaneously
across 10 independently-keyed illustrative sectors.
**Why PASS is correct:** this is the closest phase to a realistic multi-actor
deployment shape. PASS means that under concurrent, mixed-intent pressure
across independently-keyed contexts, every admission decision still
resolved correctly per-sector, with no cross-sector interference.
**Seals:** R1 `d953ea6fb5c70213fd7273dce65cf50a7e36eefbbc162116c1ad130b69a5797f` · R2 `008ee404c0861c7aa2a0673ec599d981952d07178eb6b95d33ef47e0c1cefaff`

### Phase 13 — embedder-fixed structural ceiling on one illustrative channel, tested against total compromise of its issuer private key
**What was tested:** whether a structural ceiling set by the embedder
(independent of what any signed authorization claims) still holds even in
the worst case: the issuer's own private key is assumed fully compromised.
**Why PASS is correct:** this is a deliberate defense-in-depth check — a
ceiling that only worked as long as key material stayed secret would not be
a real second layer. PASS means the ceiling held even under the assumption
that the signing key itself is in the attacker's hands.
**Seals:** R1 `46592082af4ac996b9460a41c45b892dae31fd08cac032f896e1ca4c540e0f47` · R2 `fcaae588c53e9c864a82af9c15ca8db1e5db1a6bc5c77c149c40562a02adc064`

---

## Declared boundary pressure (phases 14–17) — new in this release

These four phases are different in kind from phases 0–13: instead of
proving a defense holds, each one **measures and openly reports a limit
CRATOS states explicitly rather than silently assumes.** A `PASS` on a
declared-boundary phase means the kernel behaved exactly as its own stated
scope says it will at that boundary — not that the boundary doesn't exist.

### Phase 14 — one malformed/misanchored/misidentified/replayed proposal collapses the kernel to Null in a single shot
**What was tested:** 5 distinct single-shot collapse triggers (two
structural-malformation variants, an invalid causal anchor, an identity
mismatch, a replay), each against a fresh kernel instance with a large,
PRNG-drawn starting budget, followed by a small, well-formed follow-up
proposal.
**Why PASS is correct — and what it deliberately does NOT claim:** this is
not a claim that CRATOS resisted a denial-of-service attempt. The opposite
is what's being measured and openly reported: a single hostile proposal
does collapse the kernel unconditionally, regardless of remaining budget,
and every proposal afterward — including a trivially small, well-formed one
— is then rejected. This phase makes no claim about whether an attacker can
reach a deployment's admission call path in the first place; that is a
transport/access-control question entirely outside this kernel's own scope.
A real deployment's exposure to this boundary depends entirely on how
narrowly it restricts who can submit a proposal at all.
**Seals:** R1 `327835acd34556df2fa5ef8229e35e9908282e0fd8d8150dccde6583671971a8` · R2 `303888ad54b2e93f02dd33c130fe9b415cc8ff0adba54984d1d45a28f94a6146`

### Phase 15 — legitimate use squeezed close to budget, followed by ANY over-budget request, collapses the kernel with no distinction between critical and hostile
**What was tested:** two kernels, each legitimately squeezed close to their
lane budget through normal use, then each sent a final over-budget request
of identical PRNG-drawn magnitude — one framed as a critical/legitimate
need, one framed as hostile.
**Why PASS is correct — and what it deliberately does NOT claim:** this
phase reports, openly, that the kernel makes **no intent-based
distinction** between the two requests — both are rejected and both
collapse the kernel identically. That is by design: the kernel enforces a
resource boundary, not an intent classifier. Any system that needs to treat
"legitimate but over-budget" differently from "hostile and over-budget"
must implement that judgment in a layer above the kernel — CRATOS's
admission boundary does not, and is not claimed to, make that judgment
itself.
**Seals:** R1 `16af7583f48f002778760176cde8eb56990e5ecc0ae0b1fb93c189a4051e1525` · R2 `48605f9ef97384d87c2002a19a39045ade58c4ada0ca25b618c03ef5daff67d7`

### Phase 16 — N independently-authorized kernel instances sum to far more admitted residual than a single reference ceiling
**What was tested:** many independent kernel instances (default 50), each
individually authorized against the same reference budget, each admitting
within its own limit; the aggregate admitted total across all instances is
then compared against that single reference budget.
**Why PASS is correct — and what it deliberately does NOT claim:** this
phase confirms, and openly reports, that **no cross-instance or global
budget concept exists in this crate.** Each instance is individually
compliant with its own signed budget — that part is the guarantee — but the
sum across many instances is not bounded by that same number, because
nothing in this crate is designed to bound it. A deployment running many
instances that needs a fleet-wide ceiling must implement that coordination
in a layer above the kernel; this phase demonstrates that need is real, not
theoretical, and quantifies it directly.
**Seals:** R1 `a9fb09fbf6a49bee58fe7456b983ad1e65ea1e2943eefd77081f5a92a5051696` · R2 `d6a6b0f21aad79273f0c02da630f50c8dab1f731e5eb3e89216909fc47cd28f3`

### Phase 17 — a sequence of individually in-budget effects across different lanes is admitted lane-by-lane with no predicate over the SET or ORDER of admitted effect kinds
**What was tested:** four kernel instances, each funded across three
different lanes (ContentUpdate, DriverLoad, NetworkConnect), each admitting
one effect per lane in a PRNG-shuffled order.
**Why PASS is correct — and what it deliberately does NOT claim:** every
order tried admitted successfully, and this phase reports openly that the
kernel evaluates each lane's budget independently, with **no rule over
which combination or sequence of effect kinds is meaningful together.**
Two effects that are each individually in-budget but form a dangerous
combination *as a set* (for example, a content update alongside a driver
load) are not, and are not claimed to be, caught by this kernel — that kind
of semantic, cross-lane policy is a deliberate non-goal of this crate and
belongs in a layer above it, if a given deployment needs it.
**Seals:** R1 `6534a44236aaa7ede5461ea3890b6b1bdee3d5c692f382dd3079d04e2953d77a` · R2 `d15f6ed32b02f55e0afd3cc5351465e6ad16bdd814c2e960cfa238c003e4faae`

---

## How to check any claim above yourself

Every "what this run does NOT demonstrate" statement paraphrased above
appears verbatim, in full, inside each run's own `CRATOS_REPORT.md`, under
that phase's `### Phase N — what this run does NOT demonstrate` heading —
read it there for the authoritative wording. Every seal listed above can be
recomputed independently from that same file, using nothing but a standard
sha256 tool, per the instructions printed directly beneath each phase's
`Native seal (...)` line in the report. This document adds no claim beyond
what those reports already state publicly.
