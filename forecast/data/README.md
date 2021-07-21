## Data availability

This folder contains some intermediate data to reproduce the manuscript. 
Small datasets are included in the github repo (along with the code to create them). 

The rest are available in persistent storage. All large data will be recreated
by rerunning scripts in the `code/` directory.

HRR data and 

### HRR data

`Hospital_Referral_Regions.csv`

Original source is https://hub.arcgis.com/datasets/fedmaps::hospital-referral-regions-1/about

Contains the FIPS codes and names for all 311 HRRs.

### confirmed_7dav signals

Both are generated by `code/download_signals/download_signals.R`. Alternatively

```r
library(covidcast)
df <- covidcast_signals(
  data_source = "jhu-csse",
  signal = c("confirmed_7dav_incidence_prop", "confirmed_7dav_incidence_num"),
  start_day = "2020-06-01",
  end_day = "2021-05-01",
  geo_type = "hrr",
  as_of = "2021-05-18"
)

saveRDS(df[[1]] %>%
          select(geo_value, time_value, value) %>%
          rename(target_end_date = time_value, actual = value),
        here::here("data", "confirmed_7dav_incidence_prop.RDS"))

saveRDS(df[[1]] %>%
          select(geo_value, time_value, value),
        here::here("data", "confirmed_7dav_incidence_num.RDS"))
```

### `offline_signals/`

Created using `code/download_signals/download_signals.R`.

Available at https://doi.org/10.5683/SP2/PUE88I.

### QR forecast results

1. `results_dishonest.RDS`
1. `results_honest.RDS`
1. `results_honest_bootstrapped.RDS`
1. `predictions_honest.RDS`

The above four files
are produced by running the scripts in `code/qr_forecast/`

Available at https://doi.org/10.5683/SP2/PUE88I.

Additional results will also be produced, but are not necessary to reproduce 
the manuscript.

### Hotspot results

1. `hotspots_dishonest.RDS`
1. `hotspots_honest.RDS`
1. `hotspots_honest_bootstrapped.RDS`

The above three files
are produced by running the scripts in `code/qr_forecast/`

Available at https://doi.org/10.5683/SP2/PUE88I.

Additional results will also be produced, but are not necessary to reproduce 
the manuscript.


### ny data

1. `ny_predictions.RDS`
1. `ny_actuals.RDS`

These are small subsets of `predictions_honest.RDS` and `confirmed_7dav_incidence_prop.RDS`
respectively. They are used for the Trajectory Figure in the manuscript. They can be generated with

```r
library(tidyverse)
ny_predictions <- readRDS(here::here("data", "predictions_honest.RDS")) %>%
  filter(forecaster == "AR3", forecast_date == "2020-10-15")
ny_actuals <- readRDS(here::here("data", "confirmed_7dav_incidence_prop.RDS")) %>%
  filter(geo_value == "303", target_end_date > "2020-09-01", target_end_date < "2020-11-15") %>%
  rename(value = actual, time_value = target_end_date)
```

### all signals

1. `all_signals_wide.RDS`

This data is used for the leading/lagging analysis in the manuscript. It can be recreated with

```r
library(tidyverse)
library(covidcast)
all_signals_wide <- covidcast_signals(
    data_source = c("jhu-csse", "fb-survey", "doctor-visits", "chng", "chng", "google-symptoms"),
    signal = c("confirmed_7dav_incidence_prop", "smoothed_hh_cmnty_cli", "smoothed_adj_cli",
      "smoothed_adj_outpatient_cli", "smoothed_adj_outpatient_covid",
      "sum_anosmia_ageusia_smoothed_search"),
    start_day = "2020-04-15",
    end_day = "2021-01-01",
    geo_type = "hrr",
    as_of = "2021-05-18") %>%
  aggregate_signals(format = "wide")
```