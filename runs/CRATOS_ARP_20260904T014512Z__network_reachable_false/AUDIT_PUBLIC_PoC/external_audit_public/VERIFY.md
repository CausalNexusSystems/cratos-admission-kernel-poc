# Verifying this package

This is the PUBLIC, redacted evidence tier: it does not include CRATOS's kernel source, any compiled binary or script, or per-case/per-agent decision detail. It includes only what is needed to independently confirm that the files in this package have not been altered since publication. Full technical detail is available to auditors under NDA.

No CNS code of any kind -- not a kernel source file, not even a small standalone verification utility -- ships in this package. Verification below uses only standard operating-system tools (a `sha256`-family hash tool, present on virtually every machine) plus a plain-language description of a public, non-proprietary algorithm (SHA-256 and the Merkle Mountain Range fold, the same construction used in certificate-transparency logs and many blockchains). You do not need anything from CNS to check this package; write your own few lines of script in any language, or work through it by hand.

## Recomputation steps

1. For every `sha256,relative_path` row in `public_files_sha256.csv` (this folder's own manifest, scoped to exactly its own contents), compute `sha256sum <relative_path>` (or equivalent, e.g. `shasum -a 256` on macOS) and compare against the recorded hash.
2. For each file, compute its Merkle leaf as `sha256(relative_path_utf8_bytes || 0x00 || file_bytes)`.
3. Fold the leaves as a Merkle Mountain Range: append each leaf as a new height-0 peak, merging any two adjacent equal-height peaks with `pair_hash(a, b) = sha256(a || b)` until no two adjacent peaks share a height; then bag the remaining peaks highest-height-first with the same `pair_hash`.
4. Compare the result against `public_merkle_root.txt` in this folder (or `poc_bundle_merkle_root.txt` at the root of `AUDIT_PUBLIC_PoC/`, if verifying that bundle as a whole -- each bundle carries its own manifest scoped to exactly its own contents).

Note: `run_merkle_root.txt` in this folder records the Merkle root of the ORIGINAL, complete run directory (which included files not present in this public package, such as per-agent decision logs and source, held under NDA). It is included for cross-reference against a claim made elsewhere (e.g. a report or announcement quoting that root), but cannot itself be recomputed from this folder alone -- only `public_merkle_root.txt` can, and that is the actual public-tier integrity check.

If you would prefer CNS walk through this verification with you live, or provide a compiled verification tool under separate agreement, contact us -- that is a service we can offer on request, but it is deliberately not bundled automatically into this public package.
