# Airline-Delay
# Cleaning and Modeling Airline Delay Data Using Power Query

## Introduction
This documentation outlines the process of cleaning and transforming airline delay data using Power Query. The dataset contains 30 columns, each providing specific information about flight records. The goal is to clean the data, handle missing values, transform data types, and prepare the dataset for further analysis or modeling.

## Dataset Overview
The dataset includes the following columns:

- **Flight Information:** Unique ID, flight number, aircraft tail number, and carrier code.
- **Date & Time:** Year, month, day, scheduled and actual departure/arrival times.
- **Delays & Duration:** Scheduled and actual elapsed time, air time, departure and arrival delays.
- **Airport Data:** Origin and destination airport codes, flight distance.
- **Taxi Time:** Taxi-in and taxi-out durations.
- **Flight Status:** Cancellation status, diversion status, cancellation reason.
- **Delay Causes:** Carrier, weather, National Airspace System (NAS), security, and late aircraft delays.

## Data Cleaning Process

### Step 1: Handling Missing Values
**Affected Columns:** `CarrierDelay`, `WeatherDelay`, `NASDelay`, `SecurityDelay`, `LateAircraftDelay`
- Replace all null values with `0` to ensure consistency in calculations.
- **Power Query Step:**
  ```m
  ReplaceNullValues = Table.ReplaceValue(Source, null, 0, Replacer.ReplaceValue, {"CarrierDelay", "WeatherDelay", "NASDelay", "SecurityDelay", "LateAircraftDelay"})
  ```

### Step 2: Transforming Data Types
**Affected Columns:** `Cancelled`, `Diverted`
- Convert numeric values (0,1) to Boolean (`True/False`) for better readability.
- **Power Query Step:**
  ```m
  ChangeType = Table.TransformColumnTypes(ReplaceNullValues, {{"Cancelled", type logical}, {"Diverted", type logical}})
  ```

### Step 3: Creating a Date Column
**Affected Columns:** `Year`, `Month`, `DayofMonth`
- Merge these into a `Date` column in `yyyy/MM/dd` format.
- Generate `DateKey` in `YYYYMMDD` format for linking with a date dimension.
- **Power Query Step:**
  ```m
  AddDateColumn = Table.AddColumn(ChangeType, "Date", each Date.FromText(Text.From([Year]) & "/" & Text.From([Month]) & "/" & Text.From([DayofMonth])), type date)
  AddDateKey = Table.AddColumn(AddDateColumn, "DateKey", each Date.ToText([Date], "YYYYMMDD"), type text)
  ```

### Step 4: Formatting Time Columns
**Affected Columns:** `DepTime`, `CRSDepTime`, `ArrTime`, `CRSArrTime`
- Convert numeric values to proper `hh:mm` time format.
- **Power Query Step:**
  ```m
  ConvertTime = Table.TransformColumns(AddDateKey, {
      {"DepTime", each Time.FromText(Text.PadStart(Text.From(_), 4, "0")), type time},
      {"CRSDepTime", each Time.FromText(Text.PadStart(Text.From(_), 4, "0")), type time},
      {"ArrTime", each Time.FromText(Text.PadStart(Text.From(_), 4, "0")), type time},
      {"CRSArrTime", each Time.FromText(Text.PadStart(Text.From(_), 4, "0")), type time}
  })
  ```

### Step 5: Removing Redundant Columns
**Affected Columns:** `Year`, `Month`, `DayofMonth`, `DayOfWeek`
- These columns become redundant after creating `Date` and `DateKey`.
- **Power Query Step:**
  ```m
  RemoveColumns = Table.RemoveColumns(ConvertTime, {"Year", "Month", "DayofMonth", "DayOfWeek"})
  ```

### Step 6: Removing Duplicates in Airport Table
- Ensure unique entries for each airport based on `IATA` and `ICAO` codes.

