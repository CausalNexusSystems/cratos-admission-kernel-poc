# Notice — Binary and Source Under NDA

This package is a **public evidence-only Proof-of-Concept release**. It does
**not** include, and will never include without a separate signed agreement:

- The compiled CRATOS binary (harness executable or verifier executable), in
  any form, for any platform.
- The CRATOS source code — the admission kernel crate (`cratos/`), the test
  harness source, or the independent verifier source.
- Any build scripts, CI configuration, or dependency manifests beyond the
  generic, standard tools referenced in `VERIFY.md` (a sha256 utility).

**Everything CRATOS-specific in this package is Rust source code and a
compiled Rust binary, both of which are under NDA and are shared only with
parties who have executed a non-disclosure agreement with Causal Nexus
Systems.**

What this package contains instead is strictly limited to:

- Plain-text and Markdown **run reports** (`CRATOS_REPORT.md` /
  `CRATOS_REPORT.txt`) describing what each phase tested and its result.
- Self-referential **sha256 file manifests** and **Merkle roots**, which let
  any third party confirm the reports were not altered after the run, using
  nothing but a standard, generic sha256 tool (no CRATOS code required).
- This package's own wrapper documents (this notice, the top-level README,
  and the plain-language phase rationale document), which are original
  commentary and not sealed evidence.

If you are evaluating CRATOS and need access to the source, the compiled
binary, or the ability to run the harness yourself, that access is available
under NDA — contact Causal Nexus Systems directly. Nothing in this notice
should be read as a commitment about the terms of any such NDA; it exists
only to state plainly what this specific public package does and does not
contain.
