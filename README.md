# NYC 311 Service Requests

**Author:** Richa Shiny Tigiripally  
**Date:** April 2, 2026  
**Tools:** R, Quarto, tidyverse, httr, leaflet, ggplot2


## Overview

This assessment analyzes NYC 311 service requests filed in Manhattan during 
Q1 2024 (January through March), joined to a community district reference 
table to identify neighborhoods with the greatest need for HPD heating and 
hot water tenant outreach.


## My Thought Process

### Data Collection
Rather than manually downloading a CSV from NYC Open Data, I pulled the 311 
data programmatically using the Socrata API. This makes the analysis fully 
reproducible since anyone running the code will get the same dataset without 
needing a local file. I applied three filters at the API level: borough = 
MANHATTAN, created_date within Q1 2024, and a row limit of 200,000 to 
override the default 1,000-row cap.

The community district reference table was loaded directly from this GitHub 
repository using its raw URL, again avoiding the need for a local file.

### Data Preparation and the Join Key
The first thing I did before joining was actually look at what the 
community_board field looked like in the 311 data. I expected a clean integer 
like 1 or 12, but found values formatted as "12 MANHATTAN", "07 MANHATTAN", 
etc. with the borough name appended. This meant a direct join would have 
failed entirely.

I wrote a regex to extract the leading digits from each value, cast them to 
integers, and validated that results fell within the 1-12 range of Manhattan 
community districts. I renamed the join key to cd_numeric on both sides to 
make the cleaning step explicit and intentional.

### The Join
I used a left join to preserve all 311 records, including those that could 
not be matched to a community district. An inner join would have silently 
dropped ungeocoded records, which felt like the wrong call since those 
records still represent real service requests. I wanted to be able to 
diagnose them rather than lose them.

### Diagnosing Non-Matches
3,898 records (2.3% of the dataset) did not match. I traced these to two 
root causes: records coded as "Unspecified MANHATTAN" that were never 
geocoded to a specific district, and records coded as "64 MANHATTAN" which 
is not a valid Manhattan community board. I chose not to impute a district 
from address fields since the risk of misassignment outweighed the benefit 
for such a small share of records.

### The Recommendation
I filtered for complaint types containing "HEAT" or "HOT WATER" and 
aggregated by planning area and community board. Raw complaint counts alone 
would have favored larger districts simply because they have more housing 
units, so I normalized by residential units to get a fair comparison across 
districts of different sizes.

Upper Manhattan dominated on both measures. Within Upper Manhattan, Community 
Board 12 (Washington Heights/Inwood/Marble Hill) had the highest rate at 87.5 
complaints per 1,000 residential units, and Community Board 10 (Central 
Harlem) was second at 70.3. Both have large shares of older rent-stabilized 
housing where tenants are most dependent on landlord-provided heat.

---

## Files in this Repo

| File | Description |
|------|-------------|
| `Assessment_backup.qmd` | Full analysis code with written responses inline |
| `Assessment_backup.html` | Rendered HTML output |
| `manhattan_community_district_reference.csv` | Community district reference data |
| `Assessment_files/` | Chart image assets |
| `images/` | Header image |

## Data Note

The 311 dataset (165,894 rows) is pulled live from the NYC Open Data API 
and is not stored in this repository due to file size. The raw CSV 
(311_manhattan_q1_2024.csv) is available in box upon request or can be reproduced 
by running the .qmd file.


## Policy Recommendation Summary

Based on Q1 2024 311 data, I recommend HPD direct the tenant outreach and 
education grant to **Upper Manhattan**, specifically targeting **Community 
Board 12 (Washington Heights / Inwood / Marble Hill)** and **Community Board 
10 (Central Harlem)**.

These two community boards show the highest rates of heating and hot water 
complaints per 1,000 residential units in Manhattan, at 87.5 and 70.3 
respectively. This concentration is consistent across all three months of Q1 
2024, suggesting a structural problem with heating maintenance rather than a 
weather-driven spike.

Both neighborhoods have large shares of older rent-stabilized and low-income 
housing where tenants are most dependent on landlord-provided heat and least 
likely to know their legal rights around heating obligations. Targeted tenant 
education in these areas has the highest potential impact per dollar spent.

**Key caveats for HPD to keep in mind:**

311 data measures who reports problems, not who has them. Communities with 
lower 311 engagement due to language barriers, immigration status concerns, 
or distrust of city systems may be underrepresented even if their housing 
conditions are equally poor. Any outreach strategy should account for this 
by prioritizing multilingual materials and community-based organizations with 
existing trust in these neighborhoods.

Additionally, this analysis reflects a single quarter. HPD should treat these 
findings as a starting point for deeper engagement with local Community Boards 
and housing advocates rather than a definitive ranking of need.
