# OSCAL layer

Machine-readable compliance artifacts for this repo, authored with [compliance-trestle](https://github.com/oscal-compliance/compliance-trestle) against NIST SP 800-53 Rev 5.

## Components

| File | Describes | Controls claimed |
|------|-----------|------------------|
| [`components/compliant-s3.json`](components/compliant-s3.json) | The `compliant-s3` Terraform module ([`terraform/primitives/compliant-s3`](../terraform/primitives/compliant-s3)), which provisions an S3 bucket with KMS encryption, access logging, versioning, and least-privilege / public-access-block policies. | `sc-28` (encryption at rest), `ac-3` (access enforcement), `au-3` (audit record content), `cm-6` (baseline configuration) |

Each `implemented-requirement` names the specific Terraform resource that enforces the control (via a `terraform-resource` prop) and links to the signed evidence bundle (`rel: "evidence"`).

## Profiles

| File | Purpose |
|------|---------|
| [`profiles/cge-p-minimum.json`](profiles/cge-p-minimum.json) | Selects the minimum control set this repo claims — `sc-28`, `ac-3`, `au-3`, `cm-6` — by importing the NIST SP 800-53 Rev 5 catalog. Resolving it produces a flat catalog of just those four controls with full text (the artifact a System Security Plan would import). |

## Where the evidence lives

The component's evidence links point at the signed, timestamped run bundle produced by the Lab 4.3 + 4.4 pipeline and stored under S3 Object Lock in the evidence vault:

```
s3://cgep-lab-grc-evidence-vault-6b40f971/runs/44113314695/evidence-44113314695-2f606d8a69c3ecbb6168a1603582c7a5cc2eda77.tar.gz
```

To walk the chain yourself — start from a control claim, follow its `rel: "evidence"` href into the vault, and cryptographically verify the artifact:

```bash
EVIDENCE_VAULT="cgep-lab-grc-evidence-vault-6b40f971" \
  bash scripts/verify-evidence.sh 44113314695 --profile default
```

The script confirms integrity (SHA-256), authenticity (cosign signature), and preservation (Object Lock retention). The receipt for this run is captured at [`evidence/lab-4-4/receipt.json`](../evidence/lab-4-4/receipt.json), and the trestle validation output at [`evidence/lab-6-1/trestle-validate.txt`](../evidence/lab-6-1/trestle-validate.txt).
