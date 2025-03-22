### README.md

```markdown
# BMW X-Series Rotor Analysis

This repository contains a detailed analysis of BMW X-series rotor replacement service data and customer satisfaction (CSI) across the U.S. from 2021–2025. Built with RStudio, it features advanced visualizations—heatmaps, line plots, and interactive maps—to explore repair trends, common issues, and regional satisfaction for models X1 through X6.

## Project Structure

- **Data Files**:
  - `common_issues_expanded.csv`: Rotor-related issues (e.g., warped rotors) with frequency, mileage, and repair costs.
  - `csi_by_region_expanded.csv`: Customer Satisfaction Index (CSI) scores by U.S. region (2021–2025).
  - `service_patterns_expanded.csv`: Service frequency and mileage for rotor replacements by model and year.

- **R Scripts**:
  - `heatmap_common_issues.R`: Faceted heatmap of common issues by model and frequency.
  - `csi_advanced_plot.R`: Multi-faceted line plot of CSI trends with confidence intervals.
  - `csi_globe_map.R`: 3D globe map of 2025 CSI scores for major cities.
  - `csi_terrain_map.R`: GPS-style terrain map of 2025 CSI scores for key U.S. cities.

- **Outputs**:
  - `common_issues_heatmap_no_x7.png`: Heatmap visualization.
  - `csi_trends_advanced.png`: CSI trends plot.
  - `csi_globe_2025.html`: Interactive globe map.
  - `csi_terrain_map_2025.html`: Interactive terrain map.

## Setup

1. **Prerequisites**:
   - R and RStudio installed.
   - Required packages: `ggplot2`, `dplyr`, `stringr`, `leaflet`, `plotly`, `htmlwidgets`.
     ```R
     install.packages(c("ggplot2", "dplyr", "stringr", "leaflet", "plotly", "htmlwidgets"))
     ```

2. **Clone the Repo**:
   ```bash
   git clone https://github.com/[YourUsername]/BMWXSeriesRotorAnalysis.git
   cd BMWXSeriesRotorAnalysis
   ```

3. **Run Scripts**:
   - Place `.csv` files in your working directory (e.g., `C:/Users/Veteran`).
   - Source each `.R` script in RStudio (`Ctrl+Shift+S`) to generate visualizations.

## Data Files

### `common_issues_expanded.csv`
```csv
Issue,Model,Frequency,Mileage_Range,Cost_to_Fix
Warped Rotors,X1,Medium,35-55,700-1000
Warped Rotors,X2,Medium,35-50,650-950
Warped Rotors,X3,High,40-60,800-1200
Warped Rotors,X4,High,40-55,750-1100
Warped Rotors,X5,High,40-60,1000-1500
Warped Rotors,X6,High,45-60,1100-1600
Squealing Pads,X1,Medium,30-50,600-900
[etc., full dataset excluding X7]
```

### `csi_by_region_expanded.csv`
```csv
Region,Year,CSI_Score,Sample_Size,Top_Complaint
Northeast,2021,75,200,Cost
Midwest,2021,70,150,Vibration
South,2021,80,250,Squealing Pads
Southeast,2021,82,220,Squealing Pads
Southwest,2021,78,180,Warped Rotors
West,2021,76,190,Warped Rotors
Northwest,2021,74,160,Vibration
[etc., full dataset up to 2025]
```

### `service_patterns_expanded.csv`
```csv
Year,Model,Mileage_at_Replacement,Replacements_per_Year,Service_Type
2021,X1,40,0.5,Dealer
2021,X2,38,0.5,Independent
2021,X3,45,0.5,Mobile
2021,X4,42,0.5,Dealer
2021,X5,50,0.5,Independent
2021,X6,48,0.5,Mobile
[etc., full dataset excluding X7]
```

## Visualizations

### 1. Heatmap of Common Issues
**Script**: `heatmap_common_issues.R`
```R
library(ggplot2)
library(dplyr)
library(stringr)
setwd("C:/Users/Veteran")
data <- read.csv("common_issues_expanded.csv", fileEncoding = "latin1")
data <- data %>% filter(Model != "X7")
data$Mileage_Range <- iconv(data$Mileage_Range, to = "UTF-8", sub = "-")
data$Cost_to_Fix <- iconv(data$Cost_to_Fix, to = "UTF-8", sub = "-")
data <- data %>% mutate(Cost_Min = as.numeric(str_extract(Cost_to_Fix, "^\\d+")), Cost_Max = as.numeric(str_extract(Cost_to_Fix, "\\d+$")), Cost_Avg = (Cost_Min + Cost_Max) / 2)
data$Model <- factor(data$Model, levels = c("X1", "X2", "X3", "X4", "X5", "X6"))
data$Issue <- factor(data$Issue, levels = unique(data$Issue))
data$Frequency <- factor(data$Frequency, levels = c("Low", "Medium", "High"))
p <- ggplot(data, aes(x = Model, y = Issue, fill = Cost_Avg)) +
  geom_tile(color = "white", linewidth = 0.5) +
  geom_text(aes(label = Mileage_Range), size = 3, color = "black") +
  facet_wrap(~Frequency, ncol = 3, scales = "free") +
  scale_fill_gradient(low = "lightblue", high = "darkred", name = "Avg Cost to Fix ($)", limits = c(500, 2200), breaks = seq(500, 2200, 500)) +
  labs(title = "BMW X-Series Rotor Issues: Cost and Mileage by Frequency (Excl. X7)", x = "X-Series Model", y = "Common Issue") +
  theme_minimal() + theme(axis.text.x = element_text(angle = 45, hjust = 1), panel.grid = element_blank(), strip.background = element_rect(fill = "grey90", color = "black"), strip.text = element_text(size = 10, face = "bold"), legend.position = "bottom")
