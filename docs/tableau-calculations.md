# Tableau calculated fields

The formulas below use the exact column names from `data/saas_revenue.csv`.

## Total Revenue

```tableau
SUM([revenue_amount])
```

## Paid Users Count

```tableau
COUNTD([user_id])
```

## Average Revenue Per Paid User — ARPPU

```tableau
IF COUNTD([user_id]) > 0 THEN
    SUM([revenue_amount]) / COUNTD([user_id])
END
```

## User First Payment Date

```tableau
{{ FIXED [user_id] : MIN([payment_date]) }}
```

## User First Payment Month

```tableau
DATETRUNC('month', [User First Payment Date])
```

## Payment Month

```tableau
DATETRUNC('month', [payment_date])
```

## Payment Month Number

```tableau
DATEDIFF(
    'month',
    [User First Payment Month],
    [Payment Month]
)
```

## New MRR

Create the following row-level field:

```tableau
IF [Payment Month] = [User First Payment Month] THEN
    [revenue_amount]
END
```

Use `SUM([New MRR])` in the worksheet.

## Month-over-month Revenue Change

```tableau
IF ABS(LOOKUP(SUM([revenue_amount]), -1)) > 0 THEN
    (
        SUM([revenue_amount])
        - LOOKUP(SUM([revenue_amount]), -1)
    )
    /
    ABS(LOOKUP(SUM([revenue_amount]), -1))
END
```

Recommended configuration:

- Put month of `payment_date` on Columns.
- Put `SUM(revenue_amount)` and the calculation on Rows.
- Use a dual axis.
- Set **Compute Using** to month of `payment_date`.
- Format the calculation as a percentage.

## Cohort Revenue Change vs First Month

```tableau
(
    SUM([revenue_amount])
    /
    WINDOW_MAX(
        IF MIN([Payment Month Number]) = 0 THEN
            SUM([revenue_amount])
        END
    )
)
- 1
```

Recommended cohort table setup:

- Rows: `User First Payment Month`
- Columns: `Payment Month Number`
- Text: `SUM(revenue_amount)`
- Color: `Cohort Revenue Change vs First Month`
- Compute using: `Payment Month Number`
- Restart every: `User First Payment Month`
- Format the color calculation as a percentage
