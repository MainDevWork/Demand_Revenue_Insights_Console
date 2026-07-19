**Built a self-updating Excel console that consolidates the leads and revenue produced by marketing campaigns run across multiple divisions, and reports them on two dashboards that rebuild themselves from a single set of daily inputs**

**This replace scattered, manually maintained spreadsheets with one centralised source of information, so the Brand and Marketing division can see at any moment which campaigns are generating demand, which are generating revenue, and how each is trending month on month and quarter on quarter.**

**Note:** all data in this repository is fictitious. Read the details in the [**About the data**](#about-the-data) section.

---

## Summary

The Brand and Marketing division coordinates campaigns on behalf of several other divisions, but every team was tracking its own leads and revenue in its own files, which left the division with no single, comparable view of performance. I built a single Excel workbook that acts as a reporting console: each team enters one row per day, and every report, current month, quarter, year-to-date totals, movement percentages, trend charts and KPI tiles, is calculated automatically from those rows. The tool is anchored to the current date, so when a new month or quarter begins, every comparison window moves on its own and the workbook needs no maintenance at period ends.

## Background and problem statement

The Brand and Marketing division runs marketing campaigns through several revenue-generating divisions at once: Primary Markets, Secondary Markets, Information Services and Issuer Services on the revenue side, and demand-generation teams such as JSE SheInvests, Digital Marketing, Events and Marketing Services on the leads side. The division approached me because campaign results were scattered and could not be compared.

I identified three obstacles:

- **Performance lived in separate files.** Each team recorded its own leads and revenue, in its own format and on its own schedule, so no single view of campaign performance existed.
- **Reporting was manual and repetitive.** Every month-end summary was rebuilt by hand, which repeated effort and invited error.
- **Definitions were inconsistent.** "This month" or "quarter to date" meant different things in different files, so the numbers did not reconcile and shortlists for investment reflected individual judgement rather than an agreed standard.

## Deliverables

| File | Description |
|---|---|
| [MCA_Revenue_Demand_Generation_Report.xlsx](MCA_Revenue_Demand_Generation_Report.xlsx) | The console: an eight-sheet Excel workbook containing both dashboards, both calculation engines, the input sheets and an in-workbook ReadMe orientation page |
| [docs/USER_GUIDE.pdf](docs/USER_GUIDE.pdf) | Step-by-step instructions for entering data, reading both dashboards and running the monthly review |
| [docs/BUSINESS_CONTEXT.pdf](docs/BUSINESS_CONTEXT.pdf) | Who the tool serves, the problem it replaced and the value it delivers |

## The solution

I built the whole tool inside Excel, structured in three layers so that logic and presentation never interfere with each other: raw daily inputs feed two calculation engines, and the engines feed two dashboards. A user only ever types into the two input sheets; everything else builds itself.

### Tracking dashboard

Current month, current quarter and year-to-date totals for every business group, with colour-coded month-to-month and quarter-to-quarter percentage movements (green for growth, red for decline), and written *Insights* and *Looking Ahead* commentary refreshed at each monthly review.

![Tracking dashboard](docs/images/tracking_dashboard.png)

### Analysis dashboard

A KPI summary, monthly trend lines per business group, year-to-date contribution by group, cumulative revenue, quarterly revenue, and written key insights. Every business group keeps the same colour across every chart, so the views can be read together.

![Analysis dashboard](docs/images/analysis_dashboard.png)

### How the console is built

| Layer | Sheets | What it does |
|---|---|---|
| Input | `Demand_Input`, `Revenue_Input` | Two Excel tables: date, business group, campaign/stream, daily value. Drop-down validation restricts categories to approved values and rejects negative or non-numeric entries |
| Engine | `Calculations`, `Analysis_Calc` | `SUMIFS` over the named tables, anchored to `TODAY()`, producing month, quarter and year-to-date figures (including the January and Q1 year-wrap cases) |
| Presentation | `Dashboard`, `Analysis` | Reference the engines only; contain no calculations of their own |
| Reference | `Lists`, `ReadMe` | Approved category values that power validation, and an in-workbook orientation page |

## Value delivered

The console converts scattered, inconsistent record-keeping into one decision-ready view. Specifically, it:

- establishes a **single source of truth**, so every division's campaign performance is comparable at a glance under one consistent set of metric definitions;
- **saves time every month**, changing month-end reporting from rebuilding a summary by hand to reading one and writing a short commentary;
- **surfaces the decisions**, making over- and under-performing campaigns visible early — for example, that one campaign delivers roughly 29% of year-to-date revenue, or that a strong quarter was driven by a single month;
- **protects data quality by design**, using validation at the point of entry to prevent the misspelled categories and invalid values that silently corrupt manual reports;
- **survives staff changes**, because the in-workbook ReadMe page and the standalone documentation make a new user productive without a handover meeting.

## The process

The workbook runs end to end from the two input tables. Nothing downstream is ever edited by hand.

```
Demand_Input  (daily leads)          Revenue_Input  (daily revenue, Rands)
   one row per group per day             one row per group per day
   drop-down + numeric validation        drop-down + numeric validation
        │                                     │
        ├──────────────┬──────────────────────┤
        ▼              ▼                       ▼
   Calculations                        Analysis_Calc
   TODAY()-anchored SUMIFS:            monthly matrix, quarterly totals,
   month / quarter / YTD per group    YTD contribution, KPI helpers
        │                                     │
        ▼                                     ▼
   Dashboard                           Analysis
   tracking view: totals + movement    KPI tiles, trends, contribution,
   percentages + commentary            cumulative + quarterly + insights
```

**Tools:** Microsoft Excel throughout — structured tables, named ranges, `SUMIFS` with dynamic date windows, `INDEX`/`MATCH`, `IFERROR` guards, data validation, conditional formatting, and native charts using a colourblind-safe palette applied consistently per business group.

Because every period window is anchored to the current date, reopening the workbook in a new month refreshes every comparison automatically. Re-running the same inputs always reproduces the same reports, so the console can be refreshed as new data arrives without any loss of consistency.

## About the data

**The dataset in this repository is fictitious.** It was generated to demonstrate the design and contains no real business figures. The structure — sheets, tables, categories, formulas, validation, dashboards and charts — is identical to the version used internally, so the tool runs unchanged against this data. All figures quoted above can be verified directly against the workbook.

[**Back to the top**](#cross-departmental-demand--revenue-insights-console)
