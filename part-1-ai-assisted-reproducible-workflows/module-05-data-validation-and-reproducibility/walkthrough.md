# Module 5 — Walkthrough: Data Validation and Reproducibility

This module works on `transactions.csv`, a raw extract of card transactions. Copy it into your project's `data/raw/` (the command is in [datasets/README.md](../../datasets/README.md)).

Run §1–4 in your notebook, `notebooks/explore.ipynb` (Module 4 moved it there). The notebook runs from its own folder, so the data is at `../data/raw/transactions.csv`. §5–6 are `pytest` tests, which live in files under `tests/` and run from the terminal.

## 1. Look, then write inline asserts

Load the file and look at it before you analyze anything:

```python
import pandas as pd

df = pd.read_csv("../data/raw/transactions.csv")
df.head()
```

Two problems show up straight away. `date` is a US-format timestamp (`7/19/2014 0:00`), and `amount` is text (`$450.00 `), with a currency symbol, a trailing space, and parentheses for credits (`($45.10)`). Pandas can't compare or add text as numbers, so convert both columns first:

```python
df["date"] = pd.to_datetime(df["date"], format="%m/%d/%Y %H:%M")

s = df["amount"].str.strip()
is_neg = s.str.startswith("(")
vals = s.str.replace(r"[$,()]", "", regex=True).str.strip().astype(float)
df["amount"] = vals.mask(is_neg, -vals)
```

Blank dates become `NaT` ("not a time", the datetime version of a missing value). A date that is present but in a different format raises an error instead of being guessed at. That's what you want: don't pass `errors="coerce"` here, because it would quietly turn every unparseable date into `NaT` too, and you would never learn they existed.

Now write down what you *expect* about the data:

```python
assert len(df) > 0, "transactions file is empty"
assert {"date", "amount", "vendor"}.issubset(df.columns), \
    f"missing required columns; got {df.columns.tolist()}"
assert df["amount"].notna().all(), "amount column has nulls"
assert (df["amount"] >= 0).all(), "negative amounts not allowed"
```

The last assert fails, and it's supposed to. That's the point of writing your expectations down: the file has negative amounts, and now you know before any number reaches your manager. If any assertion fails, the code stops *immediately* with a clear message, which is far better than letting bad data flow silently into a final number.

But an assert stops at the *first* failure. You've learned there are negatives; you haven't learned whether anything else is wrong too.

## 2. Step up to a `pandera` schema

`assert` is fine for one-off checks. For real data, use a schema: it checks every rule and reports every failure at once.

```bash
uv add pandera
```

Start by writing the strict version, with every rule you'd like to be true:

```python
import pandera.pandas as pa
from pandera.typing import Series

class TransactionSchema(pa.DataFrameModel):
    date: Series[pa.DateTime]                 # every row has a date
    amount: Series[float] = pa.Field(ge=0)    # no negative amounts
    vendor: Series[str] = pa.Field(str_length={"min_value": 1})

try:
    TransactionSchema.validate(df, lazy=True)
except pa.errors.SchemaErrors as err:
    print(err.failure_cases.groupby(["column", "check"]).size())
```

`lazy=True` tells `pandera` to collect every failure instead of stopping at the first one. The report has two lines:

- **`date` / `not_nullable`: 222,881 rows.** Half the file has no date. Your asserts never checked this.
- **`amount` / `greater_than_or_equal_to(0)`: 14,518 rows.** These are the negatives the assert caught.

Each failure needs a decision, and the schema can't make it for you:

- **Negative amounts.** In card data, a negative is a credit: a refund or a reversal. If you're measuring spend net of refunds, they belong in the data, so drop the `ge=0` rule. If a negative would be impossible in your setting, keep the rule and go find out where they came from.
- **Missing dates.** A row with no date can't be put in a month, so a monthly analysis can't use it. Drop those rows *on purpose*, and record how many you dropped. Don't make `date` nullable instead: the schema would pass, and then half the data would silently fall out of every monthly total later.

Here are both decisions as code. The schema now describes the data you've *decided* to analyze:

```python
undated = df["date"].isna()
print(f"dropping {undated.sum():,} of {len(df):,} rows with no date")
df = df[~undated]

class TransactionSchema(pa.DataFrameModel):
    date: Series[pa.DateTime]     # still required: undated rows were dropped above, on purpose
    amount: Series[float]         # credits allowed: negatives are refunds
    vendor: Series[str] = pa.Field(str_length={"min_value": 1})

TransactionSchema.validate(df, lazy=True)
```

It passes, with 214,698 rows left. Both choices, and the 222,881 dropped rows, belong in your write-up's caveats (Module 7's `results.md`). If the data ever violates the schema, `pandera` reports exactly which row, which column, and which rule.

