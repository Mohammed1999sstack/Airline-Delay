# Airline-Delay: Cleaning and Modeling Airline Delay Data Using Power Query

## Introduction
This repository contains the documentation, code, and resources for cleaning, transforming, and analyzing airline delay data using **Power Query** and **Power BI**. The dataset includes 30 columns with detailed flight information, and the goal is to clean the data, handle missing values, transform data types, and prepare it for analysis and visualization. The final output is an interactive **Power BI dashboard** that provides actionable insights into flight delays, cancellations, and operational efficiency.

---

## Dataset Overview
The dataset contains the following key columns:

### Flight Information
- **FlightID**: Unique identifier for each flight.
- **FlightNum**: Flight number.
- **TailNum**: Aircraft tail number.
- **CarrierCode**: Carrier code.

### Date & Time
- **Year**, **Month**, **DayofMonth**: Date components.
- **CRSDepTime**, **DepTime**: Scheduled and actual departure times.
- **CRSArrTime**, **ArrTime**: Scheduled and actual arrival times.

### Delays & Duration
- **CRSElapsedTime**, **ActualElapsedTime**: Scheduled and actual flight duration.
- **DepDelay**, **ArrDelay**: Departure and arrival delays.
- **CarrierDelay**, **WeatherDelay**, **NASDelay**, **SecurityDelay**, **LateAircraftDelay**: Delay causes.

### Airport Data
- **Origin**, **Dest**: Origin and destination airport codes.
- **Distance**: Flight distance in miles.

### Taxi Time
- **TaxiIn**, **TaxiOut**: Taxi-in and taxi-out durations.

### Flight Status
- **Cancelled**, **Diverted**: Binary indicators for cancellations and diversions.
- **CancellationCode**: Reason for cancellation.

---

## Data Cleaning Process

### Step 1: Handling Missing Values
- **Affected Columns:** `CarrierDelay`, `WeatherDelay`, `NASDelay`, `SecurityDelay`, `LateAircraftDelay`
- Replace null values with `0` to ensure consistency.
- **Power Query Step:**
  ```m
  ReplaceNullValues = Table.ReplaceValue(Source, null, 0, Replacer.ReplaceValue, {"CarrierDelay", "WeatherDelay", "NASDelay", "SecurityDelay", "LateAircraftDelay"})
  ```

### Step 2: Transforming Data Types
- **Affected Columns:** `Cancelled`, `Diverted`
- Convert numeric values (0,1) to Boolean (`True/False`) for better readability.
- **Power Query Step:**
  ```m
  ChangeType = Table.TransformColumnTypes(ReplaceNullValues, {{"Cancelled", type logical}, {"Diverted", type logical}})
  ```

### Step 3: Creating a Date Column
- **Affected Columns:** `Year`, `Month`, `DayofMonth`
- Merge these into a `Date` column in `yyyy/MM/dd` format.
- Generate `DateKey` in `YYYYMMDD` format for linking with a date dimension.
- **Power Query Step:**
  ```m
  AddDateColumn = Table.AddColumn(ChangeType, "Date", each Date.FromText(Text.From([Year]) & "/" & Text.From([Month]) & "/" & Text.From([DayofMonth])), type date)
  AddDateKey = Table.AddColumn(AddDateColumn, "DateKey", each Date.ToText([Date], "YYYYMMDD"), type text)
  ```

### Step 4: Formatting Time Columns
- **Affected Columns:** `DepTime`, `CRSDepTime`, `ArrTime`, `CRSArrTime`
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
- **Affected Columns:** `Year`, `Month`, `DayofMonth`, `DayOfWeek`
- These columns become redundant after creating `Date` and `DateKey`.
- **Power Query Step:**
  ```m
  RemoveColumns = Table.RemoveColumns(ConvertTime, {"Year", "Month", "DayofMonth", "DayOfWeek"})
  ```

### Step 6: Removing Duplicates in Airport Table
- Ensure unique entries for each airport based on `IATA` and `ICAO` codes.

---

