# GCP Security Baseline

Terraform baseline that applies a set of preventive and detective security
controls to a single GCP project. Built as part of the CGE-P / GRC Engineering
Club hands-on labs (Lab 5.4 — GCP Security Services).

---

## What this deploys

| File | Purpose |
|------|---------|
| `org_policy.tf` | Preventive controls — three Org Policy constraints enforced at the API |
| `audit_logs.tf` | Detective control — Data Access audit logs for Storage, KMS, and IAM |
| `wif.tf` | Keyless GitHub Actions auth via Workload Identity Federation (OIDC) |
| `main.tf` | Provider configuration and project-level wiring |
| `variables.tf` | `gcp_project`, `github_repo` inputs |

---

## Preventive controls — Org Policy constraints

Three organization policy constraints are enforced (`enforce = "TRUE"`, meaning
offending requests are **rejected at the API**, not just flagged):

| Constraint | Resource | What it prevents |
|------------|----------|------------------|
| `storage.uniformBucketLevelAccess` | `uniform_bucket_access` | Legacy per-object ACLs on GCS buckets — forces IAM-only access |
| `iam.disableServiceAccountKeyCreation` | `disable_sa_keys` | Creation of long-lived service account keys (the most common GCP credential-leak vector) |
| `compute.requireOsLogin` | `require_oslogin` | SSH via project/instance metadata keys — forces IAM-governed OS Login |

---

## Detective control — Data Access audit logs

**Key GRC lesson: Data Access audit logs are OFF by default in GCP.**

Admin Activity logs are always on and free, but Data Access logs — who *read*
or *wrote* data, not just who changed configuration — have to be explicitly
enabled. Because almost nobody turns them on, their absence is one of the most
common findings in a GCP audit: when an incident happens, there is no record of
what data was actually touched.

`audit_logs.tf` enables all three Data Access log types — `DATA_READ`,
`DATA_WRITE`, and `ADMIN_READ` — for three high-value services:

- `storage.googleapis.com`
- `cloudkms.googleapis.com`
- `iam.googleapis.com`
