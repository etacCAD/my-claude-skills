---
name: coppa
description: >-
  Audits the codebase for Children's Online Privacy Protection Rule (COPPA) compliance during manual checks or pre-deployment pipelines. NOT for non-privacy security checks or generic code audits.
---

# COPPA Compliance Auditor

## Overview
This skill audits the project's compliance with the Children's Online Privacy Protection Act (COPPA). It verifies database write security, ensures parental consent restrictions on under-13 accounts, prevents sensitive mental/composure stats from being exposed to parents, and validates data deletion routines.

## Dependencies
- **Backend Engineer (Firebase BaaS)**: The Firestore data structures and Cloud Functions implementing the parental consent flows and user claims.
- **Chief Security Officer - Architecture and Dev**: Conducting database rules and platform security validations.

## Quick Start
To perform a compliance audit on the current workspace, run:
```bash
node scripts/audit_coppa.js
```

If any compliance checks fail, the script will exit with code `1` and list the violations.

## Utility Scripts
The audit script runs statically and dynamically as part of QA:
- **`node scripts/audit_coppa.js`**: Analyzes rules files, functions, and codebase files statically.
- **`node scripts/run_qa.js`**: Simulates the child registration, parent invitation, and consent/revocation flows in the emulator environment.

## Rate Limiting
No external APIs are called during local auditing, so no rate limits apply.

## Common Mistakes
- **Exposing Composure Scores to Parents**: Mental composure / self-reflection fields are highly sensitive and under COPPA/privacy rules must *never* be exposed in the parent dashboard API response or view.
- **Missing Write Constraints on Unconsented Accounts**: Allowing writes to a child's subcollections (e.g., assessments, video clips) before `parentConsentGranted` is set to `true`.
- **Incomplete Deletion**: Deleting the Firestore `/keepers/{keeperId}` record but leaving raw videos or highlight clips in Cloud Storage. Deletions must clean up both Firestore and Cloud Storage.