print(p)
ggsave("common_issues_heatmap_no_x7.png", width = 12, height = 8, dpi = 300)
```

### 2. CSI Trends Plot
**Script**: `csi_advanced_plot.R`
```R
library(ggplot2)
library(dplyr)
library(stringr)
setwd("C:/Users/Veteran")
data <- read.csv("csi_by_region_expanded.csv", fileEncoding = "latin1")
csi_summary <- data %>% group_by(Region, Year) %>% summarise(Mean_CSI = mean(CSI_Score), SE_CSI = sd(CSI_Score) / sqrt(n()), .groups = "drop") %>% mutate(Lower_CI = Mean_CSI - 1.96 * SE_CSI, Upper_CI = Mean_CSI + 1.96 * SE_CSI)
csi_summary <- csi_summary %>% group_by(Region) %>% mutate(Prev_CSI = lag(Mean_CSI), CSI_Drop = ifelse(Mean_CSI - Prev_CSI < -5, "Significant Drop", NA))
p <- ggplot(csi_summary, aes(x = Year, y = Mean_CSI, color = Region)) +
  geom_line(size = 1) + geom_point(size = 2) + geom_ribbon(aes(ymin = Lower_CI, ymax = Upper_CI, fill = Region), alpha = 0.2, color = NA) +
  geom_text(aes(label = CSI_Drop), vjust = -1, size = 3, na.rm = TRUE) + facet_wrap(~Region, ncol = 4) +
  scale_x_continuous(breaks = 2021:2025) + scale_y_continuous(limits = c(50, 100), breaks = seq(50, 100, 10)) +
  labs(title = "CSI Trends by U.S. Region (2021-2025)", subtitle = "Mean CSI with 95% Confidence Intervals", x = "Year", y = "CSI Score", caption = "Significant drops (>5 points) annotated") +
  theme_minimal() + theme(legend.position = "bottom", axis.text.x = element_text(angle = 45, hjust = 1), strip.background = element_rect(fill = "grey90", color = "black"), strip.text = element_text(size = 10, face = "bold"))
print(p)
ggsave("csi_trends_advanced.png", width = 12, height = 8, dpi = 300)
```

### 3. CSI Globe Map
**Script**: `csi_globe_map.R`
```R
library(plotly)
library(dplyr)
setwd("C:/Users/Veteran")
data <- read.csv("csi_by_region_expanded.csv", fileEncoding = "latin1") %>% filter(Year == 2025)
city_data <- data.frame(City = c("New York", "Chicago", "Miami", "Atlanta", "Los Angeles"), Region = c("Northeast", "Midwest", "South", "Southeast", "West"), Population = c(8.1, 2.6, 0.4, 0.5, 3.8), Lat = c(40.7128, 41.8781, 25.7617, 33.7490, 34.0522), Lon = c(-74.0060, -87.6298, -80.1918, -84.3880, -118.2437)) %>% left_join(select(data, Region, CSI_Score, Sample_Size, Top_Complaint), by = "Region") %>% mutate(hover_text = paste("City:", City, "<br>Region:", Region, "<br>CSI Score:", CSI_Score, "<br>Population (M):", Population, "<br>Top Complaint:", Top_Complaint))
fig <- plot_ly(city_data, type = "scattergeo", mode = "markers", lat = ~Lat, lon = ~Lon, text = ~hover_text, hoverinfo = "text", marker = list(size = ~Population * 10, color = ~CSI_Score, colorscale = "Viridis", colorbar = list(title = "CSI Score"), line = list(width = 1, color = "black"))) %>%
  layout(title = "2025 CSI Scores: Major U.S. Cities on Globe", geo = list(scope = "world", projection = list(type = "orthographic"), showland = TRUE, landcolor = toRGB("gray85"), countrycolor = toRGB("gray95"), showcountries = TRUE, showocean = TRUE, oceancolor = toRGB("lightblue"), center = list(lon = -95, lat = 37), rotation = list(lon = -100, lat = 0, roll = 0)))
