# CRATOS — Public PoC Bundle (GitHub release)

This package is a **GitHub-publishable Proof-of-Concept bundle** built from two
real, independently-generated CRATOS Adversarial Run Protocol (ARP) executions.
It is derived *entirely* from material that was already produced by the
`AUDIT_PUBLIC_PoC` tier of the CRATOS test harness — nothing inside
`runs/*/AUDIT_PUBLIC_PoC/` has been edited, regenerated, or re-sealed. Every
file in those two subfolders is byte-for-byte what the harness itself wrote
and sealed at run time. This top-level `README.md`, `NOTICE_NDA.md`, and
`PHASE_RATIONALE_PUBLIC.md` are the only files added on top, and none of them
carry cryptographic weight — they are plain-language wrapping, clearly
separated from the sealed evidence.

## What's in this package

```
CRATOS_PoC_GitHub_20260904/
├── README.md                        <- this file
├── NOTICE_NDA.md                    <- explicit NDA / no-binary notice
├── PHASE_RATIONALE_PUBLIC.md        <- plain-language "why" per phase (all 18)
├── package_files_sha256.csv         <- flat integrity manifest of THIS wrapper package
├── package_manifest_root.txt        <- root hash of package_files_sha256.csv (see note below)
└── runs/
    ├── CRATOS_ARP_20260904T014323Z__network_reachable_true/
    │   └── AUDIT_PUBLIC_PoC/        <- unmodified, as produced by the harness
    └── CRATOS_ARP_20260904T014512Z__network_reachable_false/
        └── AUDIT_PUBLIC_PoC/        <- unmodified, as produced by the harness
```

Each `AUDIT_PUBLIC_PoC/` folder is fully self-verifying on its own, with its
own `poc_bundle_files_sha256.csv` / `poc_bundle_merkle_root.txt` at its root
and a nested `external_audit_public/` tier with its own
`public_files_sha256.csv` / `public_merkle_root.txt`. See
`AUDIT_PUBLIC_PoC/README_PoC.txt` and `AUDIT_PUBLIC_PoC/external_audit_public/VERIFY.md`
inside each run for the exact recomputation instructions. **Verify each run
using its own manifest — do not use `package_files_sha256.csv` to validate
run evidence; that file only covers the three wrapper documents added in
this package, described below.**

## Why two runs

These two runs are a deliberate **with-internet / without-internet
comparison pair**, generated back to back on the same machine, same harness
build, same CRATOS core (`v0.6.0`):

| Run ID | Outbound network route at run start |
|---|---|
| `CRATOS_ARP_20260904T014323Z` | REACHABLE |
| `CRATOS_ARP_20260904T014512Z` | NOT REACHABLE |

Both runs report `VALIDATION_STATUS: PASS`, identical phase count (18/18),
and identical structural results across all 18 phases. The network-route
probe result is itself sealed into each run's own evidence (see
`run_header.txt` inside each `external_audit_public/`), so this is not an
unverified claim about the environment — it is part of what each run signs.
Together, the pair is offered as direct, reproducible evidence that CRATOS's
admission decisions do not depend on network reachability: the kernel is a
local, deterministic function of signed input and kernel state, with no
outbound calls of any kind on its decision path.

## What's new relative to prior public releases

Earlier public CRATOS PoC releases covered **13 phases** (phases 0–12 in
older wording, or phases 0–13 depending on release). This package's two runs
are the first public release covering the **full 18-phase suite (phases
0–17)**, adding four new phases under a new report category,
**"declared boundary"** (`declared_boundary_pressure`), which is additive to
the existing "fixed regression" and "randomized" phases already present in
prior releases — no earlier phase's logic, hypothesis, or check was altered
to make room for the new ones.

The four new phases (14–17) each probe a boundary that CRATOS's admission
kernel **states explicitly, rather than silently assumes**: what happens on
a malformed/misanchored/replayed proposal, what happens when a legitimate
actor and a hostile actor make the identical over-budget request, what a
single kernel instance's budget does and does not say about a fleet of many
instances, and whether admission order across different effect lanes
matters. See `PHASE_RATIONALE_PUBLIC.md` in this package for a plain-language
explanation of each phase's result and why that result is the expected,
by-design engineering outcome — not a workaround or a lucky pass.

Also new relative to earlier public releases: **every phase in the main
report now carries its own independent native seal** (a sha256 computed
purely from that one phase's own `summary.txt` content plus its own
`result.json`), shown directly under that phase's `Result: PASS` line in
`CRATOS_REPORT.md` / `CRATOS_REPORT.txt`, in addition to the existing
whole-run Merkle root. This lets a reviewer check a single phase's integrity
without needing the rest of the run.

## No binary, no source — NDA

This package contains **no compiled CRATOS binary and no CRATOS source
code of any kind** — not even a generic verifier. It contains only
plain-text/Markdown reports, self-referential sha256 manifests, and Merkle
roots. See `NOTICE_NDA.md` for the explicit statement on what is and is not
included, and why.

## How to verify this package

1. Recompute each run's own seals using that run's own manifest and the
   instructions in that run's `AUDIT_PUBLIC_PoC/external_audit_public/VERIFY.md` —
   this requires nothing beyond a standard sha256 tool; no CRATOS binary is
   needed to verify the redacted/public tier.
2. Independently recompute any single phase's native seal directly from that
   phase's section of `CRATOS_REPORT.md`/`CRATOS_REPORT.txt`, per the
   instructions printed immediately under each phase's `Native seal (...)`
   line.
3. Optionally recompute `package_files_sha256.csv` / `package_manifest_root.txt`
   in this top-level folder to confirm the three wrapper documents were not
   altered after packaging (informational only — this manifest is not part
   of CRATOS's own sealing scheme; see the note in that file).

## Scope of this package

This is a Proof-of-Concept evidence package, not a product download and not
a security audit. It documents two specific executions of the CRATOS
admission kernel's test harness, on one machine each, at the stated
timestamps. It makes no claim beyond what each run's own
"Scope and Falsifiability" section (inside each `CRATOS_REPORT.md`) states
explicitly — read that section inside each run before drawing conclusions
about what is, and is not, demonstrated.
