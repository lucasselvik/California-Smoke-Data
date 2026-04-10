# Workflow for California Smoke/Mobility Overlay

### 1. Make the grid
The smoke data has a `grid_id` feature (specific to the 10km grid) and the mobility data has a `GEOID` feature (geographic identifier used for census data). In order to plot these together, we need to include both features in the grid.
1. Download 10km grid file and a shapefile that specifies `GEOID`
2. Use Geopandas gpd.overlay() with an EPSG coordinate system to create a grid that includes both `grid_id` (which we will rename `grid_id_10km` to match smoke data) and `GEOID` (which we will rename `AREA` to match mobility data).

So now, this `grid` dataframe includes the following features:  `grid_id_10km`, `AREA`, and `geometry` (which came from the 10km grid file).

3. We also need to download a California shape file, saved as `grid_ca`.

### 2. Download and clean the smoke and mobility data
1. Download the smoke data and filter to the date range (8/1/2020 - 9/30/2020) and area (by merging with `grid_ca`)

This `smoke_2020_ca` dataframe has the following features: `grid_id_10km`, `AREA`, `geometry`, `date`, `smokePM_pred`.

2. Download the mobility data
3. Data is stored in the `STOPS_BY_DAY` feature. There is one row per month per `AREA`, where `STOPS_BY_DAY` is a list with the number of stops for each day of the month. So we need to explode these lists to make one row per day, saving the number of stops as `mobility_raw`.

Now, the `mobility` dataframe has the following features: `AREA`, `mobility_raw`, `date`.

4. Get the mobility anomaly:
