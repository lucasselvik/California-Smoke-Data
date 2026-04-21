# Workflow for Smoke-Mobility-Census-Level
## 1. Import data
1. California census tracts shapefile (`./data/tract/tl_2019_06_tract/tl_2019_06_tract.shp`)
    * `grid` has `GEOID` and `geometry`features.
2. Smoke data (download `smokePM2pt5_predictions_daily_tract_20060101-20201231.csv` from [this Dropbox](https://www.dropbox.com/scl/fo/18ll8pa4nc0lo76u3a4lt/AER1GJnXH66zmdQJvRoOBeo?rlkey=bi1ry6wx2g9ddq83ufxnuywon&e=1&st=mz7gp2su&dl=0) and put it in `.\data\tract`)
    * Filter to the date range (8/1/2020 - 9/30/2020)
    * `smoke` has `GEOID`, `date`, and `smokePM_pred` features.
3. Mobility data (download `california_202008_202009.csv` from our Box and put it in `./data`)
    1. Data is stored in the `STOPS_BY_DAY` feature. There is one row per month per `AREA`, where `STOPS_BY_DAY` is a list with the number of stops for each day of the month. Explode these lists to make one row per day, saving the number of stops as `mobility_raw`.
    2. Now `mobility` has `GEOID` (previously named `AREA`, `mobility_raw`, and `date` features.
    3. `GEOID` has an extra digit of precision at the end compared to `grid` and `smoke`, so we need to remove this digit and average the `mobility_raw` values for all instances of `GEOID` with the same first 10 digits.
    4. There was relatively little PM2.5 in the first two weeks of August 2020, so we will use this as baseline mobility:
        1. Group by weekday.
        2. Calculate the mean `mobility_raw` per weekday for the first two weeks of August 2020: these are our `baseline` values.
        3. Define `mobility_anomaly`: the difference between mobility and the baseline (`mobility_raw`-`baseline`).
        4. Define `anomaly_pct`: (`mobility_anomaly` / `mobility_baseline`).
    * Now, `mobility` has `GEOID`, `date`, and `anomaly_pct` features.

## 2. Combine data onto one grid
1. Create a grid called `space-time-grid` with all `GEOID`s and `date`s.
2. Merge `smoke` and `mobility` onto this grid to create `grid_full`, which contains features `GEOID`, `date`, `smokePD_pred`, and `anomaly_pct`.
3. Merge `geometry` onto `grid_full` and convert to a geopandas GeoDataFrame, to create `grid_final`, which contains features `GEOID`, `date`, `smokePD_pred`, `anomaly_pct`, and `geometry`.

## 3. Animate data

1. Only smoke: plot `smokePD_pred` for each `geometry` for each `date`, with yellow representing low PM2.5 and red representing high PM2.5.
2. Only mobility: plot `anomaly_pct` for each `geometry` for each `date`, with blue representing negative mobility anomaly percent, white representing no anomaly, and red representing positive mobility anomaly percent.
3. Side-by-side: both plots side-by-side, synced by `date`.
4. Overlay: after normalizing `smokePM_pred` and `anomaly_pct`, plot `smoke_norm` from white to red and `mobility_norm` as white to blue, as a bivariate choropleth.
    
