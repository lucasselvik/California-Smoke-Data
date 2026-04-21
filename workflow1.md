# Workflow for California Smoke/Mobility Overlay

### 1. Make the grid
The smoke data has a `grid_id` feature (specific to the 10km grid) and the mobility data has a `GEOID` feature (geographic identifier used for census data). In order to plot these together, we need to include both features in the grid.
1. Download 10km grid file and a shapefile that specifies `GEOID`
2. Use Geopandas gpd.overlay() with an EPSG coordinate system to create a grid that includes both `grid_id` (which we will rename `grid_id_10km` to match smoke data) and `GEOID` (which we will rename `AREA` to match mobility data).

So now, this `grid` dataframe includes the following features:  `grid_id_10km`, `AREA`, and `geometry` (which came from the 10km grid file).

3. Download a California shape file and merge `grid` onto it, creating the `grid_ca` dataframe which is filtered to California.

### 2. Download and clean the smoke and mobility data
1. Download the smoke data and filter to the date range (8/1/2020 - 9/30/2020) and area (by merging with `grid_ca`)

This `smoke_2020_ca` dataframe has the following features: `grid_id_10km`, `AREA`, `geometry`, `date`, `smokePM_pred`.

2. Download the mobility data.

Data is stored in the `STOPS_BY_DAY` feature. There is one row per month per `AREA`, where `STOPS_BY_DAY` is a list with the number of stops for each day of the month.

3. Explode these lists to make one row per day, saving the number of stops as `mobility_raw`.

Now, the `mobility` dataframe has the following features: `AREA`, `mobility_raw`, `date`.

4. Get the mobility anomaly:
    1. Make a `weekday` feature.
    2. Define a `baseline`: mean mobility for each day of the week.
    3. Define `mobility_anomaly`: the difference between mobility and the baseline (`mobility_raw`-`baseline`).
    4. Define `anomaly_pct`: (`mobility_anomaly` / `mobility_baseline`).

### 3. Add smoke and mobility data to grid

Before combining everything, we need to make sure all date-area combinations are accounted for in `grid_ca`.

1. Create an `all_area_dates` dataframe with every combination of dates in `smoke_2020` and areas in `mobility_data` and merge with `grid_ca` to create `grid_full`.
2. Merge `smoke_2020_ca` and `mobility` (only its relevant features) onto `grid_full`, creating `grid_final`, and turn this into a Geopandas GeoDataFrame.

`grid_final` has the following features: `grid_id_10km`, `AREA`, `geometry`, `date`, `smokePD_pred`, and `anomaly_pct`.

### 4. Animate data

1. Only smoke: plot `smokePD_pred` for each `geometry` for each `date`, with yellow representing low PM2.5 and red representing high PM2.5.
2. Only mobility: plot `anomaly_pct` for each `geometry` for each `date`, with blue representing negative mobility anomaly percent, white representing no anomaly, and red representing positive mobility anomaly percent.
3. Side-by-side: both plots side-by-side, synced by `date`.
4. Overlay: after normalizing `smokePM_pred` and `anomaly_pct`, plot `smoke_norm` as hue, from yellow to red, and `mobility_norm` as value, from light to dark.
