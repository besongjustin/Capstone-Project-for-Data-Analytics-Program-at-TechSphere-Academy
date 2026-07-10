# Project Documentation
## Logistics Operation Analysis Dashboard
**Capstone Project — Data Analytics Program at TechSphere Academy**
**Author: Justin Besong (Jay)**

---

## Table of Contents
1. [Project Background](#1-project-background)
2. [Data Model](#2-data-model)
3. [Data Cleaning & Transformation](#3-data-cleaning--transformation)
4. [DAX Measures](#4-dax-measures)
5. [Dashboard Design](#5-dashboard-design)
6. [Key Insights & Findings](#6-key-insights--findings)

---

## 1. Project Background

This project was completed as the capstone submission for the Data Analytics Program at TechSphere Academy. The objective was to 
design and build a fully interactive Power BI dashboard that analyzes the logistics operations of a fictional company, translating 
raw data into business intelligence across four key operational domains: revenue performance, delivery efficiency, fleet and cost 
management, and driver safety.

The dataset comprised 14 relational tables covering customers, trips, drivers, trucks, routes, fuel, maintenance, and safety 
incidents — spanning operations from January 2022 to December 2024.

---

## 2. Data Model

The data model consists of 14 tables connected through a series of relationships. Below is a summary of each table and its role in the model.

| Table | Key Column(s) | Role |
|---|---|---|
| Customers | Customer ID | Dimension — customer profiles and contract types |
| Delivery Events | Trip ID, Facility ID | Fact — delivery records with on-time flags |
| Driver Monthly Metrics | Driver ID, Month | Fact — monthly aggregated driver metrics |
| Drivers | Driver ID | Dimension — driver profiles and employment status |
| Facilities | Facility ID | Dimension — facility names and locations |
| Fuel Purchase | Trip ID | Fact — fuel cost records per trip |
| Loads | Load ID, Customer ID, Route ID | Fact — load details and revenue |
| Maintenance Records | Truck ID, Date | Fact — maintenance history and labour costs |
| Routes | Route ID | Dimension — origin and destination city pairs |
| Safety Incidence | Driver ID, Trip ID, Truck ID | Fact — safety incident records |
| Trailers | Trailer ID | Dimension — trailer specifications |
| Trips | Trip ID, Driver ID, Truck ID, Load ID, Trailer ID | Central Fact — trip records linking all entities |
| Truck Utilization Metrics | Truck ID, Month | Fact — utilization rates and maintenance costs |
| Trucks | Truck ID | Dimension — truck specifications and status |
| Date Table | Date | Date dimension — created in Power BI for time intelligence |

### Relationship Summary
- **Date Table → TRIPS** (via Dispatch Date) — One to Many
- **Date Table → MAINTENANCE RECORDS** (via Date) — One to Many
- **Date Table → SAFETY INCIDENCE** (via Incident Date) — One to Many
- **TRIPS → DELIVERY EVENTS** (via Trip ID) — One to Many
- **TRIPS → FUEL PURCHASE** (via Trip ID) — One to Many
- **TRIPS → LOADS** (via Load ID) — One to Many
- **DRIVERS → TRIPS** (via Driver ID) — One to Many
- **TRUCKS → TRIPS** (via Truck ID) — One to Many
- **ROUTES → LOADS** (via Route ID) — One to Many
- **CUSTOMERS → LOADS** (via Customer ID) — One to Many
- **FACILITIES → DELIVERY EVENTS** (via Facility ID) — One to Many

---

## 3. Data Cleaning & Transformation

Data cleaning was performed in **Power Query** before loading into the data model. Key steps included:

- Removed null and blank values from key columns across fact tables
- Corrected data types — date columns, numeric columns, and boolean flags were properly typed
- Fixed a typo in the Safety Incidence table — "Incident Tyype" was corrected to "Incident Type"
- Created a calculated **Date Table** using `CALENDARAUTO()` in DAX to enable time intelligence
- Added supporting columns to the Date Table: Month Name, Month Number, and Year

---

## 4. DAX Measures

All DAX measures were created as explicit measures and organized by report page.

### Executive Overview

```dax
Gross Total Revenue = 
SUMX(TRIPS, RELATED(LOADS[Revenue]))

Net Profit = 
SUMX(TRIPS, RELATED(LOADS[Revenue])) 
- SUM('FUEL PURCHASE'[Total Cost]) 
- SUM('MAINTENANCE RECORDS'[Total Labour Cost])

% On-Time Delivery = 
DIVIDE(
    CALCULATE(
        COUNTROWS('DELIVERY EVENTS'),
        'DELIVERY EVENTS'[On Time Flag] = TRUE()
    ),
    COUNTROWS('DELIVERY EVENTS')
)

Revenue per Mile = 
DIVIDE(
    SUM(LOADS[Revenue]),
    SUM(TRIPS[Distance in Miles])
)

Total Trips = COUNTROWS(TRIPS)

Total Customers = DISTINCTCOUNT(CUSTOMERS[Customer ID])

Py Revenue = 
CALCULATE(
    [Gross Total Revenue], 
    SAMEPERIODLASTYEAR('Date Table'[Date])
)

Py Profit = 
CALCULATE(
    [Net Profit], 
    SAMEPERIODLASTYEAR('Date Table'[Date])
)
```

### Delivery Performance

```dax
Total Deliveries = COUNTROWS('DELIVERY EVENTS')

On-Time Deliveries = 
CALCULATE(
    COUNTROWS('DELIVERY EVENTS'),
    'DELIVERY EVENTS'[On Time Flag] = TRUE()
)

Late Deliveries = 
CALCULATE(
    COUNTROWS('DELIVERY EVENTS'),
    'DELIVERY EVENTS'[On Time Flag] = FALSE()
)

Total Routes = DISTINCTCOUNT(ROUTES[Route ID])

Total Loads = COUNTROWS(LOADS)
```

### Fleet & Cost

```dax
Total Maintenance Cost = 
SUM('MAINTENANCE RECORDS'[Total Labour Cost])

Total Fuel Cost = SUM('FUEL PURCHASE'[Total Cost])

Avg Utilization Rate = AVERAGE('TRUCK UTILIZATION METRICS'[Utilization Rate])

Maintenance Frequency = COUNTROWS('MAINTENANCE RECORDS')

Average MPG = AVERAGE(TRIPS[Average MPG])
```

### Safety & Drivers

```dax
Total Safety Incidents = COUNTROWS('SAFETY INCIDENCE')

Total Incidents by Driver = DISTINCTCOUNT('SAFETY INCIDENCE'[Driver ID])

Total Injured Incidents = 
CALCULATE(
    COUNTROWS('SAFETY INCIDENCE'),
    'SAFETY INCIDENCE'[Injury Flag] = TRUE()
)

Total Vehicle Damage Cost = 
SUM('SAFETY INCIDENCE'[Vehicle damage Cost])

Total Drivers = DISTINCTCOUNT(DRIVERS[Driver ID])

Driver Trips Completed = SUM('DRIVER MONTHLY METRICS'[Trips Completed])
```

---

## 5. Dashboard Design

The dashboard was designed with a dark green and white colour theme, maintaining a clean and professional aesthetic throughout all 4 pages.

### Page 1 — Executive Overview
Provides a high-level snapshot of business performance for executive stakeholders. Features KPI cards for the 6 headline metrics, a year-over-year 
revenue trend, profit breakdown by product type, top 10 customers by revenue, customer contract type distribution, and revenue by route.

**Slicers:** Customer Name, Date

### Page 2 — Delivery Performance
Focuses on delivery efficiency across cities, facilities, and product types. Includes a combo chart showing deliveries vs revenue by city, 
on-time vs late delivery breakdown, deliveries by product type, facility-level on-time performance, and monthly trip volume trend.

**Slicers:** Event Type, Destination City

### Page 3 — Fleet & Cost
Covers asset utilization and cost management. Features maintenance cost trend over time, top trucks by maintenance cost, fuel cost by driver, 
average utilization rate by truck, and a safety incidents summary.

**Slicers:** Maintenance Type, Truck Status

### Page 4 — Safety & Drivers
Examines driver performance and safety risk. Includes safety incidents by driver, monthly incident trend, top and bottom 5 drivers by trips completed, 
and incidents broken down by type.

**Slicers:** City, Driver Name

---

## 6. Key Insights & Findings

### Revenue & Profitability
- Total revenue of **$262.5M** with a net profit of **$161.2M**, representing a 61% profit margin
- Monthly revenue remained stable between $6.57M and $7.58M across the 3-year period
- **Automotive** is the most profitable product type at $29.3M, followed by Food/Beverage at $25M
- **First Group** is the top customer at $9.1M — 42% ahead of the second-ranked customer
- **Route RTE00016** generates the highest revenue at $10.1M

### Delivery Performance
- Overall on-time delivery rate of **55.67%** — nearly 1 in 2 deliveries is late
- **28,417 late deliveries** out of 85K total, representing 33.27% of all deliveries
- **Kansas City** records the highest delivery volume
- Trip volume peaks in **September at 9.2K** and drops significantly in January, indicating strong seasonality
- **Dallas Distribution Center** recorded the lowest on-time rate among all facilities

### Fleet & Cost
- Fleet fuel costs totaled **$95.59M** — significantly outweighing maintenance costs of $5.73M
- Average fleet fuel efficiency stands at **6.50 MPG** with an average utilization rate of **0.83**
- **TRK00003** is the most expensive truck to maintain at $90K — 16% above the next highest
- Maintenance costs fluctuate between $118K and $205K monthly, suggesting inconsistent preventive maintenance scheduling
- Utilization rates are balanced across the fleet, ranging narrowly between 9.87% and 10.15%

### Safety & Drivers
- **170 safety incidents** recorded across 3 years, resulting in $1.60M in vehicle damage and 33 injuries
- **DOT Violations** are the leading incident type at 22.94%, followed by Moving Violations and Equipment Damage at 20.59% each
- **David Miller** leads in individual incidents with 7 recorded events
- Top driver **William Wilson** completed 1,429 trips — more than double the bottom-performing driver **John Smith** at 623, highlighting a significant performance gap

---

*Documentation prepared by Justin Besong (Jay)*
*LinkedIn: [linkedin.com/in/besong-justin](https://linkedin.com/in/besong-justin)*
*X: [@Justin_analyst](https://twitter.com/Justin_analyst)*
