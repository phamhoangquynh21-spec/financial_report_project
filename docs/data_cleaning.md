# Data cleaning log

All cleaning is done in Power Query, so it reruns on every refresh. After each step I reconciled the results back to the source: the row count stayed at **580 rows** and FY2023 **net profit stayed at $4,314,120**.

| # | Table | Issue found | Fix | Why it matters |
|---|---|---|---|---|
| 1 | `dim_business_line` | Names contained non-breaking spaces (U+00A0), stray whitespace and inconsistent case | Replaced U+00A0 with a normal space, trimmed, applied title case, and kept "and" in lowercase | Slicers and labels showed near-duplicate or ugly values |
| 2 | `fact_financial` | Column named `Amount, $` | Renamed to `Amount` | The comma breaks DAX references and readability |
| 3 | `dim_kpi` | Column misspelled `Leyend` | Renamed to `Legend` | Clean, professional field list |
| 4 | `dim_group`, `dim_headline`, `dim_ratios` | Column named `Orden` (Spanish) | Renamed to `Order` | Consistent English naming |
| 5 | `dim_subgroup` + `fact_financial` | Subgroup 9 "Other" was used under **both** Opex (Group 2) and COGS (Group 3) | Relabelled ID 9 as "Other Opex", added ID 16 "Other COGS", and remapped fact rows with Group 3 + Subgroup 9 to ID 16 ($263,700) | Without this, $263,700 of COGS would show up under an Opex label in subgroup breakdowns |
| 6 | Model | Helper columns (`Week`, `Day`) and numeric IDs were visible and summed by default | Hid the helper columns and set numeric attributes to "Don't summarize" | Stops meaningless sums in visuals |
| 7 | Model | Month names sorted alphabetically | Sort by `Month`; P&L lines sorted by `Order` | Months and P&L lines display in the right order |
| 8 | `dim_calendar` | Not marked as a date table | Marked as the date table | Required for reliable time intelligence (`DATEADD`, YTD) |

## Checked and kept as-is

- **Sports Equipment "Other income".** Jan–Apr each contain two "Other income" rows with different values. They are separate postings, not duplicates, so they were kept.
- **Business line IDs.** The original data dictionary listed the IDs in a different order from the data. The data was treated as the source of truth: 1 Nutrition and Food Supplements, 2 Sports Equipment, 3 Sportswear.
- **Interest and tax.** These are combined in one group in the source and cannot be separated. As a result, `EBIT Cover (x)` = EBIT ÷ (Interest + Tax) and is labelled that way, not as a true interest coverage ratio.

## Known limitation

The CSV paths are hard-coded in each Power Query source. A `DataFolder` parameter would make the project portable; this is listed as a next step in the main README.
