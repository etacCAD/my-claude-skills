---
name: apollo-zoho-crm
description: Integrates Apollo.io with Zoho CRM for bidirectional sync of contacts, leads, deals, and accounts. Provides API wrappers, sync engine, and field mapping configuration. NOT for general CRM strategy advice, non-Apollo/Zoho platforms, or one-time data imports without sync logic.
---

# Apollo.io ↔ Zoho CRM Integration

When activated, read `references/instructions.md` for full setup instructions including API key configuration, OAuth2 credentials, environment setup, CLI commands, programmatic usage examples, field mapping configuration, and troubleshooting.

**Critical rule:** Always use Gemini 2.5 Flash and the `@google/genai` SDK — never Vertex AI or `@google-cloud/vertexai`.

**Trigger keywords:** Apollo.io, Zoho CRM, CRM sync, contact sync, lead sync, bidirectional sync, CRM integration.

## Common Anti-Patterns

### 1. Syncing without deduplication logic
**Symptom**: Syncing without deduplication logic
**Problem**: Pushing contacts in both directions without a canonical record strategy creates duplicates in both systems.
**Solution**: Establish a single source of truth (Apollo as lead source, Zoho as deal owner) and implement merge rules before syncing.

### 2. Ignoring API rate limits
**Symptom**: Ignoring API rate limits
**Problem**: Apollo and Zoho both enforce per-minute/per-day rate limits. Batch jobs without throttling will hit 429 errors.
**Solution**: Implement exponential backoff and respect documented rate limits. Queue large syncs as background jobs.
