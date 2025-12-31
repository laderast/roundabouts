# Roundabouts Data Analysis

**Date:** December 31, 2025  
**Dataset:** `roundabouts_clean` (27,887 observations, 18 variables)

## Overview

This analysis explores a global dataset of roundabouts, examining their distribution across countries, construction trends over time, types, and city-level patterns.

## Key Findings

### 1. Country Distribution

- **United States dominates** with over 12,000 roundabouts in the dataset
- **Top 10 countries by number of roundabouts:**
  1. United States (~12,000+)
  2. Australia (~3,000-4,000)
  3. United Kingdom (~2,000)
  4. Sweden, Canada, New Zealand, Netherlands, Russia, Norway, France (each with several hundred to ~1,000)

### 2. City Distribution

**Top 10 cities with most roundabouts (excluding unincorporated areas):**

1. **Auckland, New Zealand** - 202 roundabouts (clear leader)
2. **Carmel, USA** (Indiana) - 146 roundabouts ("Roundabout Capital of the USA")
3. **Gothenburg, Sweden** - 117 roundabouts
4. **Calgary, Canada** - 101 roundabouts
5. **Lincoln, USA** - 89 roundabouts
6. **Miami, USA** - 88 roundabouts
7. **Portland, USA** - 82 roundabouts
8. **Colorado Springs, USA** - 79 roundabouts
9. **Seattle, USA** - 75 roundabouts
10. **Melbourne, Australia** - 73 roundabouts

**City-level insights:**
- Auckland, NZ dominates with significantly more roundabouts than any other city
- 7 of the top 10 cities are in the United States
- ~275 roundabouts are in unincorporated areas (outside city limits)
- Carmel, Indiana is famous for its aggressive roundabout adoption program

### 3. Data Quality

- **Missing completion dates:** Large number of roundabouts (~16,000) have `year_completed = 0`, indicating missing data
- **Complete data available for:** ~11,000-12,000 roundabouts with actual completion years
- **Address data:** Cities extracted from the first part of the `address` column (before comma)

### 4. Construction Timeline (1995+)

- **Exponential growth:** Roundabout construction increased dramatically from 1995 onwards
- **Peak construction:** 2015-2020, with up to 650+ roundabouts completed per year
- **Recent decline:** Sharp drop after 2020 (likely incomplete recent data or reporting lag)
- **Very few historical roundabouts:** Only a handful recorded before 1990

### 5. Country-Specific Patterns (1995+)

- **United States:** Most consistent and sustained construction activity, peaking 2015-2020
- **Canada:** Similar pattern to US but smaller scale (50-150 per year)
- **Other countries:** Limited or incomplete completion date data (Australia, UK, Sweden, Netherlands show sparse records)

### 6. Roundabout Types

- **Standard "Roundabout"** type dominates: >25,000 roundabouts (~90%+)
- **Other types are rare:**
  - Other: ~2,000-3,000
  - Traffic Calming Circle: very few
  - Signalized Roundabout/Circle: very few
  - Rotary: very few
  - Unknown: very few

### 7. Type Trends Over Time

- **Standard roundabouts** drive all growth from 1995-2020
- **Alternative types** remain consistently low (<50 per year)
- Construction of all types declined sharply after 2020

## Visualizations Created

1. ✅ Histogram of all countries
2. ✅ Histogram of top 10 countries
3. ✅ Visdat plot showing data structure and missing values
4. ✅ Histogram of `year_completed` (showing year 0 issue)
5. ✅ Histogram of `year_completed` excluding year 0
6. ✅ Faceted histogram by country (top 10, filtered to 1995+)
7. ✅ Histogram of roundabout types
8. ✅ Line plot of type vs year_completed (1995+)
9. ✅ Interactive plotly version with tooltips
10. ✅ Histogram of top 10 cities (with country labels)
11. ✅ **Final polished plot:** Top 10 cities with custom design

