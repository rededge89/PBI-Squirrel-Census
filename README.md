# Central Park Squirrel Feeding Risk Dashboard

## Project Overview

This Power BI project analyzes the 2018 Central Park Squirrel Census dataset to help New York City Parks & Recreation identify areas where **"Do Not Feed the Squirrels"** signage may be most useful.

Rather than only reporting raw squirrel sighting counts, this dashboard focuses on the relationship between:

- Squirrel eating behavior
- Squirrel approach behavior toward humans
- Location-based activity patterns by hectare
- Minimum sample-size thresholds for more reliable interpretation

The project was designed as an interview case study for a Power BI Developer role, with emphasis on data modeling, DAX measures, visual storytelling, and operational recommendations.

---

## Business Scenario

NYC Parks & Recreation is concerned that human feeding may be encouraging squirrels to approach people in Central Park.

Because the census took place in October, squirrels were likely preparing for winter. In this analysis, eating behavior is treated as a visible food-consumption signal, while approaching behavior is treated as a potential human-interaction risk signal.

The main business question is:

> Where should Parks & Rec prioritize "Do Not Feed the Squirrels" signs?

---

## Dataset

The dataset contains more than 3,000 squirrel observations from the Central Park Squirrel Census.

### Key fields used

| Field | Description |
|---|---|
| `hectare` | Park grid area where the sighting occurred |
| `lat` / `long` | Latitude and longitude of the sighting |
| `shift` | AM or PM observation shift |
| `age` | Adult, juvenile, or unknown |
| `primary_fur_color` | Main squirrel fur color |
| `eating` | Whether the squirrel was observed eating |
| `foraging` | Whether the squirrel was observed foraging |
| `approaches` | Whether the squirrel approached humans |
| `runs_from` | Whether the squirrel ran from humans |
| `indifferent` | Whether the squirrel was indifferent to humans |

---

## Dashboard Pages

### 1. Executive Summary

![Executive Summary](images/executive-summary.png)

The executive summary provides a high-level view of squirrel activity and feeding risk.

Key visuals include:

- Total squirrel sightings
- Eating rate
- Foraging rate
- Approach rate
- Feeding risk score
- Top hectares by total sightings
- Feeding risk score by hectare
- Total sightings by fur color

This page is intended for a leadership audience that needs quick answers and actionable direction.

---

### 2. Population Hotspots

![Population Hotspots](images/population-hotspots.png)

This page shows where squirrel sightings are concentrated across Central Park.

Key features:

- Heatmap-style location view
- Hectare-level sighting counts
- AM/PM breakdown by hectare
- Slicers for age, fur color, and shift

This page answers:

> Where are squirrels being observed most frequently?

The hotspot view is useful for understanding general activity density, but it does not automatically indicate feeding risk. For that, the report uses behavioral measures such as eating rate and approach rate.

---

### 3. Feeding Risk Plot

![Feeding Risk Plot](images/feeding-risk-plot.png)

This scatterplot is the core analytical view of the report.

Each bubble represents a hectare.

| Visual Element | Meaning |
|---|---|
| X-axis | Eating Rate |
| Y-axis | Approach Rate |
| Bubble size | Total Sightings |
| Bubble color | Feeding Risk Score |
| Slicer | Minimum sightings threshold |

The scatterplot helps separate hectares into behavioral zones:

| Zone | Interpretation |
|---|---|
| High Eating / High Approach | Strong candidate for no-feeding signage |
| High Eating / Low Approach | Food activity zone |
| Low Eating / High Approach | Human interaction risk zone |
| Low Eating / Low Approach | Standard monitoring zone |

A minimum sightings slicer is included to reduce distortion from small sample sizes.

---

### 4. Feeding Risk Map

![Feeding Risk Map](images/feeding-risk-map.png)

This page maps feeding risk geographically by hectare.

Key features:

- Hectare-level location mapping
- Bubble size based on total sightings
- Bubble color based on feeding risk score
- Minimum sightings slicer
- Tooltips showing hectare, eating rate, approach rate, and feeding risk score

This page answers:

> Where should Parks & Rec investigate potential no-feeding sign placement?

---

## Key Measures

### Total Sightings

```DAX
Total Sightings =
COUNTROWS(nyc_squirrels)
```

### Eating Sightings

```DAX
Eating Sightings =
CALCULATE(
    [Total Sightings],
    nyc_squirrels[eating] = TRUE()
)
```

### Eating Rate

```DAX
Eating Rate =
DIVIDE(
    [Eating Sightings],
    [Total Sightings]
)
```

### Approach Sightings

```DAX
Approach Sightings =
CALCULATE(
    [Total Sightings],
    nyc_squirrels[approaches] = TRUE()
)
```

### Approach Rate

```DAX
Approach Rate =
DIVIDE(
    [Approach Sightings],
    [Total Sightings]
)
```

### Foraging Sightings

```DAX
Foraging Sightings =
CALCULATE(
    [Total Sightings],
    nyc_squirrels[foraging] = TRUE()
)
```

### Foraging Rate

