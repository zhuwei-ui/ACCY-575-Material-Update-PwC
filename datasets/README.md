# Datasets

The datasets the Part 1 walkthroughs work on. Copy the one a module asks for into your own project's `data/raw/`; don't commit it to your repository. From your project folder:

```bash
mkdir -p data/raw
cp ~/Projects/accy575/course-materials/datasets/transactions.csv data/raw/
```

`~/Projects/accy575/course-materials` is where you cloned this repository (see [Getting the course materials](../README.md#getting-the-course-materials)); run `git -C ~/Projects/accy575/course-materials pull` first if the file isn't there yet. The Module 4 `.gitignore` already keeps `data/raw/` out of Git.

All three are public data. None of them is licensed material, so nothing here is restricted to enrolled students.

## `transactions.csv`

Card transactions, as they come out of the source system: **437,579 rows**, columns `date`, `amount`, `vendor`.

It is deliberately messy, which is why Module 5 uses it:

| Quirk | Count | What it looks like |
|---|---|---|
| Blank `date` | 222,881 | `,$364.07 ,#22 PREFERRED PUMP & EQUI` |
| `amount` stored as text | all rows | `$450.00 ` — currency symbol, trailing space |
| `date` as US-format timestamp | all dated rows | `7/19/2014 0:00` |
| Credits | 14,518 | negative or parenthesized amounts |

Dated rows run from **2014-04-02 to 2014-12-31**. Used by [Module 5](../part-1-ai-assisted-reproducible-workflows/module-05-data-validation-and-reproducibility/walkthrough.md), where the point is that a `pandera` schema catches these before an analysis quietly averages over them.

## `transactions_wcategory.csv`

The same transactions, cleaned and categorized: **214,698 rows**, columns `date`, `amount`, `vendor`, `category`.

This is exactly the dated subset of `transactions.csv` — drop its 222,881 undated rows and you get these 214,698 — with ISO dates (`2014-07-19`), numeric amounts (`450.0`), and a merchant category added. There are **403 distinct categories** (`NEWS DEALERS AND NEWSSTANDS`, `BOOK STORES`, …) across **48,360 vendors**. Two quirks survive the cleaning: 2 rows have a blank `category`, and one "category", `Cosmopolitan of Las Vegas` (19 rows), is a merchant name rather than a merchant category.

Used by [Module 6](../part-1-ai-assisted-reproducible-workflows/module-06-ai-assisted-coding-and-debugging/walkthrough.md), whose prompts assume a `category` column, and by Module 7's grouping example.

## `general_2016_2018.csv`

Single audits of federal award recipients, from the **Federal Audit Clearinghouse**: **110,948 rows**, **47 columns**, audit years 2016–2018 (roughly 37,000 each).

One row per audit submission. Alongside the identifying fields it carries the findings an analysis would actually use: `is_going_concern_included`, `is_internal_control_material_weakness_disclosed`, `is_material_noncompliance_disclosed`, `total_amount_expended`, `dollar_threshold`, `is_low_risk_auditee`, `entity_type`, `audit_type`, and the opinion fields `gaap_results` and `sp_framework_opinions`.

Used by [Module 7](../part-1-ai-assisted-reproducible-workflows/module-07-mini-project-reproducible-analysis-repo/README.md) as the mini-project dataset.

**The contact columns have been removed.** The Clearinghouse publishes 64 columns, including the names, email addresses, phone numbers, street addresses and ZIP codes of the people who certified and prepared each audit. Those 17 columns are dropped here. The 9 `auditor_firm_name` values that held an email address, typed into the wrong field, are blanked. Entity names, EINs, cities and states remain. The data was public record before we removed anything. We removed it anyway because Module 4 asks you to keep exactly this kind of field out of a repository and out of a prompt, and "public" isn't the same as "fine to paste anywhere."
