--------------------------------------------------------------------------------
10 km grid
--------------------------------------------------------------------------------
10km_grid/10km_grid_wgs84/:
This is a folder that contains the shapefile for the 10 km grid.

--------------------------------------------------------------------------------
10km_grid/smokePM2pt5_predictions_daily_10km_20060101-20201231.rds:
This is a file that contains a data frame with the final set of daily smoke PM2.5 predictions on smoke days at 10 km resolution from January 1, 2006 to December 31, 2020 for the contiguous US. The 'grid_id_10km' column in this file corresponds to the 'ID' column in the 10 km grid shapefile.

All rows in this file are predictions on smoke days. Predictions on non-smoke days are by construction 0 ug/m^3 and not included in this file. A smoke PM2.5 prediction of 0 in this file means that the grid cell-day did have a smoke day but did not have elevated PM2.5. The full set of smoke PM2.5 predictions on both smoke days and non-smoke days can be obtained by setting the smoke PM2.5 prediction to 0 on grid cell-days in the 10 km grid and in the January 1, 2006-December 31, 2020 date range that are not in this file. For example, the R code below returns the full set of smoke PM2.5 predictions:

library(lubridate)
library(sf)
library(dplyr)
library(tidyr)

# Load smokePM predictions on smoke days
preds = readRDS("./final/10km_grid/smokePM2pt5_predictions_daily_10km_20060101-20201231.rds")

# Load 10 km grid
grid_10km = read_sf("./final/10km_grid/10km_grid_wgs84/10km_grid_wgs84.shp")

# Load full set of dates
dates = seq.Date(ymd("20060101"), ymd("20201231"), by = "day")

# Get full combination of grid cell-days
# Warning: this may require a large amount of memory
out = expand.grid(grid_id_10km = grid_10km$ID, date = dates)

# Match smokePM predictions on smoke days to grid cell-days
out = left_join(out, preds, by = c("grid_id_10km", "date"))

# Predict 0 for remaining grid cell-days, which are non-smoke days
out = mutate(out, smokePM_pred = replace_na(smokePM_pred, 0))

--------------------------------------------------------------------------------
10km_grid/smokePM2pt5_predictions_daily_10km_20060101-20201231.csv:
This is the same as smokePM2pt5_predictions_daily_10km_20060101-20201231.rds, except it is saved as a CSV file.
