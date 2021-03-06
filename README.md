# Bus-Capstone
Capstone project for NYC Department of Transportation.

### Important documentation:

*   Documentation of data processing and Spark
    * __Python Data Processing__
        * [Demonstration](demonstration/): Ipython Notebooks that demonstrate all the processes

        * Core Modules

            1. [Siri Tools](siri_tools/): Modules for Bus Time data retrieval and cleaning
            2. [Time Tools](ttools.py): Homemade Timedelta Converter
            3. [GTFS](gtfs.py): Extract the Schedule Data from GTFS Schedules(originally in ZIP)
            4. [Arrival Time](arrivals.py): Estimate the arrival time for each stop using Scipy [KD-Tree](http://docs.scipy.org/doc/scipy-0.14.0/reference/generated/scipy.spatial.KDTree.html) and [Interpolate](http://docs.scipy.org/doc/scipy/reference/generated/scipy.interpolate.interp1d.html)
            5. [Performance Metrics](metrics.py): Calculate common performance measurements on each route, stop and date.

    * __[Big Data with SPARK](Spark#parse-and-manipulate-bus-time-data-using-pyspark)__

      * For details, check the [Spark](Spark/) folder.

*   [Sponsor report](https://github.com/sarangof/Bus-Capstone/blob/master/paper/sponsor_report_final.pdf)

*   [Technical report](https://github.com/sarangof/Bus-Capstone/blob/master/paper/technical_report.pdf)

## Final Product Sample
Darker means poorer on time performance for the buses

![alt text](https://github.com/sarangof/Bus-Capstone/blob/master/plots/on_time_performance_stops.png "Sample of on time performance")

Open Interactive Map in [Carto Map](https://saf537.carto.com/viz/c21efdeb-ec45-45f2-b2d3-c47993bb89ff/public_map)

## ETL procedure
  
1. **Bus Time data** (use [siri_tools](siri_tools/))
  1. Scrape: Query the Bus Time API every 60 seconds and write each JSON response to a local file.  It is recommended to run two independent scrape processes (separated by 30 seconds) to get maximum data density.  This minimizes the interruptions from some responses taking longer than 30 seconds.  
  **Requirements:**  
    * `MYKEY` file located in the OS working directory containing a single text string.  See [Bus Time documentation](http://bustime.mta.info/wiki/Developers/Index) for instructions on getting a key.
    * `jsons/` directory exists in the OS working directory  	
  2. Parse: Extract useful data elements from each vehicle record in each JSON response file.  Takes roughly one second to parse one JSON, so an entire day's worth data may take up to 15 minutes.  Speed is significiantly faster using the [Spark](Spark#parse-and-manipulate-bus-time-data-using-pyspark) code.  
  3. Clean: Using schedule data as the "truth" source, filter extracted and parsed Bus Time data to exclude any records where the reported "next stop" is invalid for the reported `trip_id`. 
  
2. **Schedule data**
  1. Download: Static feeds of the current schedule data for each borough (plus the MTA Bus Company) are available [directly from the MTA](http://web.mta.info/developers/developer-data-terms.html#data).  Historical feeds are available through a [third-party open-source project](http://transitfeeds.com/p/mta).  Shell script to download all previous feeds in one batch can be found in the [Bus Viz github] (https://github.com/efranco63/NYU_USI_BusViz/blob/master/TransitFeeds/fetch.sh).
  2. Generate metadata (list of date ranges): Use *method* [gtfs](gtfs.py).**build_metadata**(dpath) to generate a small text file within each subdirectory of `dpath` that lists the valid date ranges of each included feed.  This is necessary since schedule data changes periodically, so any schedule-comparison analysis must use only data extracted from the corresponding concurrent feed.  
  **Requirements:**
    * All downloaded transit feed files must be in their original standard format (zip)
    * Each feed gets its own subdirectory, containing current and prior feeds  
  **Example directory structure for GTFS data**
  ```
  gtfs/  
    80_brooklyn/  
      metadata.txt  
      gtfs_brooklyn_1383136207.zip  
      gtfs_brooklyn_1419914436.zip  
      gtfs_brooklyn_1386879331.zip  
      gtfs_brooklyn_20150402.zip  
    82_manhattan/  
    84_staten_island/  
    81_bronx/  
    83_queens/  
    85_bus_company/
  ```
	
3. **Stop time estimation**
  1. **Recommended**: Linear interpolation (see [demonstration notebook](demonstration/interpolate_stop_times.ipynb))
  2. **Alternative**: Spatial search (see [demonstration script](demonstration/stop_times_spatial.py))  

4. **Performance metrics**
  1. **Recommended**: Generate a single measurement for each route at the stop with the most data (see [demonstration notebook](demonstration/common_metrics.ipynb))
  2. **Alternative**: Batch process metrics for all stops, routes and dates before filtering and analyzing (use [metrics.py](metrics.py))  
  Example: `python metrics.py dec2015_interpolated.csv gtfs/ dec2015_metrics.csv`
