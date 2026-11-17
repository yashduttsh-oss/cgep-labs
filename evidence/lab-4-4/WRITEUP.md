## Lab 4.4 — Evidence Chain of Custody

This lab produces cryptographically signed, tamper-evident evidence for every
pipeline run and lets anyone verify it independently. The chain of custody rests
on four properties, each proven by a specific artifact:

| Property | What it guarantees | Artifact that proves it | How it's verified |
|----------|--------------------|-------------------------|-------------------|
| **Integrity** | The evidence bundle hasn't been altered since it was produced | `evidence-<run_id>-<sha>.tar.gz.sha256` (SHA-256 sidecar) | `verify-evidence.sh` recomputes the bundle's SHA-256 and compares it to the sidecar. The tamper test confirms detection: appending one byte changes the hash and breaks the match. |
| **Authenticity** | The evidence was signed by a known, trusted identity — not forged | `...tar.gz.sig.bundle` (Cosign signature + Fulcio certificate) | `cosign verify-blob` checks the signature against the Fulcio-issued certificate, bound to the GitHub Actions OIDC identity (`--certificate-oidc-issuer https://token.actions.githubusercontent.com`). |
| **Timestamp / Non-repudiation** | Proof of *when* it was signed, publicly recorded so the signer can't deny it | Rekor transparency-log entry (referenced inside the `.sig.bundle`) | Cosign confirms the signature is recorded in the Rekor public log — an immutable, timestamped record of the signing event. |
| **Preservation** | The evidence can't be deleted or overwritten during its retention window | S3 Object Lock retention + `version_id` in `receipt.json` | `verify-evidence.sh` reads the Object Lock retention date and fails if it has passed. Object Lock also blocks overwriting the key, so a tampered copy can never replace the original in the vault. |

### Supporting artifacts
- `.github/workflows/grc-gate.yml` — signs and uploads evidence *before* the gate
  can fail the build, so even a failing run leaves signed evidence.
- `scripts/verify-evidence.sh` — runs the integrity, authenticity/timestamp, and
  preservation checks; prints `CHAIN INTACT` only when all pass.
- `evidence/lab-4-4/receipt.json` — human-readable record tying the run to its
  bundle key, version ID, SHA-256, and commit.

### Verification result
`verify-evidence.sh <run_id>` ends with `Verified OK` (from Cosign) and
`CHAIN INTACT for run <run_id>`, exiting 0. The tamper test produces a hash that
no longer matches the sidecar, while the immutable vault copy still verifies clean
— demonstrating that tampering is both detectable and futile.