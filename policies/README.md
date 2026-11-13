 # Policy/sc28_encryption.rego

- Title: SC-28 - Encryption at Rest (GCS)
- Description: "Every google_storage_bucket must encrypt at rest with a customer-managed encryption key (CMEK)."
- Control_id: SC-28
- NIST-800-53
- Severity: High
- Remediation: "Add an encryption { default_kms_key_name = ... } block referencing a google_kms_crypto_key you control."

# Policy/cm6_required_tags.rego

- Title: CM-6 - Configuration Settings (required compliance labels)
- Description: "Every taggable resource must carry the four required labels: project, environment, managed_by, compliance_scope."
- Control_id: CM-6
- Framework: NIST-800-53
- Severity: Medium
- Remediation: "Add the four required labels (project, environment, managed_by, compliance_scope) to the resource."

# Policy/ac3_no_public.rego

- Title: AC-3 - Access Enforcement (no public GCS or open firewall)
- Description: "GCS buckets must enforce uniform_bucket_level_access AND public_access_prevention=enforced. Firewall rules must not allow 0.0.0.0/0 on management ports (22, 3389)."
- Control_id: AC-3
- Framework: NIST-800-53
- Severity: Critical
- Remediation: "Set uniform_bucket_level_access = true, public_access_prevention = enforced. For firewalls, narrow source_ranges or remove the rule."
