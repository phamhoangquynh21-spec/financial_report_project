# Data

The source data is **not included** in this repository. The report expects eight CSV files (UTF-8, comma-delimited, header row) in one folder:

| File | Rows (FY2023) | Columns | Role |
|---|---|---|---|
| `Fact.csv` | 580 | `Date`, `Amount, $`, `Business Line ID`, `Subgroup ID`, `Group ID`, `Revenue or expense` | Monthly ledger postings. Revenue is positive and expenses are negative. One posting date per month (month-end). |
| `Calendar.csv` | — | `Date`, `Year`, `Quarter`, `Month`, `Month in text`, `Week`, `Day` | Date dimension |
| `Business Line.csv` | 3 | `Business Line ID`, `Business Line` | 1 Nutrition and Food Supplements, 2 Sports Equipment, 3 Sportswear |
| `Group.csv` | 4 | `Orden`, `Group ID`, `Group` | 1 Revenue, 2 Opex, 3 COGS, 4 Interest and tax |
| `Subgroup.csv` | — | `Subgroup`, `Subgroup ID`, `Group ID` | P&L line items (e.g. salaries, rent, materials) |
| `Headlines.csv` | 7 | `Orden`, `Headline`, `Type`, `Detail` | Row layout for the P&L statement |
| `KPI.csv` | — | `Name`, `Order`, `Leyend` | KPI labels |
| `Ratios.csv` | — | `Orden`, `Ratios` | Ratio labels |

Column names are listed exactly as they appear in the source files, including the Spanish `Orden` and the misspelled `Leyend`. Power Query renames them on load; see [../docs/data_cleaning.md](../docs/data_cleaning.md).

**To reconnect:** open the project, go to **Transform data → Data source settings**, and use **Change Source** on each file to point at your folder. Then select **Refresh**.

Industry benchmarks (`dim_benchmark`) do not need a file. They are entered directly in the model as a DAX table, sourced from Damodaran, *Margins by Sector (US)*, January 2026: https://pages.stern.nyu.edu/~adamodar/New_Home_Page/datafile/margin.html