*Optional further reading: [pandera Quickstart](https://pandera.readthedocs.io/en/stable/) — the library's own reference for lightweight schema validation on pandas DataFrames.*

## 3. Set random seeds (everywhere)

If your analysis uses any randomness — sampling, train/test split, simulation — set the seed at the top of the script:

```python
import random
import numpy as np

random.seed(42)
np.random.seed(42)

np.random.rand(3)
```

Run the cell several times: you get the same three numbers every time. Comment out the two `seed` lines and run it again: the numbers change on every run. The same holds for a whole analysis. Run it twice, and the output should be identical bit-for-bit. If it isn't, you have unseeded randomness somewhere — find it.

*Optional further reading: [The Turing Way — Guide for Reproducible Research](https://the-turing-way.netlify.app/) — broad community guide on reproducibility; its Reproducible Research chapter is the entry point for the practices in this module.*

## 4. Sort before you aggregate

```python
df = df.sort_values(["date", "vendor"]).reset_index(drop=True)
```

Pandas' groupby and merge can produce different row orders on different runs. Sorting at known checkpoints makes your output deterministic.

## 5. Write your first `pytest` test

```bash
uv add --dev pytest
mkdir -p tests
touch tests/__init__.py tests/test_analysis.py
```

`tests/__init__.py` makes `tests` a package, so pytest puts the project root on the import path and `from src.analysis import …` works. Delete it and `uv run pytest` fails with `ModuleNotFoundError: No module named 'src'`.

The test imports `summary_stats` from `src/analysis.py`. If yours is still in `main.py`, move it now (Module 4 §2 shows how). The test also pins down what the function returns: one row per numeric column. If yours returns one row per statistic instead, which is what `df.agg(["mean", "median"])` does, transpose it:

```python
# src/analysis.py
def summary_stats(df):
    """Mean and median of each numeric column, one row per column."""
    return df.select_dtypes("number").agg(["mean", "median"]).T
```

In `tests/test_analysis.py`:

```python
import pandas as pd
from src.analysis import summary_stats   # written in Module 3, moved into src/ in Module 4

def test_summary_stats_returns_a_row_per_column():
    df = pd.DataFrame({"a": [1, 2, 3], "b": [4, 5, 6], "c": [7, 8, 9]})
    result = summary_stats(df)
    assert len(result) == 3
```

Three columns, not two, on purpose. With two, a function that returns one row per *statistic* (mean, median) would also have length 2 and pass for the wrong reason.

Run:

```bash
uv run pytest
```

Green dots = passing. If not, fix the function or the test.

*Optional further reading: [pytest — Get Started](https://docs.pytest.org/en/stable/getting-started.html) — pytest's own quickstart; the first ten minutes gets you most of the value.*

## 6. Snapshot test for analysis results

For larger analyses, snapshot the answer so a regression breaks the test loudly. This example assumes a `compute_monthly_totals(df)` function — if you don't have one yet, add a minimal version to `src/analysis.py` first:

```python
# src/analysis.py
def compute_monthly_totals(df):
    """Total `amount` per calendar month."""
    return (
        df.assign(month=df["date"].dt.to_period("M").astype(str))
          .groupby("month", as_index=False)["amount"].sum()
    )
```

Then create the fixtures directory and the test:

```bash
mkdir -p tests/fixtures
```

Write `tests/fixtures/sample_transactions.csv` by hand, a few rows with ISO dates and plain numbers:

```
date,amount,vendor
2014-07-03,120.00,OFFICE DEPOT
2014-07-19,-20.00,OFFICE DEPOT
2014-08-02,45.50,AMAZON.COM
```

Don't paste rows from `data/raw/transactions.csv`. `compute_monthly_totals` expects data that has already been cleaned the way §1 cleans it. Given a raw `$450.00 ` string in `amount`, `.sum()` would glue the text together instead of adding the numbers.

```python
# tests/test_monthly_totals_snapshot.py
import pandas as pd
from src.analysis import compute_monthly_totals

def test_monthly_totals_match_snapshot():
    df = pd.read_csv("tests/fixtures/sample_transactions.csv", parse_dates=["date"])
    result = compute_monthly_totals(df)
    expected = pd.read_csv("tests/fixtures/expected_monthly_totals.csv")
    pd.testing.assert_frame_equal(
        result.reset_index(drop=True),
        expected.reset_index(drop=True),
    )
```

Generate the expected file once by hand (after you've eyeballed the result and confirmed it's right), commit it, and any future change that breaks the answer will fail this test. (These fixtures live under `tests/fixtures/`, not `data/`, so the Module 4 `.gitignore` data rules don't touch them — `git add` commits them normally.)

> `assert_frame_equal` compares **dtypes**, not just values. Reading `expected` back from a CSV can change a column's dtype (an `Int64` or a `Period` column can come back as `int64`/`object`), so the test can fail on dtype even when the numbers match. If that happens, cast the columns to plain `str`/`float` before saving the snapshot, or pass `check_dtype=False`.

## You're done if…

- [ ] Every input dataset in your project is validated by `pandera` or `assert`.
- [ ] All randomness is seeded.
- [ ] You have a `tests/` directory with at least 3 passing tests.
- [ ] `uv run pytest` is green.
