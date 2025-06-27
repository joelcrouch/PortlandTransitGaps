# Trimet coverage VS Density

## ETL Pipeline Steps (High-Level)

#### Phase 1: Extraction

    Get TriMet Route Data:

        Call TriMet RouteConfig API to get all routes, directions, and stops.

        Parse JSON response.

        Store in a pandas DataFrame or similar structure.

    Get TriMet Stop Locations:

        Call TriMet Stop Location API or extract from RouteConfig if it contains full stop details.

        Parse JSON, store lat/lon.

        Alternatively (Recommended): Download "Transit Stop Peak Arrival" GeoJSON/CSV from Portland Maps Open Data portal. This will give you stop locations and peak arrival counts directly.

    Get Population Data:

        Download Portland neighborhood boundaries (GeoJSON/Shapefile) from Portland Maps Open Data.

        Download Census Tract demographic data (population, land area) for Portland from data.census.gov.

#### Phase 2: Transformation

    Process TriMet Data:

        Create geopandas GeoDataFrames for routes (LineStrings) and stops (Points).

        Calculate Transit Frequency: If using raw TriMet Arrivals, develop a script to aggregate arrivals over a defined period (e.g., peak hours on weekdays) for each stop to determine average frequency/headway. If using Portland Maps "Transit Stop Peak Arrival," this step is largely done.  We should probably use static data for the first iteration, maybe we can harvest data for a couple weeks and see if we can add real time data to this as well...Compare and contrast the predicted trimet coverage with the real trimet coverage.

    Process Population Data:

        Create a geopandas GeoDataFrame for Portland neighborhood boundaries or Census Tracts.

        Join demographic data (population, land area) to the geographic polygons.

        Calculate population_density for each polygon.

    Calculate Transit Coverage Areas:

        For each transit stop, create a buffer (e.g., 400 meters or 0.25 miles radius).

        Merge all overlapping stop buffers to create a transit_service_area.geojson.

    Analyze Gaps:

        Perform a spatial intersection between neighborhood_population_density.geojson (or census tracts) and transit_service_area.geojson.

        For each neighborhood/tract:

            Calculate the area of the neighborhood that overlaps with the transit_service_area.

            Calculate transit_coverage_percentage = (Overlapping Area / Total Neighborhood Area) * 100.

            Attribute the average transit_frequency (derived from nearby stops or aggregated within the neighborhood's covered area) to each neighborhood.

            Consider the distance from the centroid of each population polygon to the nearest transit stop.

#### Phase 3: Loading

    This part is kinda thrown at the wall and see what sticks...we might not need a React app.  First we just have to decide on the data. All this geojson stuff is up in the air.  We have downloaded and insalled qgis, so amybe we should learn to use that.
    Generate Final GeoJSON: Create a final GeoJSON file for your React app. Each feature in this GeoJSON will be a neighborhood polygon with properties like:

        neighborhood_name

        population_density

        transit_coverage_percentage

        average_transit_frequency

        underserved_score (you can define a metric here, e.g., high density + low coverage/frequency)

    Store Data (Optional but Recommended): Save intermediate and final GeoJSON files to a designated data/ directory in your project for easy access by the frontend.

### Technologies for ETL:

    Python: The workhorse for this.

        requests: For API calls.

        pandas: For tabular data manipulation.

        geopandas: Essential for all spatial operations (reading/writing spatial files, buffering, intersections, unions, coordinate system transformations).

        shapely: Underlying geometry engine for geopandas.

        Fiona: For reading/writing spatial files.

        json: For handling JSON data from APIs.

Next Steps:

    TriMet API Key: Go to the TriMet Developer Resources and get your AppID.

    Explore Portland Open Data: Head to the Portland Maps open data portal and search for population data, neighborhood boundaries, and especially the "Transit Stop Peak Arrival" dataset. This is crucial as it might save us a lot of complex frequency calculations.

    Identify Geographic Unit: Decide if you want to analyze by Portland neighborhood boundaries, Census Tracts, or a grid (e.g., Hexagons if we want a more uniform spatial analysis). Using existing neighborhood boundaries from Portland Maps or Census Tracts might be the easiest starting point.
