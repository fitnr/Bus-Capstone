# Spark_project

## This project reads 3 TB of nested JSON data and apply bunch of spark techniques to analysis the bus delays and headways

## Techniques Included

Large!!!

Read JSON 
```
sqlContext.read.json()
```

Flatten Arrays using flatMap
```
(1,2,3) (a,b,c) ($,#,&) => (1,a,$),(2,b,#),(3,c,&)
```
groupByKey & Interpolate
```
use groupBykey to cast interpolation of time&distance to all trips
Interpolate1D 
Link:http://docs.scipy.org/doc/scipy/reference/generated/scipy.interpolate.interp1d.html#scipy.interpolate.interp1d
```
Read csv using Spark_CSV tool
```
sqlContext.read.format('com.databricks.spark.csv').options(header='true').load()
```
(https://github.com/databricks/spark-csv "Spark_CSV_package")

SparkSQL manipulation

`Inner join` the interpolated data with GTFS schedule
`IF` `COUNT` to calculate the multiple on time perfornace.
For more info. Refer `ontime_ratio/ontime_ratio.sql`

## Data Schema
```
| JSON ELEMENT(schema)                           | Column NAME    | explanation                                                 |                           |                 | 
|------------------------------------------------|----------------|-------------------------------------------------------------|---------------------------|-----------------| 
| LineRef                                        | ROUTE_ID       | Name of bus line(B42)                                       |                           |                 | 
| VehicleLocation.Latitude                       | latitude       | latitude of record                                          |                           |                 | 
| VehicleLocation.Longitude                      | longitude      | longitude of record                                         |                           |                 | 
| RecordedAtTime                                 | recorded_time  | What time it get recorded                                   |                           |                 | 
| VehicleRef                                     | vehicle_id     | ID of vehicle                                               |                           |                 | 
| FramedVehicleJourneyRef.DatedVehicleJourneyRef | TRIP_ID        | Same as trip_id in GTFS*                                    |                           |                 | 
| FramedVehicleJourneyRef.DataFrameRef           | trip_date      | Date of the trip                                            |                           |                 | 
| JourneyPatternRef                              | SHAPE_ID       | Same as shape_id in GTFS*                                   |                           |                 | 
| StopPointRef                                   | STOP_ID        | "Id of next stop                                            | Same as stop_id in GTFS*" |                 | 
| Extensions.Distances.DistanceFromCall          | distance_stop  | Distance to next stop                                       |                           |                 | 
| Extensions. Distances.CallDistanceAlongRoute   | distance_shape | Stop_s total distance along the shape                       |                           |                 | 
| Extensions. Distances.PresentableDistance      | status         | "Report the current status of bus to next stop( approaching |  at stop                  |  <1 stop away)" | 
| DestinationRef                                 | destination    | Headsign of bus                                             |                           |                 | 
```