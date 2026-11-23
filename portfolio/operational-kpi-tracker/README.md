# Operational KPI Tracker — fictional dataset

**Independent portfolio demonstration · All records, people, dates and results are fictional.** This example illustrates how to turn an administrative request register into auditable indicators. It is not an actual deployment or a claim of business impact.

## Files

- [sample_requests.csv](sample_requests.csv) — eight synthetic request records; dates use ISO 8601 format and blank cells mean missing/not applicable.
- This README — data dictionary, measurement rules, reproducible spreadsheet formulas and validation checks.

Open the CSV in Excel or Google Sheets. For the formulas below, assume headers are in row 1 and records in rows 2–9, with columns A:H matching the CSV. Dates must be imported as actual dates (not text). Set **reporting date to 2026-10-17** for the example.

## Data dictionary

| Column | Meaning |
|---|---|
| request_id | Unique identifier; duplicates are flagged by status rather than repeated IDs |
| received_date | Date request entered the queue |
| original_due_date | First committed deadline; never overwrite to hide lateness |
| closed_date | Date the request was completed; blank when not closed |
| status | Closed, Open, Waiting for information, or Duplicate |
| owner | Fictional responsible analyst |
| first_review_result | Pass or Fail at the first quality check; blank if not reviewed |
| defect_category | Example classification for a first-review failure |

## KPI definitions and worked example

| KPI | Calculation | Fictional result |
|---|---|---|
| On-time closure | Eligible closed requests closed on/before original due date ÷ all eligible closed requests | **3 / 5 = 60%** |
| First-pass quality | First reviews marked Pass ÷ all first reviews marked Pass or Fail | **3 / 5 = 60%** |
| Open overdue backlog | Eligible non-closed requests due before 2026-10-17 | **2 requests** |
| Median cycle time | Median calendar days between receipt and closure for eligible closed requests | **6 days** |

**Eligibility:** exclude Duplicate and Cancelled (if added in future). Waiting for information remains eligible and does not pause the original due date. For first-pass quality, only reviewed items enter the denominator. If a denominator is zero, display **N/A**, never 0%.

## Spreadsheet implementation

Add these helper columns:

- **I1: eligible**; I2: `=IF(OR(E2="Duplicate",E2="Cancelled"),0,1)`
- **J1: on_time_closed**; J2: `=IF(AND(I2=1,E2="Closed",ISNUMBER(D2)),IF(D2<=C2,1,0),"")`
- **K1: cycle_days**; K2: `=IF(AND(I2=1,E2="Closed",ISNUMBER(D2)),D2-B2,"")`
- **L1: overdue_open**; L2: `=IF(AND(I2=1,E2<>"Closed",C2<DATE(2026,10,17)),1,0)`

Fill I2:L2 down to row 9.

Summary formulas (Excel / Google Sheets with English function names and comma separators; locale settings may require semicolons):

```excel
=IFERROR(COUNTIF(J2:J9,1)/COUNT(J2:J9),"N/A")
=IFERROR(COUNTIF(G2:G9,"Pass")/(COUNTIF(G2:G9,"Pass")+COUNTIF(G2:G9,"Fail")),"N/A")
=SUM(L2:L9)
=IFERROR(MEDIAN(K2:K9),"N/A")
```

Format the first two cells as percentages. The formulas intentionally use the original due date, not an adjusted date.

## Validation checklist

1. Confirm unique request IDs and allowed status values.
2. Ensure closed records have valid closure dates; open records must not.
3. Check `closed_date >= received_date` and `original_due_date >= received_date`.
4. Confirm each first-review result is Pass, Fail or blank; classify Fail defects consistently.
5. Reconcile counts before reporting: **5 Closed, 1 Open, 1 Waiting for information, 1 Duplicate**.
6. Confirm the four summary results against the worked example above.
7. Keep the reporting date explicit; overdue backlog changes as time passes.

## Next possible iteration

Create an Excel/Sheets dashboard with status counts, weekly trends, owner workload and data-quality alerts. A production workflow would also require access controls, a documented data dictionary, privacy review and change history.

**Scope:** Synthetic learning artifact only; no hospital, patient, employer or client information.
