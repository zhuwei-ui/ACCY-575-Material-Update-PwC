# Module 17 — Engagement Brief: Disbursement & Vendor-Payment Fraud-Risk Audit

*You are handed a task and a dataset. What you analyze, what you ask your agent, which tools you use — all of it is yours to decide. How you approach it is entirely yours; the deliverable is what's assessed.*

## Your role

You are a data analyst in the **Internal Audit** function of a large public-sector controller's office. The Chief Audit Executive has opened a procure-to-pay fraud-risk audit and assigned you **one agency's vendor payments** for the fiscal year. Your work goes to the CAE and the audit committee.

## The task

> Find the fraud-risk exceptions worth the audit committee's attention in your assigned agency's FY2025 disbursements, and report what you found and what you would do about it.

## The data

**State of Oklahoma Vendor Payments (OMES checkbook), FY2025** — real disbursements of public funds, free, no login.

- Landing page: `https://data.ok.gov/dataset/state-of-oklahoma-vendor-payments-fiscal-year-2025`
- File list (CKAN API): `https://data.ok.gov/api/3/action/package_show?id=state-of-oklahoma-vendor-payments-fiscal-year-2025` — twelve monthly `Vendors_Public_YYYYMM.csv` files (Jul 2024 – Jun 2025), ~35 MB total. **One row = one paid voucher line.**
- Public record, portal-listed as **"No License Provided"** — cite the source and your pull date; don't re-publish it as licensed open data.

A short field reference (the data ships wider than this; explore it):

| Field | What it is |
|---|---|
| `AGENCYNBR`, `AGENCYNAME` | Paying agency — **your assignment key**. |
| `VENDOR_NAME`, `CITY`, `STATE`, `POSTAL` | Vendor identity and location. |
| `PYMNT_AMT` | Dollar amount paid. |
| `PYMNT_DT`, `EFFDT`, `INVOICE_DT`, `CANCEL_DT` | Payment / effective / invoice / cancellation dates (no time of day). |
| `VOUCHER_ID`, `VOUCHER_ID_RELATED` | Voucher identifiers and related-voucher links. |
| `ACCOUNT` / `ACCTDESCR` | Expense object code — *what* was bought. |
| `FUND_CODE` / `FUNDDESCR` | Funding source — *where the money came from*. |
| `PROGRAM_CODE`, `DEPTID`, `PO_ID`, `ITEM_DESCRIPTION` | Program, department, PO, and free-text line description. |

Your assigned agency (`AGENCYNBR`) is on the assignment sheet.

## What to hand in

- **A findings memo to the audit committee** — the exceptions you are raising, why each one matters, the exposure, and the follow-up you recommend, written so the reader can act on it.
- **The repository behind it** — reproducible in the Part 1 way (`uv`, data validation, tests, honest commit history), so anyone can clone it and re-run your analysis to your numbers.

Bring both to the review session and be ready to walk us through your result.

## Ground rules

- Individual work.
- Use AI as much as you want — you own every number in the memo regardless.
- Anything you rely on from outside the file (a statutory bid threshold, a standard, a definition) you look up and cite yourself.
- No prescribed method, no required tools, no checklist. Figure it out.

## Grading

You are assessed on the **deliverable** — the memo and the reproducible repository behind it — and on how well it holds up to scrutiny. There is no set procedure to follow; reaching a defensible result is the work. See [Assessment across Part 3](../README.md#assessment-across-part-3).