```DAX
Foraging Rate =
DIVIDE(
    [Foraging Sightings],
    [Total Sightings]
)
```

---

## Feeding Risk Score

The Feeding Risk Score combines approach behavior, eating behavior, and sighting volume.

```DAX
Feeding Risk Score =
(0.6 * [Approach Rate]) +
(0.3 * [Eating Rate]) +
(0.1 * [Total Sightings Rate])
```

### Weighting logic

| Component | Weight | Reason |
|---|---:|---|
| Approach Rate | 60% | Main signal of squirrel-human interaction risk |
| Eating Rate | 30% | Supporting signal of visible food consumption |
| Total Sightings Rate | 10% | Adds operational importance based on sighting volume |

The score is intentionally weighted toward approach behavior because the business goal is to identify likely signage locations, not simply the busiest squirrel areas.

---

## Minimum Sightings Parameter

To avoid overreacting to small sample sizes, the report includes a disconnected parameter table that allows users to filter visuals by a minimum sighting threshold.

```DAX
Minimum Sightings Parameter =
DATATABLE(
    "Minimum Sightings", INTEGER,
    {
        {1},
        {5},
        {10},
        {15},
        {20},
        {25},
        {30}
    }
)
```

```DAX
Selected Minimum Sightings =
SELECTEDVALUE(
    'Minimum Sightings Parameter'[Minimum Sightings],
    10
)
```

```DAX
Show Hectare by Minimum Sightings =
IF(
    [Total Sightings] >= [Selected Minimum Sightings],
    1,
    0
)
```

This measure is used as a visual-level filter on the feeding risk scatterplot and map.

---

## Hectare Summary Table

A hectare-level summary table was created so the map could show one bubble per hectare instead of one bubble per squirrel sighting.

```DAX
Hectare Summary =
SUMMARIZE(
    nyc_squirrels,
    nyc_squirrels[hectare],
    "Hectare Lat", AVERAGE(nyc_squirrels[lat]),
    "Hectare Long", AVERAGE(nyc_squirrels[long]),
    "Hectare Sightings", COUNTROWS(nyc_squirrels)
)
```

This table supports cleaner hectare-level mapping and helps avoid overplotting individual squirrel observations.

---

## Key Insights

### 1. Eating behavior and approach behavior overlap in specific areas

Squirrels observed eating may be more likely to approach humans than squirrels not observed eating. This supports using eating rate and approach rate together when identifying candidate signage zones.

### 2. Raw sighting hotspots are not always the same as feeding-risk hotspots

The busiest squirrel areas are useful for general monitoring, but they are not automatically the best places for no-feeding signs. Feeding risk is better identified by combining approach rate, eating rate, and total sighting volume.

### 3. Minimum sample-size thresholds matter

Some hectares can show high rates due to very few observations. The minimum sightings slicer helps reduce small-sample distortion and makes the dashboard more reliable for decision-making.

---

## Recommendations

### 1. Prioritize signage in high feeding-risk hectares

Parks & Rec should prioritize "Do Not Feed the Squirrels" signage in hectares where eating behavior and approach behavior overlap.

### 2. Validate high-risk zones in the field

Before installing permanent signage, Parks & Rec should validate high-risk hectares by observing:

- Visitor feeding behavior
- Trash can or picnic area proximity
- Natural food sources
- Foot traffic levels
- Nearby seating or gathering areas

### 3. Improve future survey collection

Future squirrel surveys should include additional structured fields such as:

- Human feeding observed: Yes/No
- Trash can nearby: Yes/No
- Visitor traffic level
- Food source visible
- Tree or habitat type
- Observer ID
- Survey route ID

These fields would help separate natural food behavior from human-influenced feeding behavior.

---

## Limitations

This analysis does not prove that humans are feeding squirrels.

Eating and approaching behavior may also be influenced by:

- Natural food availability
- Habitat differences
- Observer timing
- Visibility
- Weather
- Park foot traffic
- Random variation in squirrel behavior

The Feeding Risk Score should be interpreted as a prioritization tool for field validation, not as definitive proof of human feeding.

---

## Tools Used

- Power BI Desktop
- DAX
- Power Query
- Bing Maps visual
- CSV dataset from the Central Park Squirrel Census

---

## Project Purpose

This project demonstrates:

- Power BI report design
- DAX measure development
- Parameter-driven filtering
- Geospatial visualization
- Behavioral analysis
- Data storytelling
- Operational recommendation development
- Awareness of data limitations

---

## Repository Structure

```text
.
├── data/
│   └── nyc_squirrels.csv
├── images/
│   ├── executive-summary.png
│   ├── population-hotspots.png
│   ├── feeding-risk-plot.png
│   └── feeding-risk-map.png
├── powerbi/
│   └── Central_Park_Squirrel_Feeding_Risk.pbix
└── README.md
```

---

## Final Summary

This dashboard uses the Central Park Squirrel Census to identify potential no-feeding signage priority areas. By combining eating behavior, approach behavior, sighting volume, and sample-size controls, the report moves beyond basic squirrel counts and provides Parks & Rec with a practical framework for monitoring squirrel-human interaction risk.
