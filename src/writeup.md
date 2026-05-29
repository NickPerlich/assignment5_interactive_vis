---
title: Write-Up
---

# Data Centers and County Income: Write-Up

link to project repository: https://github.com/NickPerlich/assignment5_interactive_vis

## Design Rationale

### Visual Encodings

I chose a bar chart to compare average income per capita between counties with and without data centers. A bar chart made the most sense for this comparison because my main goal is to show the difference between two groups at a point in time, and bar height is an effective way to communicate that difference clearly. I used two distinctly different colors to distinguish the two groups. I used blue for counties with data centers and orange for counties without. I placed labels above the bars showing the exact dollar value so users could see the exact difference without having to trace from the y-axis. The y-axis is fixed across all years so that changes in bar height directly show real income changes to make sure the reader is not confused by axis scale changes per year. For projected years (2025–2030), the opacity of the bar colors is decreased to visually display that the data is estimated for those years. The year label also has "(Projected)" at the end to make the distinction more explicit.

### Interaction Techniques

I used a slider for the change in year. A slider gives the viewer direct control over how quicly or slowly they explore how the data changes through the years. I added tooltips on hover to give the viewer more information if they want to understand more about a specific year. I included number of counties, top county name, and top county income. This way, the viewer can receive more information if they want it without that information cluttering the main concept I want them to take away from my visual.

### Alternatives Considered

I considered a line chart for showing income trends over time, but it would have made the per year difference less explicit. That also would have been harder to make into an interactive visual. I also considered having the bar colors be on a continuous scale with the range being all values across all years, but I felt that it would take away from the one year at a time comparison I ended up focusing on.

## References

Mongird, K., Thurber, T., Vernon, C., Burleyson, C., Akdemir, K. Z., & Rice, J. (2025). IM3 Open Source Data Center Atlas [Dataset]. Pacific Northwest National Laboratory. https://doi.org/10.57931/2550666


Epoch AI. (2025). Frontier Data Centers Hub [Dataset]. https://epoch.ai/data/data-centers


U.S. Bureau of Economic Analysis. (2024). County and MSA personal income summary: Per capita personal income, Table CAINC1 [Dataset]. https://apps.bea.gov/iTable


U.S. Bureau of Economic Analysis. (2026). County GDP summary, Table CAGDP1 [Dataset]. https://www.bea.gov/data/gdp/gdp-county-metro-and-other-areas