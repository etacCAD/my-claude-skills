---
name: accidental-data-loss-prevention
description: |
  **STOP AND VERIFY**: Before running any command or tool that results in irreversible data loss, you MUST obtain explicit user consent.
  When in doubt, ask. It is better to wait for confirmation than to accidentally delete production data or critical project assets.
  Use this for:
  - SQL: DROP TABLE/VIEW/SCHEMA/DATABASE, TRUNCATE, or broad DELETE (missing WHERE or using 1=1).
  - Cloud Storage: gsutil rm or gcloud storage rm targeting production data or critical buckets.
  - Infrastructure: gcloud projects delete, deleting Spanner/BigQuery/Dataproc resources, deleting secrets, or KMS key destruction. NOT for routine read-only queries, SELECT statements, or reversible operations.
license: Apache-2.0
metadata:
  version: v1
  publisher: google
---

# Accidental Data Loss Prevention

> [!CAUTION]
>
> **STOP AND VERIFY**: Before running any command or tool that results in
> irreversible data loss, you **MUST** obtain explicit user consent.

## Mandatory Procedure

1.  **Halt Execution**: Do **not** execute the command.
2.  **Request Consent**: Explain clearly to the user:
    -   The **impact** of this deletion.
    -   **Why** you believe this is necessary.
    -   A request for their **explicit approval** to proceed.
3.  **Wait**: Only proceed if the user provides clear, affirmative consent in
    the conversation.

## Common Anti-Patterns

### 1. Treating all DELETE operations as safe
**Symptom**: Treating all DELETE operations as safe
**Problem**: Assuming row-level DELETEs with a WHERE clause are always safe to auto-run.
**Solution**: Always verify the WHERE clause scope. If the delete affects more than a targeted set of rows, surface it for explicit confirmation.

### 2. Skipping confirmation for TRUNCATE
**Symptom**: Skipping confirmation for TRUNCATE
**Problem**: TRUNCATE behaves like DROP+CREATE — it bypasses row-level logging and cannot be rolled back in some DB engines.
**Solution**: Always require explicit user consent before running TRUNCATE, regardless of context.