### Final Plot Design Features
- **Custom color palette:** Sophisticated muted tones
  - Dark blue-grey (#3d4566) - Australia
  - Medium blue-grey (#4d5e80) - Canada
  - Red (#cc3d3d) - New Zealand
  - Yellow (#ffc44d) - Sweden
  - Orange (#ff9040) - USA
- **Typography:** Overpass font from Google Fonts (bold labels)
- **In-bar labels:** White count numbers displayed inside each bar
- **Minimal design:** No grid lines, no x-axis, clean presentation
- **Title:** "You spin me round round baby - Top 10 roundabout cities"

## Code Snippets

### Loading packages
```r
library(dplyr)
library(ggplot2)
library(visdat)
library(plotly)
library(stringr)
library(showtext)
```

### Setting up custom fonts
```r
# Add Overpass font from Google Fonts
font_add_google("Overpass", "overpass")
showtext_auto()
```

### Top 10 countries analysis
```r
top_10_countries <- roundabouts_clean |>
  count(country, sort = TRUE) |>
  slice_head(n = 10) |>
  pull(country)
```

### Extracting cities from addresses
```r
# Extract city (first part before comma)
roundabouts_clean |>
  mutate(city = str_extract(address, "^[^,]+"))
```

### Final polished city plot
```r
# Custom color palette
custom_palette <- c("#3d4566", "#4d5e80", "#cc3d3d", "#ffc44d", "#ff9040")

# Country abbreviation mapping
country_abbrev <- c(
  "New Zealand" = "NZ",
  "United States" = "USA",
  "Sweden" = "SWE",
  "Canada" = "CAN",
  "Australia" = "AUS",
  "United Kingdom" = "UK"
)

# Create final plot
p_final <- roundabouts_clean |>
  mutate(city = str_extract(address, "^[^,]+")) |>
  filter(city != "(unincorporated)") |>
  count(city, country, sort = TRUE) |>
  slice_head(n = 10) |>
  mutate(country_abbr = country_abbrev[country],
         city_label = paste0(city, " (", country_abbr, ")")) |>
  ggplot(aes(x = reorder(city_label, n), y = n, fill = country_abbr)) +
  geom_col() +
  geom_text(aes(label = n), hjust = 1.2, color = "white", 
            size = 5, family = "overpass", fontface = "bold") +
  coord_flip() +
  scale_y_continuous(limits = c(0, 225), expand = c(0, 0)) +
  scale_fill_manual(values = custom_palette) +
  labs(title = "You spin me round round baby - Top 10 roundabout cities") +
  theme_minimal(base_family = "overpass") +
  theme(panel.grid = element_blank(),
        legend.position = "none",
        plot.title = element_text(family = "overpass"),
        axis.title = element_blank(),
        axis.text.x = element_blank(),
        axis.text.y = element_text(family = "overpass", face = "bold"))

# Save plot
ggsave("plot_final_top10_cities_custom.png", p_final, width = 10, height = 6, dpi = 300)
```

### Filtering valid completion years
```r
roundabouts_clean |>
  filter(year_completed > 1995) |>
  # ... further analysis
```

### Interactive plotly plot
```r
plot_data <- roundabouts_clean |>
  filter(year_completed > 1995) |>
  count(year_completed, type)

p <- plot_ly(plot_data, 
             x = ~year_completed, 
             y = ~n, 
             color = ~type,
             type = 'scatter',
             mode = 'lines+markers',
             text = ~paste("Year:", year_completed, 
                          "<br>Type:", type,
                          "<br>Number of Roundabouts:", n),
             hovertemplate = '%{text}<extra></extra>')
```

## Saved Plot Files

All plots saved as high-resolution PNG files (300 DPI):

1. `plot1_top10_countries.png` - Bar chart of top 10 countries
2. `plot2_year_completed.png` - Histogram of completion years (excluding year 0)
3. `plot3_faceted_by_country.png` - Faceted histogram by country (1995+)
4. `plot4_roundabout_types.png` - Bar chart of roundabout types
5. `plot5_type_over_time.png` - Line plot showing type trends over time
6. `plot6_top10_cities.png` - Color-coded bar chart of top 10 cities
7. `plot_final_top10_cities_custom.png` - **Final polished version** with custom design

## Insights & Interpretation

1. **Modern traffic engineering trend:** The exponential growth after 1995 reflects the adoption of roundabouts as a safer alternative to traditional intersections, particularly in North America.

2. **US leadership:** The United States has embraced roundabouts more extensively than other countries in this dataset, especially since 2000. This is evident both at the country level and city level (7 of top 10 cities are US cities).

3. **Auckland's dominance:** Auckland, New Zealand stands out with 202 roundabouts - more than any other individual city, despite New Zealand's relatively small population. This reflects strong urban planning commitment to roundabouts.

4. **Carmel, Indiana:** Lives up to its reputation as the "Roundabout Capital of the USA" with 146 roundabouts, making it #2 globally among cities.

5. **Data completeness varies:** Completion year data is much more complete for the US and Canada compared to other countries, which may indicate differences in data collection practices.

6. **Standardization:** The overwhelming preference for standard roundabout design (vs. specialized types) suggests convergence on proven design standards.

7. **Suburban/rural adoption:** The ~275 roundabouts in "(unincorporated)" areas suggest significant adoption outside traditional city boundaries, likely in suburban and rural contexts.

8. **Recent data caveat:** The sharp decline after 2020 should be interpreted cautiously, as it likely reflects data reporting lag rather than an actual construction slowdown.

## Session Variables

Current R session includes:
- `roundabouts_clean` - Main dataset (27,887 x 18)
- `country_abbrev` - Country abbreviation mapping (6 countries)
- `custom_palette` - Custom color palette (5 colors)
- `top_10_countries` - Vector of top 10 countries
- `plot_data` - Prepared data for time series plotting (101 x 3)
- `p` - Interactive plotly object
- `p1` through `p6` - Individual ggplot objects
- `p_final` - Final polished ggplot object

## Design Decisions

### Typography
- **Font:** Overpass (Google Fonts) - Modern, clean sans-serif
- **Weight:** Bold for all labels to improve readability
- **Hierarchy:** Title remains regular weight, labels are bold

### Color Palette
Custom palette chosen for sophistication and differentiation:
- Cool blues/greys for Australia and Canada
- Warm red for New Zealand (standout leader)
- Bright yellow for Sweden
- Orange for USA cities (warm, energetic)

### Layout Choices
- **No grid lines:** Reduces visual clutter
- **No legend:** Country codes in labels make legend redundant
- **In-bar labels:** White numbers provide exact values without requiring axis reference
- **No x-axis:** Clean, minimal design focuses attention on comparisons

## Next Steps for Analysis

- Investigate geographic clustering within countries (using lat/lon coordinates)
- Create interactive map visualization showing roundabout locations
- Analyze relationship between roundabout size/diameter and type
- Examine state-level patterns within the United States
- Investigate the ~16,000 roundabouts with missing completion dates
- Explore other variables in the dataset (name, angle, code, source, etc.)
- Compare roundabout density per capita across countries/cities
- Analyze temporal patterns: Are certain times of year more common for completions?
- Create animated time-lapse of roundabout construction over time