fig
htmlwidgets::saveWidget(fig, "csi_globe_2025.html")
```

### 4. CSI Terrain Map
**Script**: `csi_terrain_map.R`
```R
library(leaflet)
library(dplyr)
setwd("C:/Users/Veteran")
data <- read.csv("csi_by_region_expanded.csv", fileEncoding = "latin1") %>% filter(Year == 2025)
city_data <- data.frame(City = c("New York", "Chicago", "Miami", "Atlanta", "Los Angeles"), Region = c("Northeast", "Midwest", "South", "Southeast", "West"), Population = c(8.1, 2.6, 0.4, 0.5, 3.8), Lat = c(40.7128, 41.8781, 25.7617, 33.7490, 34.0522), Lon = c(-74.0060, -87.6298, -80.1918, -84.3880, -118.2437)) %>% left_join(select(data, Region, CSI_Score, Sample_Size, Top_Complaint), by = "Region") %>% mutate(popup_text = paste("<b>", City, "</b><br>", "Region:", Region, "<br>", "CSI Score:", CSI_Score, "<br>", "Population (M):", Population, "<br>", "Top Complaint:", Top_Complaint))
map <- leaflet(data = city_data) %>% addProviderTiles(providers$OpenStreetMap.Mapnik, options = providerTileOptions(noWrap = TRUE)) %>% setView(lng = -98.5795, lat = 39.8283, zoom = 4) %>% addCircles(lng = ~Lon, lat = ~Lat, radius = ~Population * 50000, color = ~case_when(CSI_Score >= 80 ~ "green", CSI_Score >= 70 ~ "yellow", TRUE ~ "red"), fillOpacity = 0.7, popup = ~popup_text, label = ~City) %>% addLegend(position = "bottomright", colors = c("green", "yellow", "red"), labels = c("High (80+)", "Medium (70-79)", "Low (<70)"), title = "CSI Score Range")
print(map)
htmlwidgets::saveWidget(map, "csi_terrain_map_2025.html", selfcontained = TRUE)
```

## Why It Matters

This project bridges raw service data with customer experience, offering actionable insights for BMW X-series stakeholders. The heatmap reveals costly, frequent rotor issues (e.g., X6’s $1,100–$1,600 warped rotors), guiding maintenance priorities. The CSI trends and maps highlight regional satisfaction disparities—Southeast’s 89 vs. Midwest’s 60 in 2025—crucial for dealership strategies and customer retention. For data enthusiasts, it’s a showcase of R’s power in transforming automotive data into interactive, GPS-like visuals, blending technical depth with real-world impact.

## Conclusion

`BMWXSeriesRotorAnalysis` delivers a robust toolkit for understanding rotor replacement patterns and customer sentiment. Whether you’re a mechanic optimizing repairs, a BMW exec targeting satisfaction, or an R coder exploring visualization, this repo has something for you. Contributions welcome—fork it, tweak it, or suggest new angles like adding X7 back or real-time CSI feeds!

---
Repo: [github.com/[YourUsername]/BMWXSeriesRotorAnalysis](https://github.com/[YourUsername]/BMWXSeriesRotorAnalysis)
```

---

### Notes
- **CSV Snippets**: I included partial data for brevity—replace with full datasets when uploading to GitHub (e.g., paste all rows from my earlier responses).
- **YourUsername**: Swap `[YourUsername]` with your actual GitHub handle.
- **Why It Matters**: Added before the conclusion, emphasizing practical and technical value.
- **Compact Code**: Minified R scripts for readability while keeping functionality—expand comments if you prefer verbosity.

---

### How to Use
1. **Create Repo**: On GitHub, make a new repo named `BMWXSeriesRotorAnalysis`.
2. **Upload Files**:
   - Add the three `.csv` files (full versions from earlier).
   - Add the four `.R` scripts as separate files.
   - Add output files (`.png` and `.html`) to a `/outputs` folder if desired.
3. **Add README**:
   - Paste this markdown into `README.md` in the repo root.
4. **Push**: Commit and push to GitHub.

This README’s ready to impress on GitHub—let me know if you want to tweak the tone, add screenshots, or adjust anything else! 🌟
