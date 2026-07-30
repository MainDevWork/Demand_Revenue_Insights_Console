An Excel reporting console that consolidates the leads and revenue produced by marketing campaigns run across several JSE divisions, and reports them on two dashboards that rebuild themselves from a single set of daily inputs.

It was built for the **Marketing & Corporate Affairs** division, which runs campaigns on behalf of other divisions and had no single, comparable view of how any of them were performing.

## What the dashboards report

Two dashboards, both read-only, both built from the same two tables of daily inputs.

### Tracking dashboard

![Tracking dashboard](docs/images/tracking_dashboard.png)

*Current month, current quarter and year-to-date totals for every business group, with month-to-month and quarter-to-quarter movements coloured green for growth and red for decline. The month name at the top left is today's month, so the sheet is always current without anyone adjusting it.*

The percentages compare a period to date against the full previous period, which is why a quarter that has only just begun shows steep negatives. The sheet carries a footnote saying so, because that is the most easily misread thing on it.

### Analysis dashboard

![Analysis dashboard](docs/images/analysis_dashboard.png)

*KPI tiles, monthly trend lines per business group, year-to-date contribution, cumulative revenue and quarterly revenue. Every business group keeps the same colour on every chart, so the views can be read together.*

The trend lines stop at the current month rather than dropping to zero for months that have not happened yet. That is deliberate. The engine returns `NA()` for future months, which tells Excel to break the line instead of plotting a false floor.

**A note on the data.** Every figure in this repository is generated. It exists to populate the workbook and demonstrate that the reporting works, and it contains no real campaign results. Only the numbers are invented. The workbook itself, meaning its sheets, formulas, validation, dashboards and charts, is the one that was built for the division.

---

## What is in this repository

| File                                                                                  | Description                                                                                                                                                          |
| ------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [MCA_Revenue_Demand_Generation_Report.xlsx](MCA_Revenue_Demand_Generation_Report.xlsx) | The console. An eight-sheet workbook holding both dashboards, both calculation engines, the two input sheets, the reference lists and the in-workbook`ReadMe` page |
| [docs/images/](docs/images/)                                                           | Screenshots of every sheet described below                                                                                                                           |

Everything the tool needs is in the workbook. There is nothing to install and no external data source to connect.

## The problem

Marketing & Corporate Affairs runs campaigns through several revenue-generating divisions at once. On the revenue side those are Primary Markets, Secondary Markets, Information Services and Issuer Services. On the demand side they are JSE SheInvests, Digital Marketing, Events and Marketing Services.

Each team kept its own record of what those campaigns produced, which created three problems:

- **Performance lived in separate files.** Every team recorded its leads and revenue in its own format and on its own schedule, so no single view of campaign performance existed.
- **Reporting was manual and repetitive.** Each month-end summary was rebuilt by hand, which repeated effort and invited error.
- **Definitions were inconsistent.** "This month" and "quarter to date" meant different things in different files, so the numbers did not reconcile.

## How it is built

Three layers, arranged so that logic and presentation never interfere with each other. A user only ever types into the two green input sheets, and everything else builds itself.

| Layer        | Sheets                                     | What it does                                                                    |
| ------------ | ------------------------------------------ | ------------------------------------------------------------------------------- |
| Input        | `Demand_Input`, `Revenue_Input`        | Two structured Excel tables: date, business group, campaign/stream, daily value |
| Engine       | `Calculations`, `Analysis_Calc`        | `SUMIFS` over the named tables, bounded by dates derived from `TODAY()`     |
| Presentation | `MAIN DASHBOARD`, `Analysis Dashboard` | Reference the engines only, and contain no calculations of their own            |
| Reference    | `Lists`, `ReadMe`                      | Approved values that drive the drop-downs, and an orientation page              |

