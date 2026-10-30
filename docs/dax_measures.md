# DAX measure catalogue

All 159 measures live in the `*Measures` table. They are generated from the model's TMDL definition, so this list matches the report exactly.

Conventions: expenses are stored negative and flipped once in `01 Base`; every ratio uses `DIVIDE` to avoid divide-by-zero; month-on-month measures return blank unless exactly one month is in context (`HASONEVALUE`).

| Folder | Measures | What it holds |
|---|---|---|
| [01 Base](#01-base) | 6 | Signed ledger amounts turned into positive base figures. The sign flip happens only here. |
| [02 P&L](#02-pl) | 6 | P&L subtotals and the single measure that renders the P&L statement. |
| [03 Ratios](#03-ratios) | 10 | Margins and cost ratios, plus SWITCH-based selectors. |
| [04 Time](#04-time) | 26 | Prior-month, month-on-month, YTD and the FY2023 Jan→Dec trend labels. |
| [05 Card Deltas](#05-card-deltas) | 24 | Change vs prior month for the KPI cards: value, label and colour. |
| [06 Break-even](#06-break-even) | 11 | Break-even revenue, margin of safety and the revenue-shock scenario. |
| [07 Mix](#07-mix) | 7 | Shares of revenue, profit and cost. |
| [08 Display](#08-display) | 36 | Dynamic text and conditional-format colours (hex strings). |
| [09 Benchmark](#09-benchmark) | 24 | Revenue-weighted industry benchmarks and gaps. |
| [10 Cards](#10-cards) | 9 | Card-ready versions of key values with compact formats. |

## 01 Base

Signed ledger amounts turned into positive base figures. The sign flip happens only here.

<details><summary><code>Total Revenue</code> — Sum of revenue postings (Group 1)</summary>

```dax
Total Revenue =
// Revenue is stored positive
CALCULATE ( SUM ( fact_financial[Amount] ), fact_financial[Group ID] = 1 )
```
Format: `\$#,0`

</details>

<details><summary><code>Total COGS</code> — Cost of goods sold as a positive number (Group 3)</summary>

```dax
Total COGS =
// Expenses are stored negative, flip once here
- CALCULATE ( SUM ( fact_financial[Amount] ), fact_financial[Group ID] = 3 )
```
Format: `\$#,0`

</details>

<details><summary><code>Total Opex</code> — Operating expenses as a positive number (Group 2)</summary>

```dax
Total Opex =
- CALCULATE ( SUM ( fact_financial[Amount] ), fact_financial[Group ID] = 2 )
```
Format: `\$#,0`

</details>

<details><summary><code>Total Interest & Tax</code> — Interest and tax as a positive number (Group 4)</summary>

```dax
Total Interest & Tax =
- CALCULATE ( SUM ( fact_financial[Amount] ), fact_financial[Group ID] = 4 )
```
Format: `\$#,0`

</details>

<details><summary><code>Total Cost</code> — COGS + Opex + Interest & Tax</summary>

```dax
Total Cost =
[Total COGS] + [Total Opex] + [Total Interest & Tax]
```
Format: `\$#,0`

</details>

<details><summary><code>Amount (Abs)</code> — Absolute amount for cost breakdown visuals by subgroup</summary>

```dax
Amount (Abs) =
ABS ( SUM ( fact_financial[Amount] ) )
```
Format: `\$#,0`

</details>

## 02 P&L

P&L subtotals and the single measure that renders the P&L statement.

<details><summary><code>Gross Profit</code> — Revenue - COGS</summary>

```dax
Gross Profit =
[Total Revenue] - [Total COGS]
```
Format: `\$#,0;-\$#,0`

</details>

<details><summary><code>EBIT</code> — Gross Profit - Opex</summary>

```dax
EBIT =
[Gross Profit] - [Total Opex]
```
Format: `\$#,0;-\$#,0`

</details>

<details><summary><code>Net Profit</code> — EBIT - Interest & Tax</summary>

```dax
Net Profit =
[EBIT] - [Total Interest & Tax]
```
Format: `\$#,0;-\$#,0`

</details>

<details><summary><code>P&L Value</code> — Income-statement line value driven by dim_headline; costs shown negative</summary>

```dax
P&L Value =
// Picks the P&L line from dim_headline; subtotal lines blank out when a subgroup is in scope
VAR _line = SELECTEDVALUE ( dim_headline[Order] )
VAR _isSubtotal = SELECTEDVALUE ( dim_headline[Detail] ) = 0
VAR _value =
    SWITCH (
        _line,
        1, [Total Revenue],
        2, - [Total COGS],
        3, [Gross Profit],
        4, - [Total Opex],
        5, [EBIT],
        6, - [Total Interest & Tax],
        7, [Net Profit]
    )
RETURN
    IF ( ISINSCOPE ( dim_subgroup[Subgroup] ) && _isSubtotal, BLANK (), _value )
```
Format: `\$#,0,.0"K";(\$#,0,.0"K")`

</details>

<details><summary><code>P&L % of Revenue</code> — P&L line as % of revenue (subgroup filter removed from denominator)</summary>

```dax
P&L % of Revenue =
DIVIDE ( [P&L Value], CALCULATE ( [Total Revenue], REMOVEFILTERS ( dim_subgroup ) ) )
```
Format: `0.0%;-0.0%`

</details>

<details><summary><code>Bridge Value</code> — Waterfall input: detail P&L lines only; the visual total equals Net Profit</summary>

```dax
Bridge Value =
// On a single headline: return only detail lines (Revenue, COGS, Opex, I&T). With no headline filter (waterfall total): Net Profit
IF (
    HASONEVALUE ( dim_headline[Order] ),
    IF ( SELECTEDVALUE ( dim_headline[Detail] ) = 1, [P&L Value] ),
    [Net Profit]
)
```
Format: `\$#,0;-\$#,0`

</details>

## 03 Ratios

Margins and cost ratios, plus SWITCH-based selectors.

<details><summary><code>Gross Profit Margin %</code> — Gross Profit / Revenue</summary>

```dax
Gross Profit Margin % =
DIVIDE ( [Gross Profit], [Total Revenue] )
```
Format: `0.00%;-0.00%;0.00%`

</details>

<details><summary><code>COGS Ratio %</code> — COGS / Revenue</summary>

```dax
COGS Ratio % =
DIVIDE ( [Total COGS], [Total Revenue] )
```
Format: `0.0%`

</details>

<details><summary><code>Opex Ratio %</code> — Opex / Revenue</summary>

```dax
Opex Ratio % =
DIVIDE ( [Total Opex], [Total Revenue] )
```
Format: `0.00%;-0.00%;0.00%`

</details>

<details><summary><code>EBIT Margin %</code> — EBIT / Revenue</summary>

```dax
EBIT Margin % =
DIVIDE ( [EBIT], [Total Revenue] )
```
Format: `0.00%;-0.00%;0.00%`

</details>

<details><summary><code>Net Profit Margin %</code> — Net Profit / Revenue</summary>

```dax
Net Profit Margin % =
DIVIDE ( [Net Profit], [Total Revenue] )
```
Format: `0.00%;-0.00%;0.00%`

</details>

<details><summary><code>Interest & Tax % of Revenue</code> — Interest & Tax / Revenue</summary>

```dax
Interest & Tax % of Revenue =
DIVIDE ( [Total Interest & Tax], [Total Revenue] )
```
Format: `0.0%`

</details>

<details><summary><code>EBIT Cover (x)</code> — EBIT / (Interest + Tax). Interest and tax are combined in the source, so this is not a pure interest coverage ratio</summary>

```dax
EBIT Cover (x) =
DIVIDE ( [EBIT], [Total Interest & Tax] )
```
Format: `0.00"x";-0.00"x"`

</details>

<details><summary><code>KPI Selected Value</code> — Value of the KPI chosen in dim_kpi (defaults to Opex)</summary>

```dax
KPI Selected Value =
SWITCH (
    SELECTEDVALUE ( dim_kpi[Order], 1 ),
    1, [Total Opex],
    2, [Gross Profit],
    3, [EBIT],
    4, [Total Interest & Tax],
    5, [Net Profit]
)
```
Format: `\$#,0;-\$#,0`

</details>

<details><summary><code>KPI Selected Ratio</code> — Ratio paired with the KPI chosen in dim_kpi; format switches to x for cover</summary>

```dax
KPI Selected Ratio =
SWITCH (
    SELECTEDVALUE ( dim_kpi[Order], 1 ),
    1, [Opex Ratio %],
    2, [Gross Profit Margin %],
    3, [EBIT Margin %],
    4, [EBIT Cover (x)],
    5, [Net Profit Margin %]
)
```

</details>

<details><summary><code>Ratio Selected Value</code> — Ratio (as % of revenue) for the item chosen in dim_ratios</summary>

```dax
Ratio Selected Value =
SWITCH (
    SELECTEDVALUE ( dim_ratios[Order], 1 ),
    1, [Opex Ratio %],
    2, [Gross Profit Margin %],
    3, [EBIT Margin %],
    4, [Interest & Tax % of Revenue],
    5, [Net Profit Margin %]
)
```
Format: `0.0%;-0.0%`

</details>

## 04 Time

Prior-month, month-on-month, YTD and the FY2023 Jan→Dec trend labels.

<details><summary><code>Revenue PM</code> — Revenue in the previous month</summary>

```dax
Revenue PM =
CALCULATE ( [Total Revenue], DATEADD ( dim_calendar[Date], -1, MONTH ) )
```
Format: `\$#,0`

</details>

<details><summary><code>Revenue MoM %</code> — Month-over-month revenue change</summary>

```dax
Revenue MoM % =
// Only meaningful for a single month
VAR _pm = [Revenue PM]
RETURN IF ( HASONEVALUE ( dim_calendar[Month] ) && NOT ISBLANK ( _pm ), DIVIDE ( [Total Revenue] - _pm, _pm ) )
```
Format: `+0.0%;-0.0%;0.0%`

</details>

<details><summary><code>Net Profit PM</code> — Net profit in the previous month</summary>

```dax
Net Profit PM =
CALCULATE ( [Net Profit], DATEADD ( dim_calendar[Date], -1, MONTH ) )
```
Format: `\$#,0;-\$#,0`

</details>

<details><summary><code>Net Profit MoM %</code> — Month-over-month net profit change (vs absolute prior value)</summary>

```dax
Net Profit MoM % =
// Only meaningful for a single month
VAR _pm = [Net Profit PM]
RETURN IF ( HASONEVALUE ( dim_calendar[Month] ) && NOT ISBLANK ( _pm ), DIVIDE ( [Net Profit] - _pm, ABS ( _pm ) ) )
```
Format: `0.00%;-0.00%;0.00%`

</details>

<details><summary><code>Opex PM</code> — Opex in the previous month</summary>

```dax
Opex PM =
CALCULATE ( [Total Opex], DATEADD ( dim_calendar[Date], -1, MONTH ) )
```
Format: `\$#,0`

</details>

<details><summary><code>Opex MoM %</code> — Month-over-month opex change</summary>

```dax
Opex MoM % =
// Only meaningful for a single month
VAR _pm = [Opex PM]
RETURN IF ( HASONEVALUE ( dim_calendar[Month] ) && NOT ISBLANK ( _pm ), DIVIDE ( [Total Opex] - _pm, _pm ) )
```
Format: `+0.0%;-0.0%;0.0%`

</details>

<details><summary><code>Revenue YTD</code> — Year-to-date revenue</summary>

```dax
Revenue YTD =
TOTALYTD ( [Total Revenue], dim_calendar[Date] )
```
Format: `\$#,0`

</details>

<details><summary><code>Net Profit YTD</code> — Year-to-date net profit</summary>

```dax
Net Profit YTD =
TOTALYTD ( [Net Profit], dim_calendar[Date] )
```
Format: `\$#,0;-\$#,0`

</details>

<details><summary><code>Latest Month Date</code> — Last month-end posting date in the current selection</summary>

```dax
Latest Month Date =
MAX ( fact_financial[Date] )
```
Format: `mmm yyyy`

</details>

<details><summary><code>Latest Month Name</code> — Short name of the last month in the selection</summary>

```dax
Latest Month Name =
FORMAT ( [Latest Month Date], "mmm" )
```

</details>

<details><summary><code>Prior Month Name</code> — Short name of the month before the last month in the selection</summary>

```dax
Prior Month Name =
FORMAT ( EOMONTH ( [Latest Month Date], -1 ), "mmm" )
```

</details>

<details><summary><code>Gross Profit MoM %</code> — Month-over-month change of [Gross Profit] (percent); single month only</summary>

```dax
Gross Profit MoM % =
VAR _cur = [Gross Profit]
VAR _pm = CALCULATE ( [Gross Profit], DATEADD ( dim_calendar[Date], -1, MONTH ) )
RETURN IF ( HASONEVALUE ( dim_calendar[Month] ) && NOT ISBLANK ( _pm ), DIVIDE ( _cur - _pm, ABS ( _pm ) ) )
```
Format: `+0.0%;-0.0%;0.0%`

</details>

<details><summary><code>EBIT MoM %</code> — Month-over-month change of [EBIT] (percent); single month only</summary>

```dax
EBIT MoM % =
VAR _cur = [EBIT]
VAR _pm = CALCULATE ( [EBIT], DATEADD ( dim_calendar[Date], -1, MONTH ) )
RETURN IF ( HASONEVALUE ( dim_calendar[Month] ) && NOT ISBLANK ( _pm ), DIVIDE ( _cur - _pm, ABS ( _pm ) ) )
```
Format: `+0.0%;-0.0%;0.0%`

</details>

<details><summary><code>COGS MoM %</code> — Month-over-month change of [Total COGS] (percent); single month only</summary>

```dax
COGS MoM % =
VAR _cur = [Total COGS]
VAR _pm = CALCULATE ( [Total COGS], DATEADD ( dim_calendar[Date], -1, MONTH ) )
RETURN IF ( HASONEVALUE ( dim_calendar[Month] ) && NOT ISBLANK ( _pm ), DIVIDE ( _cur - _pm, ABS ( _pm ) ) )
```
Format: `+0.0%;-0.0%;0.0%`

</details>

<details><summary><code>Labor MoM %</code> — Month-over-month change of [Labor Cost] (percent); single month only</summary>

```dax
Labor MoM % =
VAR _cur = [Labor Cost]
VAR _pm = CALCULATE ( [Labor Cost], DATEADD ( dim_calendar[Date], -1, MONTH ) )
RETURN IF ( HASONEVALUE ( dim_calendar[Month] ) && NOT ISBLANK ( _pm ), DIVIDE ( _cur - _pm, ABS ( _pm ) ) )
```
Format: `+0.0%;-0.0%;0.0%`

</details>

<details><summary><code>Net Margin MoM pp</code> — Month-over-month change of [Net Profit Margin %] (percentage points); single month only</summary>

```dax
Net Margin MoM pp =
VAR _cur = [Net Profit Margin %]
VAR _pm = CALCULATE ( [Net Profit Margin %], DATEADD ( dim_calendar[Date], -1, MONTH ) )
RETURN IF ( HASONEVALUE ( dim_calendar[Month] ) && NOT ISBLANK ( _pm ), _cur - _pm )
```
Format: `+0.0%;-0.0%;0.0%`

</details>

<details><summary><code>Opex Ratio MoM pp</code> — Month-over-month change of [Opex Ratio %] (percentage points); single month only</summary>

```dax
Opex Ratio MoM pp =
VAR _cur = [Opex Ratio %]
VAR _pm = CALCULATE ( [Opex Ratio %], DATEADD ( dim_calendar[Date], -1, MONTH ) )
RETURN IF ( HASONEVALUE ( dim_calendar[Month] ) && NOT ISBLANK ( _pm ), _cur - _pm )
```
Format: `+0.0%;-0.0%;0.0%`

</details>

<details><summary><code>Revenue FY Trend Label</code> — Compound monthly growth rate Jan to Dec FY2023 (linear fallback when Jan or Dec is not positive), plus count of months that rose vs the prior month. Ignores the Month slicer.</summary>

```dax
Revenue FY Trend Label =
VAR _f = CALCULATE ( [Total Revenue], REMOVEFILTERS ( dim_calendar ), dim_calendar[Month] = 1 )
VAR _l = CALCULATE ( [Total Revenue], REMOVEFILTERS ( dim_calendar ), dim_calendar[Month] = 12 )
VAR _rate = IF ( _f > 0 && _l > 0, POWER ( DIVIDE ( _l, _f ), 1 / 11 ) - 1, DIVIDE ( _l - _f, ABS ( _f ) ) / 11 )
VAR _ups = CALCULATE ( COUNTROWS ( FILTER ( VALUES ( dim_calendar[Month] ), [Revenue MoM %] > 0 ) ), REMOVEFILTERS ( dim_calendar ) )
RETURN
    IF ( ISBLANK ( _f ) || ISBLANK ( _l ), BLANK (),
        "Jan→Dec " & IF ( _rate >= 0, "▲ ", "▼ " ) & FORMAT ( ABS ( _rate ), "0.0%" ) & "/mo · " & COALESCE ( _ups, 0 ) & "/11 up" )
```

</details>

<details><summary><code>Gross Profit FY Trend Label</code> — Compound monthly growth rate Jan to Dec FY2023 (linear fallback when Jan or Dec is not positive), plus count of months that rose vs the prior month. Ignores the Month slicer.</summary>

```dax
Gross Profit FY Trend Label =
VAR _f = CALCULATE ( [Gross Profit], REMOVEFILTERS ( dim_calendar ), dim_calendar[Month] = 1 )
VAR _l = CALCULATE ( [Gross Profit], REMOVEFILTERS ( dim_calendar ), dim_calendar[Month] = 12 )
VAR _rate = IF ( _f > 0 && _l > 0, POWER ( DIVIDE ( _l, _f ), 1 / 11 ) - 1, DIVIDE ( _l - _f, ABS ( _f ) ) / 11 )
VAR _ups = CALCULATE ( COUNTROWS ( FILTER ( VALUES ( dim_calendar[Month] ), [Gross Profit MoM %] > 0 ) ), REMOVEFILTERS ( dim_calendar ) )
RETURN
    IF ( ISBLANK ( _f ) || ISBLANK ( _l ), BLANK (),
        "Jan→Dec " & IF ( _rate >= 0, "▲ ", "▼ " ) & FORMAT ( ABS ( _rate ), "0.0%" ) & "/mo · " & COALESCE ( _ups, 0 ) & "/11 up" )
```

</details>

<details><summary><code>Opex FY Trend Label</code> — Compound monthly growth rate Jan to Dec FY2023 (linear fallback when Jan or Dec is not positive), plus count of months that rose vs the prior month. Ignores the Month slicer.</summary>

```dax
Opex FY Trend Label =
VAR _f = CALCULATE ( [Total Opex], REMOVEFILTERS ( dim_calendar ), dim_calendar[Month] = 1 )
VAR _l = CALCULATE ( [Total Opex], REMOVEFILTERS ( dim_calendar ), dim_calendar[Month] = 12 )
VAR _rate = IF ( _f > 0 && _l > 0, POWER ( DIVIDE ( _l, _f ), 1 / 11 ) - 1, DIVIDE ( _l - _f, ABS ( _f ) ) / 11 )
VAR _ups = CALCULATE ( COUNTROWS ( FILTER ( VALUES ( dim_calendar[Month] ), [Opex MoM %] > 0 ) ), REMOVEFILTERS ( dim_calendar ) )
RETURN
    IF ( ISBLANK ( _f ) || ISBLANK ( _l ), BLANK (),
        "Jan→Dec " & IF ( _rate >= 0, "▲ ", "▼ " ) & FORMAT ( ABS ( _rate ), "0.0%" ) & "/mo · " & COALESCE ( _ups, 0 ) & "/11 up" )
```

</details>

<details><summary><code>EBIT FY Trend Label</code> — Compound monthly growth rate Jan to Dec FY2023 (linear fallback when Jan or Dec is not positive), plus count of months that rose vs the prior month. Ignores the Month slicer.</summary>

```dax
EBIT FY Trend Label =
VAR _f = CALCULATE ( [EBIT], REMOVEFILTERS ( dim_calendar ), dim_calendar[Month] = 1 )
VAR _l = CALCULATE ( [EBIT], REMOVEFILTERS ( dim_calendar ), dim_calendar[Month] = 12 )
VAR _rate = IF ( _f > 0 && _l > 0, POWER ( DIVIDE ( _l, _f ), 1 / 11 ) - 1, DIVIDE ( _l - _f, ABS ( _f ) ) / 11 )
VAR _ups = CALCULATE ( COUNTROWS ( FILTER ( VALUES ( dim_calendar[Month] ), [EBIT MoM %] > 0 ) ), REMOVEFILTERS ( dim_calendar ) )
RETURN
    IF ( ISBLANK ( _f ) || ISBLANK ( _l ), BLANK (),
        "Jan→Dec " & IF ( _rate >= 0, "▲ ", "▼ " ) & FORMAT ( ABS ( _rate ), "0.0%" ) & "/mo · " & COALESCE ( _ups, 0 ) & "/11 up" )
```

</details>

<details><summary><code>Net Profit FY Trend Label</code> — Compound monthly growth rate Jan to Dec FY2023 (linear fallback when Jan or Dec is not positive), plus count of months that rose vs the prior month. Ignores the Month slicer.</summary>

```dax
Net Profit FY Trend Label =
VAR _f = CALCULATE ( [Net Profit], REMOVEFILTERS ( dim_calendar ), dim_calendar[Month] = 1 )
VAR _l = CALCULATE ( [Net Profit], REMOVEFILTERS ( dim_calendar ), dim_calendar[Month] = 12 )
VAR _rate = IF ( _f > 0 && _l > 0, POWER ( DIVIDE ( _l, _f ), 1 / 11 ) - 1, DIVIDE ( _l - _f, ABS ( _f ) ) / 11 )
VAR _ups = CALCULATE ( COUNTROWS ( FILTER ( VALUES ( dim_calendar[Month] ), [Net Profit MoM %] > 0 ) ), REMOVEFILTERS ( dim_calendar ) )
RETURN
    IF ( ISBLANK ( _f ) || ISBLANK ( _l ), BLANK (),
        "Jan→Dec " & IF ( _rate >= 0, "▲ ", "▼ " ) & FORMAT ( ABS ( _rate ), "0.0%" ) & "/mo · " & COALESCE ( _ups, 0 ) & "/11 up" )
```

</details>

<details><summary><code>Net Margin FY Trend Label</code> — Average monthly change in percentage points, Jan to Dec FY2023, plus count of months that rose vs the prior month. Ignores the Month slicer.</summary>

```dax
Net Margin FY Trend Label =
VAR _f = CALCULATE ( [Net Profit Margin %], REMOVEFILTERS ( dim_calendar ), dim_calendar[Month] = 1 )
VAR _l = CALCULATE ( [Net Profit Margin %], REMOVEFILTERS ( dim_calendar ), dim_calendar[Month] = 12 )
VAR _rate = ( _l - _f ) / 11
VAR _ups = CALCULATE ( COUNTROWS ( FILTER ( VALUES ( dim_calendar[Month] ), [Net Margin MoM pp] > 0 ) ), REMOVEFILTERS ( dim_calendar ) )
RETURN
    IF ( ISBLANK ( _f ) || ISBLANK ( _l ), BLANK (),
        "Jan→Dec " & IF ( _rate >= 0, "▲ ", "▼ " ) & FORMAT ( ABS ( _rate ) * 100, "0.00" ) & " pp/mo · " & COALESCE ( _ups, 0 ) & "/11 up" )
```

</details>

<details><summary><code>COGS FY Trend Label</code> — Compound monthly growth rate Jan to Dec FY2023 (linear fallback when Jan or Dec is not positive), plus count of months that rose vs the prior month. Ignores the Month slicer.</summary>

```dax
COGS FY Trend Label =
VAR _f = CALCULATE ( [Total COGS], REMOVEFILTERS ( dim_calendar ), dim_calendar[Month] = 1 )
VAR _l = CALCULATE ( [Total COGS], REMOVEFILTERS ( dim_calendar ), dim_calendar[Month] = 12 )
VAR _rate = IF ( _f > 0 && _l > 0, POWER ( DIVIDE ( _l, _f ), 1 / 11 ) - 1, DIVIDE ( _l - _f, ABS ( _f ) ) / 11 )
VAR _ups = CALCULATE ( COUNTROWS ( FILTER ( VALUES ( dim_calendar[Month] ), [COGS MoM %] > 0 ) ), REMOVEFILTERS ( dim_calendar ) )
RETURN
    IF ( ISBLANK ( _f ) || ISBLANK ( _l ), BLANK (),
        "Jan→Dec " & IF ( _rate >= 0, "▲ ", "▼ " ) & FORMAT ( ABS ( _rate ), "0.0%" ) & "/mo · " & COALESCE ( _ups, 0 ) & "/11 up" )
```

</details>

<details><summary><code>Opex Ratio FY Trend Label</code> — Average monthly change in percentage points, Jan to Dec FY2023, plus count of months that rose vs the prior month. Ignores the Month slicer.</summary>

```dax
Opex Ratio FY Trend Label =
VAR _f = CALCULATE ( [Opex Ratio %], REMOVEFILTERS ( dim_calendar ), dim_calendar[Month] = 1 )
VAR _l = CALCULATE ( [Opex Ratio %], REMOVEFILTERS ( dim_calendar ), dim_calendar[Month] = 12 )
VAR _rate = ( _l - _f ) / 11
VAR _ups = CALCULATE ( COUNTROWS ( FILTER ( VALUES ( dim_calendar[Month] ), [Opex Ratio MoM pp] > 0 ) ), REMOVEFILTERS ( dim_calendar ) )
RETURN
    IF ( ISBLANK ( _f ) || ISBLANK ( _l ), BLANK (),
        "Jan→Dec " & IF ( _rate >= 0, "▲ ", "▼ " ) & FORMAT ( ABS ( _rate ) * 100, "0.00" ) & " pp/mo · " & COALESCE ( _ups, 0 ) & "/11 up" )
```

</details>

<details><summary><code>Labor FY Trend Label</code> — Compound monthly growth rate Jan to Dec FY2023 (linear fallback when Jan or Dec is not positive), plus count of months that rose vs the prior month. Ignores the Month slicer.</summary>

```dax
Labor FY Trend Label =
VAR _f = CALCULATE ( [Labor Cost], REMOVEFILTERS ( dim_calendar ), dim_calendar[Month] = 1 )
VAR _l = CALCULATE ( [Labor Cost], REMOVEFILTERS ( dim_calendar ), dim_calendar[Month] = 12 )
VAR _rate = IF ( _f > 0 && _l > 0, POWER ( DIVIDE ( _l, _f ), 1 / 11 ) - 1, DIVIDE ( _l - _f, ABS ( _f ) ) / 11 )
VAR _ups = CALCULATE ( COUNTROWS ( FILTER ( VALUES ( dim_calendar[Month] ), [Labor MoM %] > 0 ) ), REMOVEFILTERS ( dim_calendar ) )
RETURN
    IF ( ISBLANK ( _f ) || ISBLANK ( _l ), BLANK (),
        "Jan→Dec " & IF ( _rate >= 0, "▲ ", "▼ " ) & FORMAT ( ABS ( _rate ), "0.0%" ) & "/mo · " & COALESCE ( _ups, 0 ) & "/11 up" )
```

</details>

## 05 Card Deltas

Change vs prior month for the KPI cards: value, label and colour.

<details><summary><code>Revenue vs PM</code> — Latest month vs prior month % change in Revenue</summary>

```dax
Revenue vs PM =
VAR _d = [Latest Month Date]
VAR _cur = CALCULATE ( [Total Revenue], REMOVEFILTERS ( dim_calendar ), dim_calendar[Date] = _d )
VAR _prv = CALCULATE ( [Total Revenue], REMOVEFILTERS ( dim_calendar ), dim_calendar[Date] = EOMONTH ( _d, -1 ) )
RETURN IF ( NOT ISBLANK ( _prv ), DIVIDE ( _cur - _prv, ABS ( _prv ) ) )
```
Format: `+0.0%;-0.0%;0.0%`

</details>

<details><summary><code>Revenue vs PM Label</code> — Card subtitle: arrow + change in Revenue, latest vs prior month</summary>

```dax
Revenue vs PM Label =
VAR _c = [Revenue vs PM]
RETURN
    IF (
        ISBLANK ( _c ),
        "No prior month",
        [Latest Month Name] & IF ( _c >= 0, " ▲ ", " ▼ " ) & FORMAT ( ABS ( _c ), "0.0%" ) & " vs " & [Prior Month Name]
    )
```

</details>

<details><summary><code>Revenue vs PM Color</code> — Font colour for Revenue delta (increase is good)</summary>

```dax
Revenue vs PM Color =
VAR _c = [Revenue vs PM]
RETURN SWITCH ( TRUE (), ISBLANK ( _c ), "#5A6478", _c >= 0, "#2A9D8F", "#C8553D" )
```

</details>

<details><summary><code>Gross Profit vs PM</code> — Latest month vs prior month % change in Gross Profit</summary>

```dax
Gross Profit vs PM =
VAR _d = [Latest Month Date]
VAR _cur = CALCULATE ( [Gross Profit], REMOVEFILTERS ( dim_calendar ), dim_calendar[Date] = _d )
VAR _prv = CALCULATE ( [Gross Profit], REMOVEFILTERS ( dim_calendar ), dim_calendar[Date] = EOMONTH ( _d, -1 ) )
RETURN IF ( NOT ISBLANK ( _prv ), DIVIDE ( _cur - _prv, ABS ( _prv ) ) )
```
Format: `+0.0%;-0.0%;0.0%`

</details>

<details><summary><code>Gross Profit vs PM Label</code> — Card subtitle: arrow + change in Gross Profit, latest vs prior month</summary>

```dax
Gross Profit vs PM Label =
VAR _c = [Gross Profit vs PM]
RETURN
    IF (
        ISBLANK ( _c ),
        "No prior month",
        [Latest Month Name] & IF ( _c >= 0, " ▲ ", " ▼ " ) & FORMAT ( ABS ( _c ), "0.0%" ) & " vs " & [Prior Month Name]
    )
```

</details>

<details><summary><code>Gross Profit vs PM Color</code> — Font colour for Gross Profit delta (increase is good)</summary>

```dax
Gross Profit vs PM Color =
VAR _c = [Gross Profit vs PM]
RETURN SWITCH ( TRUE (), ISBLANK ( _c ), "#5A6478", _c >= 0, "#2A9D8F", "#C8553D" )
```

</details>

<details><summary><code>Opex vs PM</code> — Latest month vs prior month % change in Opex</summary>

```dax
Opex vs PM =
VAR _d = [Latest Month Date]
VAR _cur = CALCULATE ( [Total Opex], REMOVEFILTERS ( dim_calendar ), dim_calendar[Date] = _d )
VAR _prv = CALCULATE ( [Total Opex], REMOVEFILTERS ( dim_calendar ), dim_calendar[Date] = EOMONTH ( _d, -1 ) )
RETURN IF ( NOT ISBLANK ( _prv ), DIVIDE ( _cur - _prv, ABS ( _prv ) ) )
```
Format: `+0.0%;-0.0%;0.0%`

</details>

<details><summary><code>Opex vs PM Label</code> — Card subtitle: arrow + change in Opex, latest vs prior month</summary>

```dax
Opex vs PM Label =
VAR _c = [Opex vs PM]
RETURN
    IF (
        ISBLANK ( _c ),
        "No prior month",
        [Latest Month Name] & IF ( _c >= 0, " ▲ ", " ▼ " ) & FORMAT ( ABS ( _c ), "0.0%" ) & " vs " & [Prior Month Name]
    )
```

</details>

<details><summary><code>Opex vs PM Color</code> — Font colour for Opex delta (increase is bad)</summary>

```dax
Opex vs PM Color =
VAR _c = [Opex vs PM]
RETURN SWITCH ( TRUE (), ISBLANK ( _c ), "#5A6478", _c >= 0, "#C8553D", "#2A9D8F" )
```

</details>

<details><summary><code>EBIT vs PM</code> — Latest month vs prior month % change in EBIT</summary>

```dax
EBIT vs PM =
VAR _d = [Latest Month Date]
VAR _cur = CALCULATE ( [EBIT], REMOVEFILTERS ( dim_calendar ), dim_calendar[Date] = _d )
VAR _prv = CALCULATE ( [EBIT], REMOVEFILTERS ( dim_calendar ), dim_calendar[Date] = EOMONTH ( _d, -1 ) )
RETURN IF ( NOT ISBLANK ( _prv ), DIVIDE ( _cur - _prv, ABS ( _prv ) ) )
```
Format: `+0.0%;-0.0%;0.0%`

</details>

<details><summary><code>EBIT vs PM Label</code> — Card subtitle: arrow + change in EBIT, latest vs prior month</summary>

```dax
EBIT vs PM Label =
VAR _c = [EBIT vs PM]
RETURN
    IF (
        ISBLANK ( _c ),
        "No prior month",
        [Latest Month Name] & IF ( _c >= 0, " ▲ ", " ▼ " ) & FORMAT ( ABS ( _c ), "0.0%" ) & " vs " & [Prior Month Name]
    )
```

</details>

<details><summary><code>EBIT vs PM Color</code> — Font colour for EBIT delta (increase is good)</summary>

```dax
EBIT vs PM Color =
VAR _c = [EBIT vs PM]
RETURN SWITCH ( TRUE (), ISBLANK ( _c ), "#5A6478", _c >= 0, "#2A9D8F", "#C8553D" )
```

</details>

<details><summary><code>Net Profit vs PM</code> — Latest month vs prior month % change in Net Profit</summary>

```dax
Net Profit vs PM =
VAR _d = [Latest Month Date]
VAR _cur = CALCULATE ( [Net Profit], REMOVEFILTERS ( dim_calendar ), dim_calendar[Date] = _d )
VAR _prv = CALCULATE ( [Net Profit], REMOVEFILTERS ( dim_calendar ), dim_calendar[Date] = EOMONTH ( _d, -1 ) )
RETURN IF ( NOT ISBLANK ( _prv ), DIVIDE ( _cur - _prv, ABS ( _prv ) ) )
```
Format: `+0.0%;-0.0%;0.0%`

</details>

<details><summary><code>Net Profit vs PM Label</code> — Card subtitle: arrow + change in Net Profit, latest vs prior month</summary>

```dax
Net Profit vs PM Label =
VAR _c = [Net Profit vs PM]
RETURN
    IF (
        ISBLANK ( _c ),
        "No prior month",
        [Latest Month Name] & IF ( _c >= 0, " ▲ ", " ▼ " ) & FORMAT ( ABS ( _c ), "0.0%" ) & " vs " & [Prior Month Name]
    )
```

</details>

<details><summary><code>Net Profit vs PM Color</code> — Font colour for Net Profit delta (increase is good)</summary>

```dax
Net Profit vs PM Color =
VAR _c = [Net Profit vs PM]
RETURN SWITCH ( TRUE (), ISBLANK ( _c ), "#5A6478", _c >= 0, "#2A9D8F", "#C8553D" )
```

</details>

<details><summary><code>Net Margin vs PM</code> — Latest month vs prior month change in Net Margin (percentage points)</summary>

```dax
Net Margin vs PM =
VAR _d = [Latest Month Date]
VAR _cur = CALCULATE ( [Net Profit Margin %], REMOVEFILTERS ( dim_calendar ), dim_calendar[Date] = _d )
VAR _prv = CALCULATE ( [Net Profit Margin %], REMOVEFILTERS ( dim_calendar ), dim_calendar[Date] = EOMONTH ( _d, -1 ) )
RETURN IF ( NOT ISBLANK ( _prv ), _cur - _prv )
```
Format: `+0.0%;-0.0%;0.0%`

</details>

<details><summary><code>Net Margin vs PM Label</code> — Card subtitle: arrow + change in Net Margin, latest vs prior month</summary>

```dax
Net Margin vs PM Label =
VAR _c = [Net Margin vs PM]
RETURN
    IF (
        ISBLANK ( _c ),
        "No prior month",
        [Latest Month Name] & IF ( _c >= 0, " ▲ ", " ▼ " ) & FORMAT ( ABS ( _c ) * 100, "0.0" ) & " pp" & " vs " & [Prior Month Name]
    )
```

</details>

<details><summary><code>Net Margin vs PM Color</code> — Font colour for Net Margin delta (increase is good)</summary>

```dax
Net Margin vs PM Color =
VAR _c = [Net Margin vs PM]
RETURN SWITCH ( TRUE (), ISBLANK ( _c ), "#5A6478", _c >= 0, "#2A9D8F", "#C8553D" )
```

</details>

<details><summary><code>Gross Margin vs PM</code> — Latest month vs prior month change in Gross Margin (percentage points)</summary>

```dax
Gross Margin vs PM =
VAR _d = [Latest Month Date]
VAR _cur = CALCULATE ( [Gross Profit Margin %], REMOVEFILTERS ( dim_calendar ), dim_calendar[Date] = _d )
VAR _prv = CALCULATE ( [Gross Profit Margin %], REMOVEFILTERS ( dim_calendar ), dim_calendar[Date] = EOMONTH ( _d, -1 ) )
RETURN IF ( NOT ISBLANK ( _prv ), _cur - _prv )
```
Format: `+0.0%;-0.0%;0.0%`

</details>

<details><summary><code>Gross Margin vs PM Label</code> — Card subtitle: arrow + change in Gross Margin, latest vs prior month</summary>

```dax
Gross Margin vs PM Label =
VAR _c = [Gross Margin vs PM]
RETURN
    IF (
        ISBLANK ( _c ),
        "No prior month",
        [Latest Month Name] & IF ( _c >= 0, " ▲ ", " ▼ " ) & FORMAT ( ABS ( _c ) * 100, "0.0" ) & " pp" & " vs " & [Prior Month Name]
    )
```

</details>

<details><summary><code>Gross Margin vs PM Color</code> — Font colour for Gross Margin delta (increase is good)</summary>

```dax
Gross Margin vs PM Color =
VAR _c = [Gross Margin vs PM]
RETURN SWITCH ( TRUE (), ISBLANK ( _c ), "#5A6478", _c >= 0, "#2A9D8F", "#C8553D" )
```

</details>

<details><summary><code>Opex Ratio vs PM</code> — Latest month vs prior month change in Opex Ratio (percentage points)</summary>

```dax
Opex Ratio vs PM =
VAR _d = [Latest Month Date]
VAR _cur = CALCULATE ( [Opex Ratio %], REMOVEFILTERS ( dim_calendar ), dim_calendar[Date] = _d )
VAR _prv = CALCULATE ( [Opex Ratio %], REMOVEFILTERS ( dim_calendar ), dim_calendar[Date] = EOMONTH ( _d, -1 ) )
RETURN IF ( NOT ISBLANK ( _prv ), _cur - _prv )
```
Format: `+0.0%;-0.0%;0.0%`

</details>

<details><summary><code>Opex Ratio vs PM Label</code> — Card subtitle: arrow + change in Opex Ratio, latest vs prior month</summary>

```dax
Opex Ratio vs PM Label =
VAR _c = [Opex Ratio vs PM]
RETURN
    IF (
        ISBLANK ( _c ),
        "No prior month",
        [Latest Month Name] & IF ( _c >= 0, " ▲ ", " ▼ " ) & FORMAT ( ABS ( _c ) * 100, "0.0" ) & " pp" & " vs " & [Prior Month Name]
    )
```

</details>

<details><summary><code>Opex Ratio vs PM Color</code> — Font colour for Opex Ratio delta (increase is bad)</summary>

```dax
Opex Ratio vs PM Color =
VAR _c = [Opex Ratio vs PM]
RETURN SWITCH ( TRUE (), ISBLANK ( _c ), "#5A6478", _c >= 0, "#C8553D", "#2A9D8F" )
```

</details>

## 06 Break-even

Break-even revenue, margin of safety and the revenue-shock scenario.

<details><summary><code>Fixed Cost</code> — Costs treated as fixed: Opex + Interest & Tax</summary>

```dax
Fixed Cost =
// Simplifying assumption: COGS is variable, everything below gross profit is fixed
[Total Opex] + [Total Interest & Tax]
```
Format: `\$#,0`

</details>

<details><summary><code>Contribution Margin %</code> — 1 - COGS ratio: share of each revenue dollar left to cover fixed costs</summary>

```dax
Contribution Margin % =
1 - [COGS Ratio %]
```
Format: `0.00%;-0.00%;0.00%`

</details>

<details><summary><code>Break-even Revenue</code> — Revenue needed to cover fixed costs in the current context</summary>

```dax
Break-even Revenue =
DIVIDE ( [Fixed Cost], [Contribution Margin %] )
```
Format: `\$#,0`

</details>

<details><summary><code>Margin of Safety %</code> — How far revenue can fall before reaching break-even</summary>

```dax
Margin of Safety % =
DIVIDE ( [Total Revenue] - [Break-even Revenue], [Total Revenue] )
```
Format: `0.00%;-0.00%;0.00%`

</details>

<details><summary><code>Months in Selection</code> — Number of month-end postings in context</summary>

```dax
Months in Selection =
DISTINCTCOUNT ( fact_financial[Date] )
```
Format: `0`

</details>

<details><summary><code>Avg Monthly Revenue</code> — Total revenue / months in selection</summary>

```dax
Avg Monthly Revenue =
DIVIDE ( [Total Revenue], [Months in Selection] )
```
Format: `\$#,0`

</details>

<details><summary><code>Monthly Break-even Revenue</code> — Break-even revenue per month (average fixed cost / contribution margin)</summary>

```dax
Monthly Break-even Revenue =
DIVIDE ( [Break-even Revenue], [Months in Selection] )
```
Format: `\$#,0`

</details>

<details><summary><code>Revenue Change Selected</code> — What-if revenue change picked on the slicer (default 0%)</summary>

```dax
Revenue Change Selected =
SELECTEDVALUE ( prm_revenue_change[Value], 0 )
```
Format: `+0%;-0%;0%`

</details>

<details><summary><code>Scenario Revenue</code> — Revenue after the what-if change</summary>

```dax
Scenario Revenue =
[Total Revenue] * ( 1 + [Revenue Change Selected] )
```
Format: `\$#,0`

</details>

<details><summary><code>Scenario Net Profit</code> — Net profit if revenue moves by the what-if % (COGS scales, opex and I&T fixed)</summary>

```dax
Scenario Net Profit =
VAR _x = [Revenue Change Selected]
RETURN
    [Total Revenue] * ( 1 + _x ) - [Total COGS] * ( 1 + _x ) - [Fixed Cost]
```
Format: `\$#,0;-\$#,0`

</details>

<details><summary><code>Scenario Net Margin %</code> — Scenario net profit / scenario revenue</summary>

```dax
Scenario Net Margin % =
DIVIDE ( [Scenario Net Profit], [Scenario Revenue] )
```
Format: `0.00%;-0.00%;0.00%`

</details>

## 07 Mix

Shares of revenue, profit and cost.

<details><summary><code>Revenue Share %</code> — Share of total revenue across business lines</summary>

```dax
Revenue Share % =
DIVIDE ( [Total Revenue], CALCULATE ( [Total Revenue], REMOVEFILTERS ( dim_business_line ) ) )
```
Format: `0.0%`

</details>

<details><summary><code>Profit Share %</code> — Share of total net profit across business lines (negative = destroys profit)</summary>

```dax
Profit Share % =
DIVIDE ( [Net Profit], CALCULATE ( [Net Profit], REMOVEFILTERS ( dim_business_line ) ) )
```
Format: `0.0%;-0.0%`

</details>

<details><summary><code>Cost % of Revenue</code> — Subgroup cost as % of revenue in the same business line / period</summary>

```dax
Cost % of Revenue =
DIVIDE ( [Amount (Abs)], CALCULATE ( [Total Revenue], REMOVEFILTERS ( dim_subgroup ), REMOVEFILTERS ( dim_group ) ) )
```
Format: `0.0%`

</details>

<details><summary><code>Share of Group %</code> — Subgroup share of its P&L group (e.g. Labor share of COGS)</summary>

```dax
Share of Group % =
DIVIDE ( [Amount (Abs)], CALCULATE ( [Amount (Abs)], REMOVEFILTERS ( dim_subgroup ) ) )
```
Format: `0.0%`

</details>

<details><summary><code>Labor Cost</code> — COGS labor cost (largest single cost line)</summary>

```dax
Labor Cost =
CALCULATE ( [Total COGS], dim_subgroup[Subgroup] = "Labor" )
```
Format: `\$#,0`

</details>

<details><summary><code>Labor % of Revenue</code> — Labor cost / revenue</summary>

```dax
Labor % of Revenue =
DIVIDE ( [Labor Cost], [Total Revenue] )
```
Format: `0.0%`

</details>

<details><summary><code>Expense % of Revenue</code> — Cost line (non-revenue) as % of revenue; revenue subgroups return blank</summary>

```dax
Expense % of Revenue =
DIVIDE (
    - CALCULATE ( SUM ( fact_financial[Amount] ), fact_financial[Group ID] <> 1 ),
    CALCULATE ( [Total Revenue], REMOVEFILTERS ( dim_subgroup ), REMOVEFILTERS ( dim_group ) )
)
```
Format: `0.00%;-0.00%;0.00%`

</details>

## 08 Display

Dynamic text and conditional-format colours (hex strings).

<details><summary><code>Net Profit Color</code> — Bar colour: coral for loss, teal for profit</summary>

```dax
Net Profit Color =
IF ( [Net Profit] < 0, "#C8553D", "#2A9D8F" )
```

</details>

<details><summary><code>Margin Color</code> — Colour for net margin: coral < 0, amber < 15%, teal otherwise</summary>

```dax
Margin Color =
VAR _m = [Net Profit Margin %]
RETURN SWITCH ( TRUE (), _m < 0, "#C8553D", _m < 0.15, "#E09F3E", "#2A9D8F" )
```

</details>

<details><summary><code>Report Period</code> — Header text for the selected period</summary>

```dax
Report Period =
VAR _min = CALCULATE ( MIN ( fact_financial[Date] ) )
VAR _max = CALCULATE ( MAX ( fact_financial[Date] ) )
RETURN IF ( FORMAT ( _min, "mmm yyyy" ) = FORMAT ( _max, "mmm yyyy" ), FORMAT ( _max, "mmmm yyyy" ), FORMAT ( _min, "mmm" ) & " – " & FORMAT ( _max, "mmm yyyy" ) )
```

</details>

<details><summary><code>Insight Headline</code> — Dynamic one-line insight: best and worst net-margin month in the selection</summary>

```dax
Insight Headline =
VAR _months =
    ADDCOLUMNS (
        SUMMARIZE ( fact_financial, dim_calendar[Month], dim_calendar[Month in text] ),
        "@npm", [Net Profit Margin %]
    )
VAR _best = TOPN ( 1, _months, [@npm], DESC )
VAR _worst = TOPN ( 1, _months, [@npm], ASC )
VAR _bestName = MAXX ( _best, dim_calendar[Month in text] )
VAR _worstName = MAXX ( _worst, dim_calendar[Month in text] )
RETURN
    IF (
        COUNTROWS ( _months ) > 1,
        "Net margin " & FORMAT ( [Net Profit Margin %], "0.0%" ) & " · best " & _bestName & " " & FORMAT ( MAXX ( _best, [@npm] ), "0.0%" ) & ", weakest " & _worstName & " " & FORMAT ( MAXX ( _worst, [@npm] ), "0.0%" ),
        "Net margin " & FORMAT ( [Net Profit Margin %], "0.0%" ) & " in " & [Report Period]
    )
```

</details>

<details><summary><code>Loss-making Months</code> — Number of months with negative net profit in context</summary>

```dax
Loss-making Months =
COUNTROWS ( FILTER ( VALUES ( dim_calendar[Month] ), [Net Profit] < 0 ) )
```
Format: `0`

</details>

<details><summary><code>Page Title Overview</code> — Header subtitle for Executive Overview</summary>

```dax
Page Title Overview =
"FY2023 performance · " & [Report Period]
```

</details>

<details><summary><code>COGS Ratio Label</code> — Tile note: COGS as % of revenue</summary>

```dax
COGS Ratio Label =
FORMAT ( [COGS Ratio %], "0.0%" ) & " of revenue"
```

</details>

<details><summary><code>Labor Share Label</code> — Tile note: labor as % of revenue</summary>

```dax
Labor Share Label =
FORMAT ( [Labor % of Revenue], "0.0%" ) & " of revenue"
```

</details>

<details><summary><code>Fixed Cost Label</code> — Tile note: average fixed cost per month</summary>

```dax
Fixed Cost Label =
"Fixed cost " & FORMAT ( DIVIDE ( [Fixed Cost], [Months in Selection] ), "$#,0" ) & " / month"
```

</details>

<details><summary><code>Avg Revenue Label</code> — Tile note: months in selection</summary>

```dax
Avg Revenue Label =
FORMAT ( [Months in Selection], "0" ) & " months in view"
```

</details>

<details><summary><code>Safety Label</code> — Tile note: what margin of safety means</summary>

```dax
Safety Label =
"Revenue can fall " & FORMAT ( [Margin of Safety %], "0%" ) & " before a loss"
```

</details>

<details><summary><code>Contribution Label</code> — Tile note: contribution per dollar</summary>

```dax
Contribution Label =
"Each $1 of sales leaves " & FORMAT ( [Contribution Margin %], "$0.00" )
```

</details>

<details><summary><code>Neutral Color</code> — Muted grey for neutral notes</summary>

```dax
Neutral Color =
"#5A6478"
```

</details>

<details><summary><code>Revenue MoM Color</code> — Column colour for Revenue MoM % mini chart (increase is good)</summary>

```dax
Revenue MoM Color =
VAR _c = [Revenue MoM %]
RETURN IF ( ISBLANK ( _c ), "#B8C2D3", IF ( _c >= 0, "#2A9D8F", "#C8553D" ) )
```

</details>

<details><summary><code>Gross Profit MoM Color</code> — Column colour for Gross Profit MoM % mini chart (increase is good)</summary>

```dax
Gross Profit MoM Color =
VAR _c = [Gross Profit MoM %]
RETURN IF ( ISBLANK ( _c ), "#B8C2D3", IF ( _c >= 0, "#2A9D8F", "#C8553D" ) )
```

</details>

<details><summary><code>Opex MoM Color</code> — Column colour for Opex MoM % mini chart (increase is bad)</summary>

```dax
Opex MoM Color =
VAR _c = [Opex MoM %]
RETURN IF ( ISBLANK ( _c ), "#B8C2D3", IF ( _c >= 0, "#C8553D", "#2A9D8F" ) )
```

</details>

<details><summary><code>EBIT MoM Color</code> — Column colour for EBIT MoM % mini chart (increase is good)</summary>

```dax
EBIT MoM Color =
VAR _c = [EBIT MoM %]
RETURN IF ( ISBLANK ( _c ), "#B8C2D3", IF ( _c >= 0, "#2A9D8F", "#C8553D" ) )
```

</details>

<details><summary><code>Net Profit MoM Color</code> — Column colour for Net Profit MoM % mini chart (increase is good)</summary>

```dax
Net Profit MoM Color =
VAR _c = [Net Profit MoM %]
RETURN IF ( ISBLANK ( _c ), "#B8C2D3", IF ( _c >= 0, "#2A9D8F", "#C8553D" ) )
```

</details>

<details><summary><code>Net Margin MoM Color</code> — Column colour for Net Margin MoM pp mini chart (increase is good)</summary>

```dax
Net Margin MoM Color =
VAR _c = [Net Margin MoM pp]
RETURN IF ( ISBLANK ( _c ), "#B8C2D3", IF ( _c >= 0, "#2A9D8F", "#C8553D" ) )
```

</details>

<details><summary><code>COGS MoM Color</code> — Column colour for COGS MoM % mini chart (increase is bad)</summary>

```dax
COGS MoM Color =
VAR _c = [COGS MoM %]
RETURN IF ( ISBLANK ( _c ), "#B8C2D3", IF ( _c >= 0, "#C8553D", "#2A9D8F" ) )
```

</details>

<details><summary><code>Opex Ratio MoM Color</code> — Column colour for Opex Ratio MoM pp mini chart (increase is bad)</summary>

```dax
Opex Ratio MoM Color =
VAR _c = [Opex Ratio MoM pp]
RETURN IF ( ISBLANK ( _c ), "#B8C2D3", IF ( _c >= 0, "#C8553D", "#2A9D8F" ) )
```

</details>

<details><summary><code>Labor MoM Color</code> — Column colour for Labor MoM % mini chart (increase is bad)</summary>

```dax
Labor MoM Color =
VAR _c = [Labor MoM %]
RETURN IF ( ISBLANK ( _c ), "#B8C2D3", IF ( _c >= 0, "#C8553D", "#2A9D8F" ) )
```

</details>

<details><summary><code>Revenue FY Trend Color</code> — Grey when the FY2023 Revenue Jan-to-Dec trend moves less than 0.5%/mo; otherwise green when favourable (rising), red when not</summary>

```dax
Revenue FY Trend Color =
VAR _f = CALCULATE ( [Total Revenue], REMOVEFILTERS ( dim_calendar ), dim_calendar[Month] = 1 )
VAR _l = CALCULATE ( [Total Revenue], REMOVEFILTERS ( dim_calendar ), dim_calendar[Month] = 12 )
VAR _rate = IF ( _f > 0 && _l > 0, POWER ( DIVIDE ( _l, _f ), 1 / 11 ) - 1, DIVIDE ( _l - _f, ABS ( _f ) ) / 11 )
RETURN
    SWITCH ( TRUE (), ISBLANK ( _f ) || ISBLANK ( _l ), "#5A6478",
        ABS ( _rate ) < 0.005, "#5A6478",
        _rate >= 0, "#2A9D8F", "#C8553D" )
```

</details>

<details><summary><code>Gross Profit FY Trend Color</code> — Grey when the FY2023 Gross Profit Jan-to-Dec trend moves less than 0.5%/mo; otherwise green when favourable (rising), red when not</summary>

```dax
Gross Profit FY Trend Color =
VAR _f = CALCULATE ( [Gross Profit], REMOVEFILTERS ( dim_calendar ), dim_calendar[Month] = 1 )
VAR _l = CALCULATE ( [Gross Profit], REMOVEFILTERS ( dim_calendar ), dim_calendar[Month] = 12 )
VAR _rate = IF ( _f > 0 && _l > 0, POWER ( DIVIDE ( _l, _f ), 1 / 11 ) - 1, DIVIDE ( _l - _f, ABS ( _f ) ) / 11 )
RETURN
    SWITCH ( TRUE (), ISBLANK ( _f ) || ISBLANK ( _l ), "#5A6478",
        ABS ( _rate ) < 0.005, "#5A6478",
        _rate >= 0, "#2A9D8F", "#C8553D" )
```

</details>

<details><summary><code>Opex FY Trend Color</code> — Grey when the FY2023 Opex Jan-to-Dec trend moves less than 0.5%/mo; otherwise green when favourable (falling), red when not</summary>

```dax
Opex FY Trend Color =
VAR _f = CALCULATE ( [Total Opex], REMOVEFILTERS ( dim_calendar ), dim_calendar[Month] = 1 )
VAR _l = CALCULATE ( [Total Opex], REMOVEFILTERS ( dim_calendar ), dim_calendar[Month] = 12 )
VAR _rate = IF ( _f > 0 && _l > 0, POWER ( DIVIDE ( _l, _f ), 1 / 11 ) - 1, DIVIDE ( _l - _f, ABS ( _f ) ) / 11 )
RETURN
    SWITCH ( TRUE (), ISBLANK ( _f ) || ISBLANK ( _l ), "#5A6478",
        ABS ( _rate ) < 0.005, "#5A6478",
        _rate <= 0, "#2A9D8F", "#C8553D" )
```

</details>

<details><summary><code>EBIT FY Trend Color</code> — Grey when the FY2023 EBIT Jan-to-Dec trend moves less than 0.5%/mo; otherwise green when favourable (rising), red when not</summary>

```dax
EBIT FY Trend Color =
VAR _f = CALCULATE ( [EBIT], REMOVEFILTERS ( dim_calendar ), dim_calendar[Month] = 1 )
VAR _l = CALCULATE ( [EBIT], REMOVEFILTERS ( dim_calendar ), dim_calendar[Month] = 12 )
VAR _rate = IF ( _f > 0 && _l > 0, POWER ( DIVIDE ( _l, _f ), 1 / 11 ) - 1, DIVIDE ( _l - _f, ABS ( _f ) ) / 11 )
RETURN
    SWITCH ( TRUE (), ISBLANK ( _f ) || ISBLANK ( _l ), "#5A6478",
        ABS ( _rate ) < 0.005, "#5A6478",
        _rate >= 0, "#2A9D8F", "#C8553D" )
```

</details>

<details><summary><code>Net Profit FY Trend Color</code> — Grey when the FY2023 Net Profit Jan-to-Dec trend moves less than 0.5%/mo; otherwise green when favourable (rising), red when not</summary>

```dax
Net Profit FY Trend Color =
VAR _f = CALCULATE ( [Net Profit], REMOVEFILTERS ( dim_calendar ), dim_calendar[Month] = 1 )
VAR _l = CALCULATE ( [Net Profit], REMOVEFILTERS ( dim_calendar ), dim_calendar[Month] = 12 )
VAR _rate = IF ( _f > 0 && _l > 0, POWER ( DIVIDE ( _l, _f ), 1 / 11 ) - 1, DIVIDE ( _l - _f, ABS ( _f ) ) / 11 )
RETURN
    SWITCH ( TRUE (), ISBLANK ( _f ) || ISBLANK ( _l ), "#5A6478",
        ABS ( _rate ) < 0.005, "#5A6478",
        _rate >= 0, "#2A9D8F", "#C8553D" )
```

</details>

<details><summary><code>Net Margin FY Trend Color</code> — Grey when the FY2023 Net Margin Jan-to-Dec trend moves less than 0.25 pp/mo; otherwise green when favourable (rising), red when not</summary>

```dax
Net Margin FY Trend Color =
VAR _f = CALCULATE ( [Net Profit Margin %], REMOVEFILTERS ( dim_calendar ), dim_calendar[Month] = 1 )
VAR _l = CALCULATE ( [Net Profit Margin %], REMOVEFILTERS ( dim_calendar ), dim_calendar[Month] = 12 )
VAR _rate = ( _l - _f ) / 11
RETURN
    SWITCH ( TRUE (), ISBLANK ( _f ) || ISBLANK ( _l ), "#5A6478",
        ABS ( _rate ) < 0.0025, "#5A6478",
        _rate >= 0, "#2A9D8F", "#C8553D" )
```

</details>

<details><summary><code>COGS FY Trend Color</code> — Grey when the FY2023 COGS Jan-to-Dec trend moves less than 0.5%/mo; otherwise green when favourable (falling), red when not</summary>

```dax
COGS FY Trend Color =
VAR _f = CALCULATE ( [Total COGS], REMOVEFILTERS ( dim_calendar ), dim_calendar[Month] = 1 )
VAR _l = CALCULATE ( [Total COGS], REMOVEFILTERS ( dim_calendar ), dim_calendar[Month] = 12 )
VAR _rate = IF ( _f > 0 && _l > 0, POWER ( DIVIDE ( _l, _f ), 1 / 11 ) - 1, DIVIDE ( _l - _f, ABS ( _f ) ) / 11 )
RETURN
    SWITCH ( TRUE (), ISBLANK ( _f ) || ISBLANK ( _l ), "#5A6478",
        ABS ( _rate ) < 0.005, "#5A6478",
        _rate <= 0, "#2A9D8F", "#C8553D" )
```

</details>

<details><summary><code>Opex Ratio FY Trend Color</code> — Grey when the FY2023 Opex Ratio Jan-to-Dec trend moves less than 0.25 pp/mo; otherwise green when favourable (falling), red when not</summary>

```dax
Opex Ratio FY Trend Color =
VAR _f = CALCULATE ( [Opex Ratio %], REMOVEFILTERS ( dim_calendar ), dim_calendar[Month] = 1 )
VAR _l = CALCULATE ( [Opex Ratio %], REMOVEFILTERS ( dim_calendar ), dim_calendar[Month] = 12 )
VAR _rate = ( _l - _f ) / 11
RETURN
    SWITCH ( TRUE (), ISBLANK ( _f ) || ISBLANK ( _l ), "#5A6478",
        ABS ( _rate ) < 0.0025, "#5A6478",
        _rate <= 0, "#2A9D8F", "#C8553D" )
```

</details>

<details><summary><code>Labor FY Trend Color</code> — Grey when the FY2023 Labor Jan-to-Dec trend moves less than 0.5%/mo; otherwise green when favourable (falling), red when not</summary>

```dax
Labor FY Trend Color =
VAR _f = CALCULATE ( [Labor Cost], REMOVEFILTERS ( dim_calendar ), dim_calendar[Month] = 1 )
VAR _l = CALCULATE ( [Labor Cost], REMOVEFILTERS ( dim_calendar ), dim_calendar[Month] = 12 )
VAR _rate = IF ( _f > 0 && _l > 0, POWER ( DIVIDE ( _l, _f ), 1 / 11 ) - 1, DIVIDE ( _l - _f, ABS ( _f ) ) / 11 )
RETURN
    SWITCH ( TRUE (), ISBLANK ( _f ) || ISBLANK ( _l ), "#5A6478",
        ABS ( _rate ) < 0.005, "#5A6478",
        _rate <= 0, "#2A9D8F", "#C8553D" )
```

</details>

<details><summary><code>Takeaway P&L</code> — P&L page header insight: where each revenue dollar goes</summary>

```dax
Takeaway P&L =
"COGS takes " & FORMAT ( [COGS Ratio %], "0.0%" ) & " of revenue and opex " & FORMAT ( [Opex Ratio %], "0.0%" ) & ", leaving a " & FORMAT ( [Net Profit Margin %], "0.0%" ) & " net margin"
```

</details>

<details><summary><code>Takeaway Lines</code> — Business Lines page header insight: profit concentration and the loss-making line</summary>

```dax
Takeaway Lines =
VAR _t = ADDCOLUMNS ( VALUES ( dim_business_line[Business Line] ), "@np", [Net Profit], "@rev", [Total Revenue] )
VAR _np = SUMX ( _t, [@np] )
VAR _rev = SUMX ( _t, [@rev] )
VAR _top = TOPN ( 1, _t, [@np], DESC )
VAR _low = TOPN ( 1, _t, [@np], ASC )
VAR _lowNp = MAXX ( _low, [@np] )
RETURN
    IF ( COUNTROWS ( _t ) < 2,
        MAXX ( _t, dim_business_line[Business Line] ) & ": net margin " & FORMAT ( [Net Profit Margin %], "0.0%" ),
        MAXX ( _top, dim_business_line[Business Line] ) & " earns " & FORMAT ( DIVIDE ( MAXX ( _top, [@np] ), _np ), "0%" ) & " of net profit on "
            & FORMAT ( DIVIDE ( MAXX ( _top, [@rev] ), _rev ), "0%" ) & " of revenue"
            & IF ( _lowNp < 0, "; " & SUBSTITUTE ( MAXX ( _low, dim_business_line[Business Line] ), " and Food Supplements", "" ) & " loses $" & FORMAT ( -_lowNp / 1000000, "0.00" ) & "M", "" ) )
```

</details>

<details><summary><code>Takeaway Cost</code> — Cost page header insight: the single largest cost line</summary>

```dax
Takeaway Cost =
VAR _t = ADDCOLUMNS ( VALUES ( dim_subgroup[Subgroup] ), "@c", [Total COGS] + [Total Opex] )
VAR _top = TOPN ( 1, _t, [@c], DESC )
VAR _c = MAXX ( _top, [@c] )
RETURN
    MAXX ( _top, dim_subgroup[Subgroup] ) & " is the largest cost line at $" & FORMAT ( _c / 1000000, "0.00" ) & "M, "
        & FORMAT ( DIVIDE ( _c, [Total Revenue] ), "0.0%" ) & " of revenue"
```

</details>

<details><summary><code>Takeaway Break-even</code> — Break-even page header insight: the month closest to break-even</summary>

```dax
Takeaway Break-even =
VAR _t = ADDCOLUMNS ( SUMMARIZE ( fact_financial, dim_calendar[Month], dim_calendar[Month in text] ), "@ms", [Margin of Safety %] )
VAR _low = TOPN ( 1, _t, [@ms], ASC )
VAR _ms = MAXX ( _low, [@ms] )
VAR _txt = IF ( _ms >= 0, "revenue only " & FORMAT ( _ms, "0.0%" ) & " above break-even", "revenue " & FORMAT ( -_ms, "0.0%" ) & " below break-even" )
RETURN
    IF ( COUNTROWS ( _t ) < 2,
        UPPER ( LEFT ( _txt, 1 ) ) & MID ( _txt, 2, 200 ),
        "Tightest month: " & MAXX ( _low, dim_calendar[Month in text] ) & ", " & _txt )
```

</details>

<details><summary><code>Takeaway Benchmark</code> — Benchmark page header insight: how many lines beat their industry net margin</summary>

```dax
Takeaway Benchmark =
VAR _t = ADDCOLUMNS ( VALUES ( dim_business_line[Business Line] ), "@gap", [Net Margin Gap vs Benchmark] )
VAR _n = COALESCE ( COUNTROWS ( FILTER ( _t, [@gap] > 0 ) ), 0 )
VAR _low = TOPN ( 1, _t, [@gap], ASC )
VAR _gap = MAXX ( _low, [@gap] )
RETURN
    _n & " of " & COUNTROWS ( _t ) & " lines beat their industry net margin"
        & IF ( _gap < 0, "; " & SUBSTITUTE ( MAXX ( _low, dim_business_line[Business Line] ), " and Food Supplements", "" ) & " trails by " & FORMAT ( -_gap * 100, "0.0" ) & " pp", "" )
```

</details>

## 09 Benchmark

Revenue-weighted industry benchmarks and gaps.

<details><summary><code>Benchmark Gross Margin</code> — Revenue-weighted industry benchmark for Gross Margin (Damodaran, Jan 2026): each business line mapped to its industry</summary>

```dax
Benchmark Gross Margin =
// Weight each line's industry benchmark by that line's revenue in the current context
VAR _t =
    ADDCOLUMNS (
        VALUES ( dim_business_line[Business Line ID] ),
        "@rev", [Total Revenue],
        "@bm", LOOKUPVALUE ( dim_benchmark[Bm Gross Margin], dim_benchmark[Business Line ID], dim_business_line[Business Line ID] )
    )
RETURN
    DIVIDE ( SUMX ( _t, [@rev] * [@bm] ), SUMX ( _t, [@rev] ) )
```
Format: `0.0%;-0.0%`

</details>

<details><summary><code>Gross Margin Gap vs Benchmark</code> — Gross Margin actual minus benchmark (percentage points)</summary>

```dax
Gross Margin Gap vs Benchmark =
[Gross Profit Margin %] - [Benchmark Gross Margin]
```
Format: `+0.0%;-0.0%;0.0%`

</details>

<details><summary><code>Gross Margin Benchmark Color</code> — Teal when Gross Margin beats its benchmark, coral otherwise</summary>

```dax
Gross Margin Benchmark Color =
VAR _g = [Gross Margin Gap vs Benchmark]
RETURN IF ( ISBLANK ( _g ), "#5A6478", IF ( _g >= 0, "#2A9D8F", "#C8553D" ) )
```

</details>

<details><summary><code>Benchmark Opex Ratio</code> — Revenue-weighted industry benchmark for Opex Ratio (Damodaran, Jan 2026): each business line mapped to its industry</summary>

```dax
Benchmark Opex Ratio =
// Weight each line's industry benchmark by that line's revenue in the current context
VAR _t =
    ADDCOLUMNS (
        VALUES ( dim_business_line[Business Line ID] ),
        "@rev", [Total Revenue],
        "@bm", LOOKUPVALUE ( dim_benchmark[Bm Opex Ratio], dim_benchmark[Business Line ID], dim_business_line[Business Line ID] )
    )
RETURN
    DIVIDE ( SUMX ( _t, [@rev] * [@bm] ), SUMX ( _t, [@rev] ) )
```
Format: `0.0%;-0.0%`

</details>

<details><summary><code>Opex Ratio Gap vs Benchmark</code> — Opex Ratio actual minus benchmark (percentage points); negative is better</summary>

```dax
Opex Ratio Gap vs Benchmark =
[Opex Ratio %] - [Benchmark Opex Ratio]
```
Format: `+0.0%;-0.0%;0.0%`

</details>

<details><summary><code>Opex Ratio Benchmark Color</code> — Teal when Opex Ratio beats its benchmark, coral otherwise</summary>

```dax
Opex Ratio Benchmark Color =
VAR _g = [Opex Ratio Gap vs Benchmark]
RETURN IF ( ISBLANK ( _g ), "#5A6478", IF ( _g <= 0, "#2A9D8F", "#C8553D" ) )
```

</details>

<details><summary><code>Benchmark EBIT Margin</code> — Revenue-weighted industry benchmark for EBIT Margin (Damodaran, Jan 2026): each business line mapped to its industry</summary>

```dax
Benchmark EBIT Margin =
// Weight each line's industry benchmark by that line's revenue in the current context
VAR _t =
    ADDCOLUMNS (
        VALUES ( dim_business_line[Business Line ID] ),
        "@rev", [Total Revenue],
        "@bm", LOOKUPVALUE ( dim_benchmark[Bm EBIT Margin], dim_benchmark[Business Line ID], dim_business_line[Business Line ID] )
    )
RETURN
    DIVIDE ( SUMX ( _t, [@rev] * [@bm] ), SUMX ( _t, [@rev] ) )
```
Format: `0.0%;-0.0%`

</details>

<details><summary><code>EBIT Margin Gap vs Benchmark</code> — EBIT Margin actual minus benchmark (percentage points)</summary>

```dax
EBIT Margin Gap vs Benchmark =
[EBIT Margin %] - [Benchmark EBIT Margin]
```
Format: `+0.0%;-0.0%;0.0%`

</details>

<details><summary><code>EBIT Margin Benchmark Color</code> — Teal when EBIT Margin beats its benchmark, coral otherwise</summary>

```dax
EBIT Margin Benchmark Color =
VAR _g = [EBIT Margin Gap vs Benchmark]
RETURN IF ( ISBLANK ( _g ), "#5A6478", IF ( _g >= 0, "#2A9D8F", "#C8553D" ) )
```

</details>

<details><summary><code>Benchmark Net Margin</code> — Revenue-weighted industry benchmark for Net Margin (Damodaran, Jan 2026): each business line mapped to its industry</summary>

```dax
Benchmark Net Margin =
// Weight each line's industry benchmark by that line's revenue in the current context
VAR _t =
    ADDCOLUMNS (
        VALUES ( dim_business_line[Business Line ID] ),
        "@rev", [Total Revenue],
        "@bm", LOOKUPVALUE ( dim_benchmark[Bm Net Margin], dim_benchmark[Business Line ID], dim_business_line[Business Line ID] )
    )
RETURN
    DIVIDE ( SUMX ( _t, [@rev] * [@bm] ), SUMX ( _t, [@rev] ) )
```
Format: `0.00%;-0.00%;0.00%`

</details>

<details><summary><code>Net Margin Gap vs Benchmark</code> — Net Margin actual minus benchmark (percentage points)</summary>

```dax
Net Margin Gap vs Benchmark =
[Net Profit Margin %] - [Benchmark Net Margin]
```
Format: `+0.0%;-0.0%;0.0%`

</details>

<details><summary><code>Net Margin Benchmark Color</code> — Teal when Net Margin beats its benchmark, coral otherwise</summary>

```dax
Net Margin Benchmark Color =
VAR _g = [Net Margin Gap vs Benchmark]
RETURN IF ( ISBLANK ( _g ), "#5A6478", IF ( _g >= 0, "#2A9D8F", "#C8553D" ) )
```

</details>

<details><summary><code>Benchmark Industry Label</code> — Industry used as benchmark for the selected business line</summary>

```dax
Benchmark Industry Label =
IF ( HASONEVALUE ( dim_business_line[Business Line ID] ), LOOKUPVALUE ( dim_benchmark[Benchmark Industry], dim_benchmark[Business Line ID], SELECTEDVALUE ( dim_business_line[Business Line ID] ) ), "Revenue-weighted mix of industries" )
```

</details>

<details><summary><code>Benchmark Source</code> — Footnote text for benchmark visuals</summary>

```dax
Benchmark Source =
"Benchmark: Damodaran, Margins by Sector (US), data as of Jan 2026. Nutrition = Food Processing, Sports Equipment = Recreation, Sportswear = Apparel. Opex = SG&A + R&D."
```

</details>

<details><summary><code>Nutrition Opex Saving Scenario</code> — Opex saved if Nutrition ran at Sports Equipment's opex ratio (illustrative, assumes direct costs)</summary>

```dax
Nutrition Opex Saving Scenario =
VAR _nutRev = CALCULATE ( [Total Revenue], dim_business_line[Business Line ID] = 1 )
VAR _nutOpex = CALCULATE ( [Total Opex], dim_business_line[Business Line ID] = 1 )
VAR _seRatio = CALCULATE ( [Opex Ratio %], dim_business_line[Business Line ID] = 2 )
RETURN MAX ( 0, _nutOpex - _nutRev * _seRatio )
```
Format: `\$#,0`

</details>

<details><summary><code>Scenario Group Net Margin</code> — Group net margin after the Nutrition opex scenario</summary>

```dax
Scenario Group Net Margin =
VAR _allRev = CALCULATE ( [Total Revenue], REMOVEFILTERS ( dim_business_line ) )
VAR _allNP = CALCULATE ( [Net Profit], REMOVEFILTERS ( dim_business_line ) )
RETURN DIVIDE ( _allNP + [Nutrition Opex Saving Scenario], _allRev )
```
Format: `0.0%`

</details>

<details><summary><code>Gross Margin Gap Label</code> — Tile note: Gross Margin vs industry benchmark in pp</summary>

```dax
Gross Margin Gap Label =
VAR _g = [Gross Margin Gap vs Benchmark]
RETURN IF ( ISBLANK ( _g ), "No benchmark", IF ( _g >= 0, "▲ ", "▼ " ) & FORMAT ( ABS ( _g ) * 100, "0.0" ) & " pp vs industry " & FORMAT ( [Benchmark Gross Margin], "0.0%" ) )
```

</details>

<details><summary><code>Opex Ratio Gap Label</code> — Tile note: Opex Ratio vs industry benchmark in pp</summary>

```dax
Opex Ratio Gap Label =
VAR _g = [Opex Ratio Gap vs Benchmark]
RETURN IF ( ISBLANK ( _g ), "No benchmark", IF ( _g >= 0, "▲ ", "▼ " ) & FORMAT ( ABS ( _g ) * 100, "0.0" ) & " pp vs industry " & FORMAT ( [Benchmark Opex Ratio], "0.0%" ) )
```

</details>

<details><summary><code>EBIT Margin Gap Label</code> — Tile note: EBIT Margin vs industry benchmark in pp</summary>

```dax
EBIT Margin Gap Label =
VAR _g = [EBIT Margin Gap vs Benchmark]
RETURN IF ( ISBLANK ( _g ), "No benchmark", IF ( _g >= 0, "▲ ", "▼ " ) & FORMAT ( ABS ( _g ) * 100, "0.0" ) & " pp vs industry " & FORMAT ( [Benchmark EBIT Margin], "0.0%" ) )
```

</details>

<details><summary><code>Net Margin Gap Label</code> — Tile note: Net Margin vs industry benchmark in pp</summary>

```dax
Net Margin Gap Label =
VAR _g = [Net Margin Gap vs Benchmark]
RETURN IF ( ISBLANK ( _g ), "No benchmark", IF ( _g >= 0, "▲ ", "▼ " ) & FORMAT ( ABS ( _g ) * 100, "0.0" ) & " pp vs industry " & FORMAT ( [Benchmark Net Margin], "0.0%" ) )
```

</details>

<details><summary><code>Scorecard Actual</code> — Actual value of the scorecard metric in dim_metric</summary>

```dax
Scorecard Actual =
SWITCH ( SELECTEDVALUE ( dim_metric[Order] ), 1, [Gross Profit Margin %], 2, [Opex Ratio %], 3, [EBIT Margin %], 4, [Net Profit Margin %] )
```
Format: `0.00%;-0.00%;0.00%`

</details>

<details><summary><code>Scorecard Industry</code> — Industry benchmark of the scorecard metric</summary>

```dax
Scorecard Industry =
SWITCH ( SELECTEDVALUE ( dim_metric[Order] ), 1, [Benchmark Gross Margin], 2, [Benchmark Opex Ratio], 3, [Benchmark EBIT Margin], 4, [Benchmark Net Margin] )
```
Format: `0.00%;-0.00%;0.00%`

</details>

<details><summary><code>Scorecard Gap</code> — Actual minus industry, in percentage points</summary>

```dax
Scorecard Gap =
VAR _a = [Scorecard Actual]
VAR _b = [Scorecard Industry]
RETURN IF ( NOT ISBLANK ( _a ) && NOT ISBLANK ( _b ), ( _a - _b ) * 100 )
```
Format: `0.00`

</details>

<details><summary><code>Scorecard Gap Color</code> — Teal when the line beats industry (lower is better for Opex ratio)</summary>

```dax
Scorecard Gap Color =
VAR _g = [Scorecard Gap]
VAR _lowerBetter = SELECTEDVALUE ( dim_metric[Order] ) = 2
RETURN IF ( ISBLANK ( _g ), "#5A6478", IF ( ( _g >= 0 && NOT _lowerBetter ) || ( _g <= 0 && _lowerBetter ), "#2A9D8F", "#C8553D" ) )
```

</details>

## 10 Cards

Card-ready versions of key values with compact formats.

<details><summary><code>Card Revenue</code> — KPI card display of [Total Revenue] in compact units</summary>

```dax
Card Revenue =
[Total Revenue]
```
Format: `\$0,,.00"M";-\$0,,.00"M"`

</details>

<details><summary><code>Card Gross Profit</code> — KPI card display of [Gross Profit] in compact units</summary>

```dax
Card Gross Profit =
[Gross Profit]
```
Format: `\$0,,.00"M";-\$0,,.00"M"`

</details>

<details><summary><code>Card Opex</code> — KPI card display of [Total Opex] in compact units</summary>

```dax
Card Opex =
[Total Opex]
```
Format: `\$0,,.00"M";-\$0,,.00"M"`

</details>

<details><summary><code>Card EBIT</code> — KPI card display of [EBIT] in compact units</summary>

```dax
Card EBIT =
[EBIT]
```
Format: `\$0,,.00"M";-\$0,,.00"M"`

</details>

<details><summary><code>Card Net Profit</code> — KPI card display of [Net Profit] in compact units</summary>

```dax
Card Net Profit =
[Net Profit]
```
Format: `\$0,,.00"M";-\$0,,.00"M"`

</details>

<details><summary><code>Card COGS</code> — KPI card display of [Total COGS] in compact units</summary>

```dax
Card COGS =
[Total COGS]
```
Format: `\$0,,.00"M";-\$0,,.00"M"`

</details>

<details><summary><code>Card Labor</code> — KPI card display of [Labor Cost] in compact units</summary>

```dax
Card Labor =
[Labor Cost]
```
Format: `\$0,,.00"M";-\$0,,.00"M"`

</details>

<details><summary><code>Card Break-even Month</code> — KPI card display of [Monthly Break-even Revenue] in compact units</summary>

```dax
Card Break-even Month =
[Monthly Break-even Revenue]
```
Format: `\$0,,.00"M";-\$0,,.00"M"`

</details>

<details><summary><code>Card Avg Revenue</code> — KPI card display of [Avg Monthly Revenue] in compact units</summary>

```dax
Card Avg Revenue =
[Avg Monthly Revenue]
```
Format: `\$0,,.00"M";-\$0,,.00"M"`

</details>
