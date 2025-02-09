# Airline-Delay
Overview
This project focuses on analyzing airline delays using a data warehouse (DWH) with a star schema design. The schema includes several dimensions and a fact table to store and analyze flight data, including delays, cancellations, and other relevant metrics.

Schema Description
The star schema consists of the following tables:

Dimensions
Dim Airport

airport: Name of the airport.

country_code: Country code where the airport is located.

iata: IATA code of the airport.

icao: ICAO code of the airport.

latitude: Latitude coordinate of the airport.

longitude: Longitude coordinate of the airport.

region_name: Region name where the airport is located.

Dim Carrier

Carrier Name: Name of the airline carrier.

IATA Carrier Code: IATA code for the carrier.

ICAO Carrier Code: ICAO code for the carrier.

Dim Cancellation

Cancellation code: Code representing the reason for cancellation.

Cancellation Reason: Description of the cancellation reason.

Dim Date

Date: Date in a standard format.

dateKey: Unique key for the date.

Day: Day of the month.

Day Name: Name of the day (e.g., Monday, Tuesday).

Month Name: Name of the month.

Month No.: Number representing the month.

Year: Year.

Fact Table
Fact Flights

Actual Elapsed Time: Actual time taken for the flight.

Air Time: Time spent in the air.

Air Delay: Delay caused by air traffic.

Air_Time: Time spent in the air (duplicate field, consider removing).

CancellationCode: Code indicating the reason for cancellation.

Cancelled: Flag indicating if the flight was cancelled.

CarrierDelay: Delay caused by the carrier.

CRSArr_Time: Scheduled arrival time.

CRSDep_Time: Scheduled departure time.

CRSElapsed Time: Scheduled elapsed time.

dateKey: Foreign key linking to the Dim Date table.

Dep_Delay: Departure delay.

Dep_time: Actual departure time.

Dest: Destination airport code.

Distance: Distance of the flight.

Diverted: Flag indicating if the flight was diverted.

flight id: Unique identifier for the flight.

Flight Num: Flight number.

LateAircraftDelay: Delay caused by late aircraft.

NASDelay: Delay caused by the National Airspace System.

Measures
Column: This section likely refers to the measures or metrics that can be calculated from the fact table, such as average delay times, total cancellations, etc.

Usage
To use this schema for analyzing airline delays:

Data Ingestion: Populate the dimension tables with relevant data (e.g., airport details, carrier information, date details).

Fact Table Population: Load flight data into the Fact Flights table, ensuring proper linkage to the dimension tables via foreign keys.

Analysis: Perform queries and analysis on the fact table to derive insights into flight delays, cancellations, and other metrics.