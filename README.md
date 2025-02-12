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

## Summary
This documentation provides a structured approach to cleaning and modeling airline delay data using Power Query. The cleaned dataset is prepared for advanced analytics and visualization in Power BI, enabling stakeholders to derive actionable insights on flight delays and cancellations.


To use this schema for analyzing airline delays:

Data Ingestion: Populate the dimension tables with relevant data (e.g., airport details, carrier information, date details).

Fact Table Population: Load flight data into the Fact Flights table, ensuring proper linkage to the dimension tables via foreign keys.

Analysis: Perform queries and analysis on the fact table to derive insights into flight delays, cancellations, and other metrics.