## Data Modeling
   ![Home Page](https://github.com/Mohammed1999sstack/Airline-Delay/blob/main/ScreenShots/DATA%20Moddling.png)
### Objective
The data model is designed to:
- Analyze causes and patterns of flight delays.
- Understand reasons for flight cancellations.
- Identify trends and correlations between carriers, airports, and dates.
- Provide actionable insights for improving operational efficiency.

### Fact Table: `Fact Flights`
- Contains detailed flight data, including delays, cancellations, and durations.
- Key columns: `FlightID`, `DateKey`, `CarrierCode`, `Dest`, `DepDelay`, `ArrDelay`, `Cancelled`, `Diverted`, etc.

### Dimension Tables
1. **Dim Airport**: Contains airport details (`IATA`, `ICAO`, `Airport Name`, `Country Code`, `Latitude`, `Longitude`, `Region`).
2. **Dim Carrier**: Contains carrier details (`Carrier Name`, `IATA Carrier Code`, `ICAO Carrier Code`).
3. **Dim Cancellation**: Contains cancellation reasons (`Cancellation Code`, `Cancellation Reason`).
4. **Dim Date**: Contains date details (`Date`, `DateKey`, `Day`, `Day Name`, `Month`, `Month Name`, `Year`).

### Table Relationships
- **Fact Flights ↔ Dim Date:** `DateKey`
- **Fact Flights ↔ Dim Airport:** `Dest = IATA`
- **Fact Flights ↔ Dim Carrier:** `Carrier Code`
- **Fact Flights ↔ Dim Cancellation:** `CancellationCode`

---

## Power BI Dashboard

### Overview
The **Power BI dashboard** provides an interactive and visually appealing way to explore flight delay data. It consists of multiple pages, each focusing on a specific aspect of the data.

### Pages
1. **Home Page**:
   ![Home Page](https://github.com/Mohammed1999sstack/Airline-Delay/blob/main/ScreenShots/Home%20page.png)
   - Project title and navigation buttons to access the dashboard, project description, and Q&A section.

2. **Overview Page**:
   ![Overview Page](https://github.com/Mohammed1999sstack/Airline-Delay/blob/main/ScreenShots/overview%20page.png)
   - Key metrics: Total flights, offline flights, early flights, and late flights.
   - Top 10 carriers and airports by flights.
   - Flight trends by month and quarter.
   - Regional flight comparison.

3. **Geographical Analysis Page**:
   ![Geographical Analysis Page](https://github.com/Mohammed1999sstack/Airline-Delay/blob/main/ScreenShots/Geographical%20page.png)
   - Total delay hours by region (displayed on a map).
   - Top 5 and bottom 5 regions by delays.
   - Delay ratio by carrier and region.

4. **Drill-Through Pages**:
   - **Region Details**:
     ![Region Details](https://github.com/Mohammed1999sstack/Airline-Delay/blob/main/ScreenShots/Drill%20throught%20region.png)
     - Average departure and arrival delays, delay breakdown by reason, monthly delay trends.
   - **Airport Details**:
     ![Airport Details](https://github.com/Mohammed1999sstack/Airline-Delay/blob/main/ScreenShots/Drill%20throught%20Airport.png)
     - Airport-specific metrics, delay patterns by month, day, and hour.

5. **Date & Time Analysis Page**:
   ![Date & Time Analysis Page](https://github.com/Mohammed1999sstack/Airline-Delay/blob/main/ScreenShots/Date%20%26%20Time%20page.png)
   - Delay trends by month, day of the week, and hour.
   - Breakdown of delay types (Carrier, NAS, Security, Weather, Late Aircraft).

6. **Cancellations & Diversions Analysis Page**:
   ![Cancellations & Diversions Analysis Page](https://github.com/Mohammed1999sstack/Airline-Delay/blob/main/ScreenShots/Cancel%20%26%20Drivted.png)
   - Key KPIs: Canceled flights, cancellation rate, diverted flights, diversion ratio.
   - Trend analysis of cancellations and diversions over time.
   - Cancellation reasons and trends by carrier.

7. **slicer**:
   ![Cancellations & Diversions Analysis Page](https://github.com/Mohammed1999sstack/Airline-Delay/blob/main/ScreenShots/Slicer_view.png)
The slicer allows users to filter flight data based on:  
- **Airport** – Select a specific airport or view all.  
- **Carrier** – Filter by airline carrier.  
- **Date** – Choose a date range.  
- **Hour** – Select specific hours of the day.  
To use it, simply adjust the dropdowns and sliders, then apply the filters to update the dashboard.  
---

## Features & Functionalities  
- **Interactive Visuals**: Users can filter and slice data to focus on specific categories.  
- **Slicer Filters**: Easily refine data by selecting specific airports, carriers, dates, and hours.  
- **Drill-Through Capabilities**: Analyze data at granular levels (e.g., specific regions or airports).  
- **Geographical Mapping**: Visualize regional impacts of flight delays.  
- **Time-Based Trend Analysis**: Identify delay patterns over time.  

---

## Technologies Used
- **Power Query**: For data cleaning and transformation.
- **Power BI Desktop**: For data modeling and visualization.
- **DAX (Data Analysis Expressions)**: For calculations and measures.

---

## How to Use the Dashboard
1. Open the **Power BI file (.pbix)**.
2. Navigate through different pages using the **sidebar menu**.
3. Use **filters and slicers** to explore specific insights.
4. Click on regions, carriers, or airports for **drill-through analysis**.

---

## Future Enhancements
- Integration with **real-time flight data APIs** for up-to-date insights.
- **Predictive analytics** for forecasting delays and cancellations.
- Additional **user customization options** for personalized dashboards.

---