## Data Modeling
 ![Image Alt](https://github.com/Mohammed1999sstack/Airline-Delay/blob/main/ScreenShots/DATA%20Moddling.png)
### Objective
We aim to model airline delay data to:
- Analyze causes and patterns of flight delays.
- Understand reasons for flight cancellations.
- Identify trends and correlations between carriers, airports, and dates.
- Provide actionable insights for improving operational efficiency.

### Fact Table: `Fact Flights`
- `ActualElapsedTime`, `AirTime`, `AirDelay`, `CancellationCode`, `Cancelled`
- `CarrierDelay`, `CRSArr_Time`, `CRSDep_Time`, `CRSElapsedTime`
- `DateKey`, `DepDelay`, `DepTime`, `Dest`, `Distance`
- `Diverted`, `FlightID`, `FlightNum`, `LateAircraftDelay`
- `NASDelay`, `SecurityDelay`, `TaxiIn`, `TaxiOut`, `WeatherDelay`

### Dimension Tables
#### `Dim Airport`
- `IATA`, `ICAO`, `Airport Name`, `Country Code`, `Latitude`, `Longitude`, `Region`

#### `Dim Carrier`
- `Carrier Name`, `IATA Carrier Code`, `ICAO Carrier Code`

#### `Dim Cancellation`
- `Cancellation Code`, `Cancellation Reason`

#### `Dim Date`
- `Date`, `DateKey`, `Day`, `Day Name`, `Month`, `Month Name`, `Year`
- **DAX Code to Generate Date Table:**
  ```DAX
  Dim Date =
  VAR minDate = MIN('Fact Flights'[DateKey])
  VAR maxDate = MAX('Fact Flights'[DateKey])
  VAR startDate = DATE(LEFT(minDate, 4), 1, 1)
  VAR endDate = DATE(LEFT(maxDate, 4), 12, 31)
  RETURN
      ADDCOLUMNS(
          CALENDAR(startDate, endDate),
          "Year", YEAR([Date]),
          "Month No.", MONTH([Date]),
          "Day", DAY([Date]),
          "Month Name", FORMAT([Date], "MMMM"),
          "Day Name", FORMAT([Date], "DDD"),
          "DateKey", FORMAT([Date], "yyyyMMdd")
      )
  ```

## Table Relationships
- **Fact Flights ↔ Dim Date:** `DateKey`
- **Fact Flights ↔ Dim Airport:** `Dest = IATA`
- **Fact Flights ↔ Dim Carrier:** `Carrier Code`
- **Fact Flights ↔ Dim Cancellation:** `CancellationCode`

# Visualization in Power BI

## Overview
I used **Power BI Desktop** to create visualizations for this project. The report consists of multiple pages, each serving a specific purpose.

## Home Page
 ![Image Alt](https://github.com/Mohammed1999sstack/Airline-Delay/blob/main/ScreenShots/Home%20page.png)
The first visual, located in the **upper left corner**, is the **Home Page**. This page contains:
- The **Project Title**
- **Navigation Buttons** to:
  - View the Dashboard
  - Access the Project Description
  - Open the Q&A section
  
This page serves as the entry point for users to navigate through the report easily.

## First Page: Overview
 ![Image Alt](https://github.com/Mohammed1999sstack/Airline-Delay/blob/main/ScreenShots/overview%20page.png)
The first page, provides an **overview** of key metrics and insights. It includes:
- Total flights, offline flights, early flights, and late flights
- Top 10 carriers by flights
- Top 10 airports by flights
- Flight trends by month and quarter
- Regional flight comparison

This page summarizes flight performance across different airlines and airports.

## second Page: Geographical Analysis
 ![Image Alt](https://github.com/Mohammed1999sstack/Airline-Delay/blob/main/ScreenShots/Geographical%20page.png)
The second page, focuses on **regional analysis**. It helps in understanding flight delays across different regions. The key elements include:
- Total delay hours by region (displayed on a map)
- Top 5 and bottom 5 regions by delays
- Total plane and delay ratio by carrier
- Number of airports and delay ratio by region

This page enables users to identify delay patterns across different geographical locations.

## Drill-Through to Region Details
 ![Image Alt](https://github.com/Mohammed1999sstack/Airline-Delay/blob/main/ScreenShots/Drill%20throught%20region.png)
The dashboard also supports **drill-through functionality**, allowing users to select a specific region from the geographical analysis page and explore detailed statistics. The drill-through page includes:
- Average departure and arrival delay
- Delay breakdown by reason (weather, security, aircraft, carrier)
- Monthly delay trends
- Delay ratio by time of day and hour
- Drill-through capability to analyze specific airports within a region

This page allows users to analyze flight delays at a granular level, identifying key causes and trends at specific airports.

## Drill-Through to Airport Details
 ![Image Alt](https://github.com/Mohammed1999sstack/Airline-Delay/blob/main/ScreenShots/Drill%20throught%20Airport.png)
The dashboard also supports **drill-through functionality**, allowing users to select a specific airport from the geographical analysis page and explore detailed statistics. The drill-through page includes:
- **Airport-Specific Metrics**: Number of planned flights, average departure and arrival delays, and delays by different reasons.
- **Total Delay by Airport**: Visualized on an interactive map.
- **Delay Patterns**:
  - Delay ratio by month to observe seasonal trends.
  - Delay ratio by day to identify weekly patterns.
  - Delay ratio by hour to analyze peak delay times.
- **Detailed Flight Data**: Includes aircraft tail numbers, total distance covered, number of flights, and total delay for each aircraft.

This drill-through functionality enhances the dashboard by providing deeper insights into delay trends at an airport level, making it easier for users to diagnose issues and optimize operations.



This README serves as a provisional guide for the **GitHub repository**, outlining the purpose and structure of the dashboard visuals.




To use this schema for analyzing airline delays:

Data Ingestion: Populate the dimension tables with relevant data (e.g., airport details, carrier information, date details).

Fact Table Population: Load flight data into the Fact Flights table, ensuring proper linkage to the dimension tables via foreign keys.

Analysis: Perform queries and analysis on the fact table to derive insights into flight delays, cancellations, and other metrics.
