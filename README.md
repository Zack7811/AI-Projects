# Car Demand Analyst (Claude Skill)

Turns a vehicle sales or listings dataset into a ranked view of **what sells, where, when, and what to push**. Claude acts as a Senior Data Analyst: it validates the data, runs the analysis, and gives recommendations with caveats.

Works for any market, as long as your data has the fields listed below.

## What you get

- Top 10 cars (brand and brand+series) and top 10 model years
- Best months by volume, with a warning when the pattern is a data artifact
- Province/region, then district/city, ranking
- MoM trend line (YoY only when two full years exist)
- Mileage, price, age and colour mix
- Optional demand-strength score (volume, growth, price stability)
- "What to push" recommendations with a confidence level
- Optional outputs: `.xlsx` workbook, `.docx` report, or markdown summary

## Install

Copy the folder to your Claude Code skills directory:

```bash
mkdir -p ~/.claude/skills/car-demand-analyst
cp SKILL.md README.md ~/.claude/skills/car-demand-analyst/
```

Claude loads it automatically when your request matches. You can also paste the contents of `SKILL.md` into any chat.

## Data requirements

| Priority | Fields |
|---|---|
| **Required** | Vehicle name (brand, series or trim), date or month, one row per sale or listing |
| **Strongly useful** | Model year, mileage (KM), price, province/region, district/city |
| **Optional** | Colour, fuel, gearbox, body type, seller type, engine size, buyer ID, plus the push-signal fields below |

Accepted sources: `.xlsx`, `.csv`, or a BigQuery table you have access to. Missing optional fields are skipped and reported.

## How to use

Share the file and ask in plain language. Examples:

- "Which cars sell the most in this dataset? Rank the top 10."
- "What is in demand by province, and which cars should we push there?"
- "Show the MoM trend and tell me if there is seasonality."
- "Build an Excel workbook and a Word report from this file."

### Workflow

1. **Ask:** Claude inspects the file and asks the questions below, including your output format.
2. **Analyse:** runs the core analysis with the fields you have.
3. **Deliver:** gives you the format you picked.
4. **Enrich (optional):** afterwards Claude suggests ways to strengthen the recommendations and asks about your own metrics and definitions.

### Output formats (Claude always asks)

| Option | Result |
|---|---|
| Quick answer | Short chat reply with key rankings and recommendations |
| Report (.docx) | Written report with tables and charts |
| Google Sheet | Sheet per question with charts (falls back to `.xlsx` if no Google connector) |
| Excel (.xlsx) | Workbook with native charts, summary and assumptions |
| CSV | One flat file per table, no charts |

You can pick more than one.

### Questions Claude asks first

Claude will inspect the file and ask only what the data cannot answer:

0. Which output format do you want (above)?

1. Is 1 row = 1 sale, or 1 listing? (If there is no sale or customer ID, 1 row is treated as 1 sale, as a proxy.)
2. Is the vehicle field a brand, series or full trim?
3. Is "Model" a model name or a model year?
4. How many years does the data cover?
5. Whether you are a dealership, marketplace or manufacturer, since push logic differs.
6. Optionally, any business bands (for example mileage tiers) and translation tables (colours, fuel).

## Business rules (optional)

Only needed if you have your own definitions. Without them, Claude uses labelled default bands and says so. If you supply them, Claude uses them exactly. Examples:

- KM `< 50,000` = new-ish, `100,000-200,000` = average, `> 200,000` = used
- Gaps between bands (here 50k-100k) are labelled explicitly rather than guessed
- Aging thresholds (for example 45 / 60 / 90 days), floorplan rate, margin targets
- A colour translation table, such as the Turkish-to-English one used in the example

## Push signals (optional, unlock stronger recommendations)

None of these are required. Claude delivers your first result with what the file already has, then shows a **Signal Availability table** (Available / Proxy possible / Missing) and suggests ways to enrich the recommendations. It also asks how **you** define your own metrics (for example aged-stock days or margin), since definitions differ by company, and uses yours instead of the defaults. Then it updates your report or sheet.

| Group | Fields that help |
|---|---|
| **Inventory and aging** | Date listed or acquired, stock on hand, floorplan/holding cost, planned model-year changeover |
| **Market and competition** | Competitor or market price, views / VDP clicks / leads / test drives, local stock of the same trim or colour |
| **Financial and incentives** | Acquisition and reconditioning cost, manufacturer rebates, volume-bonus targets |
| **Vehicle-specific** | Multi-year sales history (for seasonality), recall or service status |

Recommendations are tagged by confidence:

- **Tier 1, demand only:** what sells (volume, geography, trend)
- **Tier 2, plus inventory and market:** what to push now (aging, market-day supply, price-to-market, scarcity)
- **Tier 3, plus margin and incentives:** what to push for profit (gross profit, floorplan cost, rebates, bonus targets)

When a field is missing, Claude uses a labelled proxy where one exists (for example days since listing as inventory age) or asks you for the field. It never invents a signal.

## Push calendar

Ask "what should we push by month or season" and Claude returns a table of month, car to push, the metric behind it, region focus and confidence tier. This needs at least 2 years of data; with less, the calendar is marked directional and partial months are excluded.

## Built-in data checks

Every analysis reports:

- Duplicate rows removed
- Corrupted values (price, KM) and how they were handled
- Partial months and heavy skew toward one month
- Capped extracts (many categories at nearly identical counts), treated as ties
- Concentration of sales in the top 1, 3 and 10

## Limits to know

- **Listings are not sales.** A snapshot of active listings shows supply, not completed demand.
- **No YoY with one year of data.** Claude provides MoM only and says so.
- **Gaps under about 1 point between ranks are ties**, not real differences.
- **No personal data in outputs.** Names, phone numbers, plates, VINs and exact addresses are dropped or aggregated. Anonymise raw logs before sharing.
- Currency stays in the local currency of the market. No conversion unless requested.

## Example result (Turkey used cars, 2020)

Roughly 49.5k rows after removing 51 duplicates; 1 row = 1 sale (proxy):

| Question | Result |
|---|---|
| Top series | Hyundai Accent (6.9%); ranks #2-#9 within 1 pt |
| Top model years | 2012, 2011, 2013 |
| Top province | İstanbul (25.7%) |
| Top district | Çankaya, Ankara |
| Top colour | White (30%); 26% blank |
| Caveat | Jul-Aug = 91% of rows, a scrape-window artifact; no YoY |

This is for format reference only. Claude always recomputes from your data.

## Files

| File | Purpose |
|---|---|
| `SKILL.md` | The skill instructions Claude follows |
| `README.md` | This guide |

## Troubleshooting

- **Skill does not trigger:** mention the file and the goal, for example "analyse this car sales file and rank top sellers", or invoke it as `/car-demand-analyst`.
- **Rankings look flat:** the extract is probably capped per category; ask Claude to check the coverage section.
- **Peak month looks extreme:** check whether the file is a listings snapshot; request a full-year source.
- **Columns not recognised:** tell Claude which column is the vehicle, date, location and price.
