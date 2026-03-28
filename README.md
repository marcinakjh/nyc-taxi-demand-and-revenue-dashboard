NYC TLC Yellow Taxi Analysis

SQL + Tableau End-to-End Project
An end-to-end data analysis project built on the NYC Taxi & Limousine Commission (TLC) Yellow Taxi dataset. Raw trip records (3.4M+ rows) were queried and pre-aggregated in PostgreSQL, then visualized in an interactive Tableau Public dashboard surfacing demand trends, fare variance, tip behavior, and corridor traffic patterns.

https://public.tableau.com/app/profile/john.henry.marcinak/vizzes

Project Structure
nyc-tlc-taxi-analysis/
│
├── sql/
│   ├── query1_tip_analysis.sql
│   ├── query2_fare_percentiles.sql
│   ├── query3_rolling_7day_trip_trend.sql
│   └── query4_top_corridors.sql
│
├── data/
│   ├── tip_analysis.csv
│   ├── fare_percentiles.csv
│   ├── rolling_7day_trip_trend.csv
│   └── top_corridors.csv
│
└── README.md

Data Source

Dataset: NYC TLC Yellow Taxi Trip Records — January 2025
Source: NYC Taxi & Limousine Commission
Volume: 3.4M+ trip records
Database: PostgreSQL (table: trips)
Zone Reference: TLC Taxi Zone Lookup (taxi_zone_lookup.csv)


Workflow
NYC TLC Raw Data (3.4M+ rows)
        ↓
PostgreSQL — 4 pre-aggregation queries
        ↓
CSV exports (aggregated, dashboard-ready)
        ↓
Tableau Public — 4-sheet interactive dashboard
Pre-aggregating in SQL before connecting to Tableau keeps the dashboard fast and publication-ready, avoiding the performance cost of connecting Tableau directly to a multi-million row table.

SQL Queries
Query 1 — Tip Analysis by Hour & Payment Type
File: sql/query1_tip_analysis.sql
Calculates average tip amount and tip percentage broken down by hour of day and payment method (Card vs. Cash). Uses NULLIF to safely handle zero fare amounts and a CASE statement to convert raw payment type codes into readable labels.
Key technique: NULLIF(fare_amount, 0) prevents division-by-zero errors when calculating tip percentage.

Query 2 — Fare Percentiles by Pickup Zone
File: sql/query2_fare_percentiles.sql
Computes median fare, 90th percentile fare, and interquartile range (IQR) for each pickup zone. Filters to fares between $2.50 and $200 to exclude data entry errors, and limits to zones with more than 500 trips for statistical reliability.
Key technique: PERCENTILE_CONT() ordered-set aggregate function to compute median and IQR directly in SQL.

Query 3 — Rolling 7-Day Trip & Revenue Trend
File: sql/query3_rolling_7day_trip_trend.sql
Builds a daily summary of trip volume and revenue, then applies a 7-day rolling average using a window function. The rolling average smooths out day-of-week fluctuations to reveal the underlying demand trend across January 2025.
Key technique: AVG() OVER (ORDER BY pickup_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) window function for rolling aggregation.

Query 4 — Top Corridors by Day & Hour
File: sql/query4_top_corridors.sql
Identifies the top 5 busiest pickup-to-dropoff zone corridors for every combination of day of week and hour of day. Excludes same-zone trips and uses RANK() with a window partition to surface the highest-traffic routes at each time slot.
Key technique: RANK() OVER (PARTITION BY day_of_week, pickup_hour ORDER BY trip_count DESC) to rank corridors within each time window.

Dashboard
The Tableau dashboard is organized into four sheets:
SheetData SourceChart TypeQuestion AnsweredDemand & Revenue Trendrolling_7day_trip_trendDual-axis bar + lineHow did daily trip volume and revenue evolve through January?Fare Variance by Zonefare_percentilesSorted horizontal barWhich pickup zones have the most unpredictable fares?Tip Behavior by Hourtip_analysisDual-axis bar + lineHow do tip amounts and percentages vary by time of day and payment type?Corridor Traffic Heatmaptop_corridorsDay × Hour heatmapWhen and where is corridor demand highest across the week?

Key Findings

Demand peaked on January 16 with 138,711 trips and $3.6M in daily revenue — the highest single day in the dataset
New Year's Day (Jan 1) saw a sharp drop in trips, recovering steadily through the following weeks
Queens zones near JFK (South Ozone Park, Baisley Park, Kew Gardens) show the widest fare variance — trips from these zones sometimes go to JFK (long, expensive) and sometimes stay local (short, cheap), producing high IQR values
JFK Airport itself has moderate fare variance ($23.60 IQR) because most pickups there go to Manhattan at a consistent rate
Card tips hold steady at ~26% across all 24 hours; cash tips record near $0, reflecting a structural gap in the TLC data where cash gratuities go uncaptured
Friday and Saturday evenings (6–11pm) are the busiest time windows across all corridors
Upper East Side (zones 236 ↔ 237) is the single busiest corridor in the dataset


Tools Used

PostgreSQL — data storage and pre-aggregation
SQL — window functions, ordered-set aggregates, CTEs
Tableau Public — interactive dashboard and publication
NYC TLC Open Data — primary data source


Notes
This project was completed as a learning exercise to practice end-to-end data analysis workflows — from raw data ingestion and SQL aggregation through to interactive visualization and publishing.