```
Demand_Input  (daily leads)          Revenue_Input  (daily revenue, Rands)
  one row per group per day             one row per group per day
  validated at the point of entry       validated at the point of entry
        |                                     |
        +------------------+------------------+
        |                                     |
        v                                     v
   Calculations                          Analysis_Calc
   TODAY()-anchored SUMIFS:              monthly matrix, quarterly totals,
   month / quarter / YTD per group       YTD contribution, KPI helpers
        |                                     |
        v                                     v
   MAIN DASHBOARD                        Analysis Dashboard
   totals, movement percentages          KPI tiles, trends, contribution,
                                         cumulative and quarterly revenue
```

### The input layer

![Demand_Input sheet](docs/images/demand_input.png)

Both input sheets are structured Excel tables, so every formula downstream refers to `DemandTbl` and `RevenueTbl` by name and picks up new rows automatically. All four columns are validated at the point of entry:

| Column          | Rule                                                                                 |
| --------------- | ------------------------------------------------------------------------------------ |
| Date            | Must be a date, and not earlier than 2020                                            |
| Business Group  | Drop-down, restricted to the approved list                                           |
| Campaign/Stream | Drop-down, restricted to the approved list                                           |
| Daily value     | Leads must be a whole number, revenue may have decimals, and neither may be negative |

Validation is what makes the automation reliable rather than merely automatic. A misspelled business group would not raise an error, it would silently report zero for that team, and nobody would notice until someone asked why a campaign had stopped producing.

The approved values live on one sheet, so the lists that drive both drop-downs and the pairings between groups and campaigns are all in a single place:

![Lists sheet](docs/images/lists.png)

### The engine layer

![Calculations sheet](docs/images/calculations.png)

The block at the top is where the whole design rests. It derives the current month, the current year, the previous month, the current and previous quarter and the year start, all from `TODAY()`. Every figure below it, and everything on the dashboard that reads from it, is bounded by those anchors rather than by a hard-coded date.

### The orientation layer

![ReadMe sheet](docs/images/readme_sheet.png)

The workbook opens on a `ReadMe` sheet that explains the tab colour convention, how to add a row, the rules that keep the formulas intact, and what to check when a number looks wrong. Anyone who downloads the file has the instructions in front of them without needing anything else.

## The technique

*This section is for readers who want the mechanics. Everything above stands without it.*

Every figure on the tracking dashboard is a `SUMIFS` bounded by dates the workbook works out for itself. This is the current-month leads figure for one business group:

```excel
=IFERROR(SUMIFS(DemandTbl[Daily Leads (count)],
                DemandTbl[Business Group], $A14,
                DemandTbl[Date], ">="&DATE($B$4,$B$3,1),
                DemandTbl[Date], "<"&DATE($B$4,$B$3+1,1)), 0)
```

`$B$3` and `$B$4` are the current month and year, both read from `TODAY()`. Nothing is pinned to a specific date, so opening the workbook in a new month moves every comparison window on its own.

The year boundaries are handled explicitly rather than left to fail. In January, the previous month is December of the previous year, and both the month and the year it belongs to have to change together:

```excel
=IF(B3=1,12,B3-1)      previous month
=IF(B3=1,B4-1,B4)      the year that month belongs to
```

The same pattern carries Q1 back to Q4, where the previous-quarter upper bound uses `EDATE` so the boundary lands on the right day. Every ratio is wrapped in `IFERROR`, so a period with no prior data returns a blank rather than a division error.

**Tools:** Microsoft Excel throughout, using structured tables, named ranges, `SUMIFS` with dynamic date windows, `IFERROR` guards, `NA()` to break chart series, data validation, conditional formatting, and native charts with a fixed palette applied consistently per business group.

## Author and contact

**Ndivhuwo**, design, build and documentation.

- Email: [ndivhuwojse@gmail.com](mailto:ndivhuwojse@gmail.com)
- LinkedIn: [www.linkedin.com/in/ndivhuwo-makhavhu](https://www.linkedin.com/in/ndivhuwo-makhavhu)
- GitHub: [github.com/MainDevWork](https://github.com/MainDevWork)

[**Back to the top**](#demand--revenue-insights-console)