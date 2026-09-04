CRATOS — Public Proof-of-Concept Package (REDACTED, for public distribution)
Run: CRATOS_ARP_20260904T014323Z

This folder is self-contained: it does not depend on anything else from
the originating repository and can be copied, e.g. to a public GitHub
repository or referenced from a public post, for independent third-party
verification that this run's evidence has not been altered since
publication.

This package deliberately does NOT include, and never will:
- CRATOS's kernel source code (`cratos`/`cratos-harness` crates), in
any form, including any generic or kernel-independent helper utility
- any compiled binary of CRATOS, or of any verification tool, at all
- per-case or per-agent decision-level telemetry
- the protocol descriptor or machine-readable exercise descriptor

This is a standing CNS policy, not a case-by-case judgment call: source
code -- kernel or otherwise -- is never included in a public evidence
package, and is not shown to auditors or investors as part of one either.
Auditors and investors see results and cryptographic commitments; source-
level review, where it happens at all, happens separately, under NDA, on
its own terms. All of the material listed above is available to auditors
under NDA (`AUDIT_NDA/`), not in this public tier.

What this package DOES include is everything needed to independently
confirm the cryptographic seal, using tools you already have (no CNS code
of any kind is required -- see VERIFY.md):

Contents:
CRATOS_REPORT.md / .txt        redacted run report: pass/fail per phase, aggregate counts, scope
external_audit_public/          this package's own copy of the redacted report + manifests + VERIFY.md
poc_bundle_files_sha256.csv     sha256 of every file in THIS folder (computed last, over the finished bundle)
poc_bundle_merkle_root.txt      Merkle root of poc_bundle_files_sha256.csv -- recompute this yourself
with any standard sha256 tool plus the plain-language recipe in VERIFY.md

Note on disclosure: this is a deliberately redacted public evidence tier,
following the same public/NDA separation used elsewhere at Causal Nexus
Systems. Publishing this package (e.g. alongside a summary of this run) is
safe: it discloses the outcome of the exercise and lets a third party
verify it was not tampered with, without disclosing how the kernel is
implemented internally, and without handing over any code.

To verify: see external_audit_public/VERIFY.md for the recomputation steps.
