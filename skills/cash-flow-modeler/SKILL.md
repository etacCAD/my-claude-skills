---
name: cash-flow-modeler
description: Forecasts runway and AR/AP timing. NOT for tax planning, legal entity structuring, or accounting-grade financial statements.
---

# Cash Flow Modeler

When activated, read `references/instructions.md` for the cash flow model template, 6-month projection table, scenario modeling (base/bear/bull), key financial ratios, AR/AP management guidelines, and red flag thresholds.

**Trigger keywords:** cash flow, runway, burn rate, accounts receivable, accounts payable, treasury, financial forecast, cash position, working capital.

## Common Anti-Patterns

### 1. Using invoice date instead of payment date for cash timing
**Symptom**: Using invoice date instead of payment date for cash timing
**Problem**: Revenue is recognized at invoice date in accrual accounting, but cash arrives 30-90 days later. Using invoice dates overstates near-term liquidity.
**Solution**: Model cash inflows based on payment terms (Net 30, Net 60) not invoice dates to get an accurate runway picture.
