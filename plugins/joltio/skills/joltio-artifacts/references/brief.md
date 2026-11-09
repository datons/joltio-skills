# Dashboard brief

"Energy market analysis" is not a question. A dashboard that tries to show the whole market ends up as a price chart, a mix chart and a map of Europe that help nobody decide anything. Before writing any UI, agree with the user on one brief and save it as `BRIEF.md` in the project folder. The CLI does not upload `.md` files, so the brief stays next to the source without reaching the published artifact.

## The method

Work through these five steps in order, with the user, one at a time. Do not fill a step on the user's behalf; propose and let them correct.

1. **Who.** One concrete reader: "the asset manager of a 50 MW solar portfolio", not "the sector" or "analysts".
2. **Decision.** What that reader decides with the dashboard, and how often: daily bidding, monthly performance review, a PPA negotiation. If there is no decision, say so: the dashboard is context, and should stay small.
3. **Main question.** One question, written as a question, answered by one number. "How much is my energy really worth this month?", not "Prices and generation".
4. **Why questions.** Two to four questions that explain the main number when it moves. Each one is also answered by a number or a single chart.
5. **Evidence.** For each question: the measure, its unit, the period, and the governed function that returns it. Run that function with `run_function` before writing it down. A question without a function is a gap: report it, do not invent the data.

## Rules for the build

- Every chart, number and table answers a question in the brief. Anything that answers none is removed, however interesting.
- The main question and its number come first, above the fold, with the period it covers.
- Each number states its unit, its period and the function it comes from.
- Comparisons carry their reference: against the average, the previous period or a target the user named. A number with no reference cannot tell the reader whether to act.
- Name markets and programs exactly as Joltio Settlements does: MD, RT (split into RT1, RT2 and RT5), S1, S2, S3, MIC (continuous), Balance and final. Do not present other I90 or OMIE programs (PHFC, P48CIERRE, individual intraday sessions) as markets of their own; if the brief needs one, ask the user first.
- Explain every visual encoding without requiring hover: color, stroke style (solid, dashed, dotted), width, marker shape or size, fill, pattern, opacity, shaded bands and reference lines. Up to four series, label each one directly at the line end or on the bar; with more, or when labels collide, place a legend above the chart. The legend reproduces the exact mark, follows the visual order of the series, uses the market names above and states the unit when series differ. Never rely on color alone. A single-series chart with a clear title and axis needs no legend.
- If the brief changes while building, update `BRIEF.md` first, then the UI.

## Starting points for the electricity market

These are starting points to propose in step 3, not a menu to copy. Confirm the reader and the decision first.

| Reader | Decision | Main question | Typical why questions |
|---|---|---|---|
| Renewable asset owner | Sell spot or sign a PPA, curtail or not | How much is my energy really worth, and why does it differ from the average price? | Hours at zero or negative price; solar and wind share of the mix; captured price trend over years; cost of imbalances and balancing services |
| Market representative or trader | What to bid tomorrow and in which market | Where is the margin tomorrow: day-ahead, intraday or balancing? | Day-ahead versus intraday spread; balancing prices and volumes; expected renewables and demand |
| Industrial consumer or retailer | When to consume, how much to hedge | How much will energy cost me, and in which hours? | Hourly price profile; forward versus spot; peak and off-peak spread |
| General market follower | None operational | How does the market compare with normal? | Price versus the same period last year; mix shift; neighbouring prices |

## BRIEF.md template

```markdown
# <Dashboard name>

- **Reader:** <one concrete person or role>
- **Decision:** <what they decide with it> · **Cadence:** <daily / weekly / monthly / one-off>
- **Main question:** <one question?>
- **Headline number:** <measure, unit, period> · **Reference:** <what it is compared with>

## Why questions

### <question?>
- Measure: <measure, unit>
- Period: <period>
- Function: `<slug>`

### <question?>
- Measure: <measure, unit>
- Period: <period>
- Function: `<slug>`

## Gaps

<questions with no governed function yet, and what that function should return; "None" if every question is covered>

## Out of scope

<what this dashboard deliberately does not answer>
```

## Worked example

```markdown
# Captured price of a solar portfolio

- **Reader:** Asset manager of a solar portfolio in Spain
- **Decision:** Whether to keep selling spot or close a PPA · **Cadence:** Monthly
- **Main question:** How much is my solar energy really worth this month?
- **Headline number:** Solar captured price, €/MWh, current month · **Reference:** Day-ahead average price for the same month

## Why questions

### How far below the average price is it?
- Measure: captured / average ratio, %
- Period: month
- Function: `<captured price function>`

### How many hours went for nothing?
- Measure: hours with day-ahead price ≤ 0
- Period: month
- Function: `<price summary function>`

### Is it getting worse?
- Measure: captured / average ratio, %
- Period: monthly, last 36 months
- Function: `<captured price function>`

### How much solar was in the mix?
- Measure: solar share of generation, %
- Period: month
- Function: `<generation mix function>`

## Gaps

No governed function returns the solar captured price yet. It needs the hourly day-ahead price weighted by hourly solar output, per month.

## Out of scope

Plant-level output, imbalances and balancing costs: they need the portfolio's own units, a later dashboard.
```
