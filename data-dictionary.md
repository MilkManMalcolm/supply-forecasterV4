# Sales CSV data dictionary

The sales CSV describes **one product on one historical date per row**. The stable header names below are English in every interface language. Use the downloadable CSV template for the shape of the file; its **two example rows are illustrative and are not enough to train a forecast**. Use the longer sample data to explore the complete workflow.

The built-in sample sales, weather forecasts, holiday flags, events, recipes, and stock settings are **synthetic, illustrative demonstration data**, not verified Hong Kong records or validated commercial recipes. User-uploaded data has user-supplied provenance that the app does not verify. Uploaded product names are kept as provided rather than replaced with sample names or fabricated translations.

## Required columns

All nine columns are required, with no extra columns. Use the exact header names shown, with one header row. Quantity counts refer to individual products, not ingredient weights or supplier packs.

| Column | Meaning and accepted data | Example |
|---|---|---|
| `date` | A real calendar date in strict ISO `YYYY-MM-DD` format. | `2026-01-05` |
| `product_name` | The product name as text; it must not be blank. Keep the name consistent from day to day. | `Pineapple buns` |
| `units_baked` | Nonnegative numeric count of that product baked on that date. | `80` |
| `units_sold` | Nonnegative numeric count of that product sold on that date; it cannot exceed `units_baked`. | `72` |
| `sold_out` | Whether the product is known to have sold out. Use `0`, `1`, `true`, or `false`: `1`/`true` means yes and `0`/`false` means no. | `false` |
| `forecast_temperature_c` | Numeric **forecast** temperature for the target date in degrees Celsius; negative temperatures are valid. This must represent information available before that target date. | `20.5` |
| `forecast_rain_mm` | Nonnegative numeric **forecast** rainfall for the target date, in millimetres, available before that target date. | `2.0` |
| `is_holiday` | Whether the date is a holiday. Use `0`, `1`, `true`, or `false`. | `0` |
| `nearby_event` | Whether a relevant nearby event is expected on that date. Use `0`, `1`, `true`, or `false`. | `1` |

All cells in the required columns should be filled; use `0` for a genuinely zero value, not as a substitute for unknown data. Save the file as comma-separated values (`.csv`), with dates written as text in the required form. If a product name contains a comma, enclose the name in double quotes, as standard CSV requires.

For example, the following two rows show the format only. They do **not** meet the minimum history requirement:

```csv
date,product_name,units_baked,units_sold,sold_out,forecast_temperature_c,forecast_rain_mm,is_holiday,nearby_event
2026-01-05,Pineapple buns,80,72,false,20.5,2.0,0,0
2026-01-06,Pineapple buns,80,80,true,21.0,0.0,0,1
```

## Before importing

- The file limit is **5 MB** (5 MiB, or 5,242,880 bytes). Supply **at least 56 days of daily history for each product**; the minimum is a validity threshold, not a promise of good accuracy. After the first 28 history days used to build earlier-sales features, each product also needs at least 16 non-stockout targets for fitting. If there are too few, add more complete history.
- There must be **at most one row for each product/date pair**. Each product’s history must be consecutive without missing dates, and **all products must have the same first date and the same last date**. Do not silently fill a missing day with zero or drop newer records to force dates to match.
- Use valid `YYYY-MM-DD` dates, valid numbers, nonnegative baked/sold quantities and rainfall, and the flag values above. `units_sold` must not exceed `units_baked`. Fix any import error rather than guessing missing values.
- The planning/as-of date is the latest complete historical date. The seven forecast dates begin the following day, so a demonstration based on older history can produce a historical planning window rather than the visitor’s actual next week.
- Daily weather and event information should be consistent for the same date across products. **Historical weather fields must be forecasts available before each target day**, not weather observations inserted after the day passed. Forecast-period temperature, rainfall, holiday and event conditions are assumptions to review, not a live weather feed.

Recipes, ingredient stock, pack sizes, and delivery lead times are edited in the application; they are not extra CSV columns or separate CSV imports. Newly uploaded product names may be forecast, but their recipes must be supplied before complete ingredient coverage can be claimed. Partial advice must not be mistaken for a complete order.

## What the records can and cannot tell us

Previous sales and rolling sales summaries may be used as inputs only when they come from earlier dates. A target day’s actual `units_sold`, `units_baked`, and `sold_out` are not predictors for that same day. Date-based weekday and seasonal features can be derived from the date.

A known stockout can hide demand: `units_sold` then tells us what was sold, not how much customers would have bought with unlimited stock. Flag actual known stockouts; equal baked and sold counts alone do not prove one. Flagged targets can be excluded from fitting while their past records remain useful context. This avoids treating censored sales as full demand, but loses training examples and can still leave bias. No exact unmet demand is inferred.

Forecasts and order suggestions are estimates for demonstration and planning. Their quality depends on the history and assumptions; missing records, demand changes, and the recursive use of earlier predictions can make later days less reliable. They do not prove real-world accuracy or business savings.
