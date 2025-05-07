# **Airline-Delay: Comprehensive Analysis of Flight Delays Using Power BI**

## **Project Description**  
This project provides an in-depth analysis of airline delay data to identify operational inefficiencies, root causes of disruptions, and opportunities for improvement. Leveraging **Power Query** for data cleaning and **Power BI** for interactive visualizations, it delivers actionable insights for airlines, airports, and aviation stakeholders to enhance on-time performance and passenger satisfaction.  

---

## **Contents**  
1. [Business Case](#1-business-case)  
2. [Dataset Overview](#2-dataset-overview)  
3. [Tools and Technologies Used](#3-tools-and-technologies-used)  
4. [Data Cleaning and Transformation](#4-data-cleaning-and-transformation)  
5. [Data Modeling](#5-data-modeling)  
6. [Power BI Dashboard](#6-power-bi-dashboard)  
7. [How to Use the Dashboard](#7-how-to-use-the-dashboard)  
8. [Future Enhancements](#8-future-enhancements)  
9. [Data Source & Project Files](#9-data-source--project-files)  
10. [Contributors](#10-contributors)  

---

## **1. Business Case**  
### **1.1. Overview**  
Flight delays cost airlines billions annually and frustrate passengers. This project addresses:  
- **Operational Efficiency**: Pinpoint delay-prone routes, times, and carriers.  
- **Root Cause Analysis**: Breakdown delays by weather, carrier, or air traffic.  
- **Strategic Decision-Making**: Optimize schedules and resource allocation.  

### **1.2. Key Business Questions**  
- Which **airports** or **carriers** have the worst delay records?  
- How do delays vary by **season**, **day of the week**, or **time of day**?  
- What percentage of delays are preventable (e.g., carrier vs. weather)?  

---

## **2. Dataset Overview**  
The dataset includes **30+ columns** with granular flight details:  
- **Flight Metadata**: `FlightID`, `TailNum`, `CarrierCode`.  
- **Timestamps**: `CRSDepTime`, `ActualElapsedTime`, `ArrDelay`.  
- **Delay Causes**: `CarrierDelay`, `WeatherDelay`, `NASDelay`, `SecurityDelay`.  
- **Geospatial**: `Origin`, `Dest`, `Distance`.  
- **Status Flags**: `Cancelled`, `Diverted`, `CancellationCode`.  

---

## **3. Tools and Technologies Used**  
| Tool               | Purpose                                                                 |
|--------------------|-------------------------------------------------------------------------|
| **Power BI**       | Interactive dashboards and visualizations.                              |
| **Power Query**    | Data cleaning (handling nulls, type conversion, feature engineering).   |
| **DAX**            | Advanced metrics (e.g., "On-Time Performance %").                       |
| **GitHub**         | Version control and project documentation.                              |

---

## **4. Data Cleaning and Transformation**  
### **Key Steps**  
1. **Null Value Handling**:  
   - Replaced nulls in delay columns (`WeatherDelay`, `LateAircraftDelay`) with `0` for accurate aggregation.  
   ```m
   ReplaceNullValues = Table.ReplaceValue(Source, null, 0, Replacer.ReplaceValue, {"CarrierDelay", "WeatherDelay"})
   ```  

2. **Boolean Conversion**:  
   - Converted `Cancelled` and `Diverted` to `True/False` for intuitive filtering.  
   ```m
   ChangeType = Table.TransformColumnTypes(Source, {{"Cancelled", type logical}, {"Diverted", type logical}})
   ```  

3. **Time Formatting**:  
   - Standardized `DepTime`/`ArrTime` to `HH:MM` format.  
   ```m
   ConvertTime = Table.TransformColumns(Source, {{"DepTime", each Time.FromText(Text.PadStart(Text.From(_), 4, "0")), type time}})
   ```  

4. **Date Dimension**:  
   - Created a `DateKey` (YYYYMMDD) for seamless integration with time-based analysis.  
   ```m
   AddDateKey = Table.AddColumn(Source, "DateKey", each Date.ToText([Date], "YYYYMMDD"), type text)
   ```

---

## **5. Data Modeling**  
### **5.1. Star Schema Architecture**  
![Data Model](https://github.com/Mohammed1999sstack/Airline-Delay/blob/main/ScreenShots/DATA%20Moddling.png)  

### **5.2. Data Model Components**  
#### **Fact Table**  
- **Fact_Flights**: Core metrics like `DepDelay`, `ArrDelay`, `Cancelled`.  

#### **Dimension Tables**  
| Table               | Key Attributes                              |
|---------------------|--------------------------------------------|
| **Dim_Airport**     | `IATA`, `Region`, `Country`                |
| **Dim_Carrier**     | `CarrierCode`, `CarrierName`               |
| **Dim_Date**        | `DateKey`, `DayOfWeek`, `MonthName`        |
| **Dim_Time**        | `Hour`, `Minute`, `Time` (HH:MM)           |
| **Dim_Cancellation**| `CancellationCode`, `Reason`               |

### **5.3. DAX Dimension Tables**  
#### **Dim_Time**  
Creates a time dimension with **minute-level granularity** (1440 rows for 24 hours):  
```dax
Dim Time = 
VAR MinutesInDay = 1440
RETURN
    ADDCOLUMNS(
        GENERATESERIES(0, MinutesInDay - 1, 1),
        "Hour", INT([Value] / 60),
        "Minute", MOD([Value], 60),
        "Time", TIME(INT([Value] / 60), MOD([Value], 60), 0)
    )
```

#### **Dim_Date**  
Generates a full **2008 calendar year** with hierarchical attributes:  
```dax
Dim Date = 
VAR startDate = DATE(2008, 1, 1)
VAR endDate = DATE(2008, 12, 31)
RETURN
    ADDCOLUMNS(
        CALENDAR(startDate, endDate),
        "Year", YEAR([Date]),
        "Month", MONTH([Date]),
        "Day", DAY([Date]),
        "Month Name", FORMAT([Date], "MMMM"),
        "Day Name", FORMAT([Date], "DDD"),
        "dateKey", FORMAT([Date], "yyyyMMdd")
    )
```

### **5.4. Relationships**  
- `Fact_Flights[DateKey]` → `Dim_Date[DateKey]`  
- `Fact_Flights[Dest]` → `Dim_Airport[IATA]`  
- `Fact_Flights[CRSDepTime]` → `Dim_Time[Time]` *(for time-based analysis)*  

---

## **6. Power BI Dashboard**  
### **Interactive Report Pages**  

#### **1. Home Page**  
![Home Page](https://github.com/Mohammed1999sstack/Airline-Delay/blob/main/ScreenShots/Home%20page.png)  
**Description**:  
The landing page introduces the dashboard's scope with navigation buttons to explore delay trends, cancellation reasons, and carrier performance. The clean design emphasizes usability with intuitive filters and a professional layout.

---

#### **2. Overview Page**  
![Overview Page](https://github.com/Mohammed1999sstack/Airline-Delay/blob/main/ScreenShots/overview%20page.png)  
**Description**:  
- **Key Metrics Cards**: Displays total flights (1.9M), delayed flights (73.7%), and cancellation rate (6.3%)  
- **Top 10 Analysis**: Bar charts highlight worst-performing carriers and airports  
- **Trend Visuals**: Line charts show monthly patterns in delays and cancellations  
- **Regional Comparison**: Donut chart breaks down flights by geographic region  

---

#### **3. Geographical Analysis Page**  
![Geographical Page](https://github.com/Mohammed1999sstack/Airline-Delay/blob/main/ScreenShots/Geographical%20page.png)  
**Description**:  
- **Heat Map**: Color-coded by total delay hours (darker = worse delays)  
- **Performance Rankings**: Tables show top/bottom 5 airports by on-time performance  
- **Carrier Overlay**: Filter to compare airlines' regional performance  
- **Tooltip Details**: Hover over regions for delay breakdowns by cause  

---

#### **4. Drill-Through Pages**  
##### **Airport Details**  
![Airport Drill-Through](https://github.com/Mohammed1999sstack/Airline-Delay/blob/main/ScreenShots/Drill%20throught%20Airport.png)  
**Description**:  
- **Airport Profile**: Header shows airport code, name, and region  
- **Delay Metrics**: Average delay minutes and cancellation percentage  
- **Time Patterns**: Heatmap of delays by hour/day of week  
- **Delay Causes**: Donut chart of weather/carrier/NAS delays  

##### **Region Details**  
![Region Drill-Through](https://github.com/Mohammed1999sstack/Airline-Delay/blob/main/ScreenShots/Drill%20throught%20region.png)  
**Description**:  
- **Regional Benchmarking**: Compares to national averages  
- **Trend Analysis**: 12-month delay pattern visualization  
- **Carrier Performance**: Top delayed airlines in the region  

---

#### **5. Cancellations & Diversions Page**  
![Cancellations Page](https://github.com/Mohammed1999sstack/Airline-Delay/blob/main/ScreenShots/Cancel%20%26%20Drivted.png)  
**Description**:  
- **Root Cause Analysis**: Stacked bar chart of cancellation reasons  
- **Monthly Trends**: Line chart showing cancellation rate fluctuations  
- **Carrier Comparison**: Table ranking airlines by cancellation frequency  
- **Diversion Metrics**: Cards showing total diversions and diversion rate  

---

#### **6. Slicer Panel**  
![Slicer](https://github.com/Mohammed1999sstack/Airline-Delay/blob/main/ScreenShots/Slicer_view.png)  
**Description**:  
Dynamic filters control all dashboard pages:  
- **Date Range**: Year/Month selector or custom date picker  
- **Carrier Dropdown**: Multi-select airline filter  
- **Airport Selector**: Choose origin/destination airports  
- **Time Slider**: Filter by hour of day  

---

## **7. How to Use the Dashboard**  
1. **Apply Filters**: Use slicers to focus on specific time periods, airlines, or airports  
2. **Drill Down**: Click on any visual to see detailed breakdowns (e.g., click a bar in "Top Delayed Carriers")  
3. **Cross-Filter**: Selections automatically update all connected visuals  
4. **Reset Views**: Use the "Reset" button in the top navigation to clear filters  

---

## **8. Future Enhancements**  
- **Real-Time Integration**: Connect to live flight APIs for up-to-date monitoring  
- **Predictive Analytics**: Add machine learning to forecast delays  
- **Passenger Impact**: Incorporate customer satisfaction metrics  
- **Cost Analysis**: Estimate financial impact of delays  

---

## **9. Data Source & Project Files**  
- **Dataset**: [Kaggle Airline Delay Data](https://www.kaggle.com/datasets/giovamata/airlinedelaycauses)  
- **Power BI File**: [Download .pbix](https://github.com/Mohammed1999sstack/Airline-Delay)  
- **Full Screenshots**: [View Gallery](https://github.com/Mohammed1999sstack/Airline-Delay/tree/main/ScreenShots)  

---

## **10. Contributors**
- [**Mohammed Osama Alsamadoni**](https://github.com/Mohammed1999sstack)  
- [**Abdallah Mostafa Shrief**](https://github.com/bedoo123)  
- [**Yasmeen Raghep Mostafa**](https://github.com/yasmeenragheb)


**Feedback Welcome!** Please open issues or PRs with suggestions.  

--- 
