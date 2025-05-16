# SkeeterMeter

This repository contains the data, code, and Shiny application for SkeeterMeter, a statistical model and visualization tool that forecasts mosquito trap counts across Winnipeg using weather-driven GLMMs. Developed with glmmTMB, it leverages 14-day rolling averages of temperature, precipitation, and humidity, incorporating trap- and year-level random effects and seasonal splines. Built for public health insights and risk communication.

## **Statistical Analysis**

To model temporal patterns in adult mosquito abundance in Winnipeg, I fit a generalized linear mixed model (GLMM) using the `glmmTMB` package (version 1.1.11) in R (version 4.5.0). The outcome of interest was the mosquito trap count per trap per day, derived from daily municipal surveillance records. Observations with missing values or non-numeric data were excluded from analysis.

## **Response Variable and Data Processing**

The response variable was the number of mosquitoes captured per trap per day. Daily trap count data were sourced from the City of Winnipeg and pre-processed to standardize date formats, remove non-numeric entries (e.g., "no data"), and pivot the dataset to a long format with one row per trap per day. Each trap was assigned to one of four Winnipeg quadrants or labeled as outside city limits based on its identifier. Data were augmented with epidemiological week (week) and year (year) fields for temporal modeling.

## **Meteorological Covariates**

Meteorological data, including mean temperature, total daily precipitation, and relative humidity, were retrieved from Environment and Climate Change Canada via the `weathercan` package (version 0.7.3.9000). Data were obtained from the Winnipeg Richardson International Airport (station ID: 27174), the longest-standing daily meteorological station in the city.

To capture lagged weather effects, I calculated 14-day rolling means for the following:

1. Degree days above 10°C (daily mean temperature minus 10°C, truncated at 0),
2. Mean precipitation
3. Relative humidity

These values were matched to mosquito trap counts based on date.

## **Model Specification**

I modeled mosquito abundance using a negative binomial GLMM with a log link to account for overdispersion common in count data. The model included:

1. Fixed effects: second-degree polynomial terms for rolling mean temperature and precipitation, and a linear term for relative humidity.
2. Spline term: a natural cubic spline (`ns()`) with 4 degrees of freedom was included on the week variable to flexibly model seasonal trends in mosquito abundance, such as the mid-summer peak commonly observed in temperate regions.
3. Random intercepts: to account for repeated measures and spatial/temporal clustering, random intercepts were included for both trap site and year.

The final model was specified as follows:
```
glmmTMB(
  number ~ poly(mean_temp_14, 2) + poly(mean_rh_14, 1) +
    poly(total_rain_14, 2) + ns(week, df = 4) +
    (1 | trap) + (1 | year),
  family = nbinom2,
  data = model_data
)
```
Model selection was based on ecological reasoning and preliminary model comparisons using AIC. Predictions were generated using the `predict()` function with the `type = "response"` argument to obtain estimates on the count scale. These were plotted alongside observed counts to assess model fit.

# References

- **glmmTMB**: Brooks ME, Kristensen K, van Benthem KJ, Magnusson A, Berg CW, Nielsen A, Skaug HJ, Maechler M, Bolker BM (2017). “glmmTMB Balances Speed and
Flexibility Among Packages for Zero-inflated Generalized Linear Mixed Modeling.” _The R Journal_, *9*(2), 378-400. doi:10.32614/RJ-2017-066
<https://doi.org/10.32614/RJ-2017-066>. | McGillycuddy M, Warton DI, Popovic G, Bolker BM (2025). “Parsimoniously Fitting Large Multivariate Random Effects in glmmTMB.” _Journal of
Statistical Software_, *112*(1), 1-19. doi:10.18637/jss.v112.i01 <https://doi.org/10.18637/jss.v112.i01>.

- **weathercan**: LaZerte S, Albers S (2018). “weathercan: Download and format weather data from Environment and Climate Change Canada.” _The Journal of Open
Source Software_, *3*(22), 571. <https://joss.theoj.org/papers/10.21105/joss.00571>.

- **dplyr**: Wickham H, François R, Henry L, Müller K, Vaughan D (2023). _dplyr: A Grammar of Data Manipulation_. doi:10.32614/CRAN.package.dplyr
<https://doi.org/10.32614/CRAN.package.dplyr>, R package version 1.1.4, <https://CRAN.R-project.org/package=dplyr>.

- **ggplot2**: Wickham H (2016). _ggplot2: Elegant Graphics for Data Analysis_. Springer-Verlag New York. ISBN 978-3-319-24277-4,
<https://ggplot2.tidyverse.org>.

- **lubridate**: Grolemund G, Wickham H (2011). “Dates and Times Made Easy with lubridate.” _Journal of Statistical Software_, *40*(3), 1-25.
<https://www.jstatsoft.org/v40/i03/>.

- **readr**: Wickham H, Hester J, Bryan J (2024). _readr: Read Rectangular Text Data_. doi:10.32614/CRAN.package.readr
<https://doi.org/10.32614/CRAN.package.readr>, R package version 2.1.5, <https://CRAN.R-project.org/package=readr>.

- **janitor**: Firke S (2024). _janitor: Simple Tools for Examining and Cleaning Dirty Data_. doi:10.32614/CRAN.package.janitor
<https://doi.org/10.32614/CRAN.package.janitor>, R package version 2.2.1, <https://CRAN.R-project.org/package=janitor>.

- **DHARMa**: Hartig F (2024). _DHARMa: Residual Diagnostics for Hierarchical (Multi-Level / Mixed) Regression Models_. doi:10.32614/CRAN.package.DHARMa
<https://doi.org/10.32614/CRAN.package.DHARMa>, R package version 0.4.7, <https://CRAN.R-project.org/package=DHARMa>.

- **performance**: Lüdecke D, Ben-Shachar M, Patil I, Waggoner P, Makowski D (2021). “performance: An R Package for Assessment, Comparison and Testing of
Statistical Models.” _Journal of Open Source Software_, *6*(60), 3139. doi:10.21105/joss.03139 <https://doi.org/10.21105/joss.03139>.

- **car**: Fox J, Weisberg S (2019). _An R Companion to Applied Regression_, Third edition. Sage, Thousand Oaks CA. <https://www.john-fox.ca/Companion/>.

- **rstatix**: Kassambara A (2023). _rstatix: Pipe-Friendly Framework for Basic Statistical Tests_. doi:10.32614/CRAN.package.rstatix
<https://doi.org/10.32614/CRAN.package.rstatix>, R package version 0.7.2, <https://CRAN.R-project.org/package=rstatix>.

- **tidyr**: Wickham H, Vaughan D, Girlich M (2024). _tidyr: Tidy Messy Data_. doi:10.32614/CRAN.package.tidyr <https://doi.org/10.32614/CRAN.package.tidyr>,
R package version 1.3.1, <https://CRAN.R-project.org/package=tidyr>.

- **purrr**: Wickham H, Henry L (2025). _purrr: Functional Programming Tools_. doi:10.32614/CRAN.package.purrr <https://doi.org/10.32614/CRAN.package.purrr>,
R package version 1.0.4, <https://CRAN.R-project.org/package=purrr>.

- **tibble**: Müller K, Wickham H (2023). _tibble: Simple Data Frames_. doi:10.32614/CRAN.package.tibble <https://doi.org/10.32614/CRAN.package.tibble>, R
package version 3.2.1, <https://CRAN.R-project.org/package=tibble>.

- **stringr**: Wickham H (2023). _stringr: Simple, Consistent Wrappers for Common String Operations_. doi:10.32614/CRAN.package.stringr
<https://doi.org/10.32614/CRAN.package.stringr>, R package version 1.5.1, <https://CRAN.R-project.org/package=stringr>.

- **splines**: R Core Team (2025). _R: A Language and Environment for Statistical Computing_. R Foundation for Statistical Computing, Vienna, Austria.
<https://www.R-project.org/>.

- **shiny**: Chang W, Cheng J, Allaire J, Sievert C, Schloerke B, Xie Y, Allen J, McPherson J, Dipert A, Borges B (2024). _shiny: Web Application Framework
for R_. doi:10.32614/CRAN.package.shiny <https://doi.org/10.32614/CRAN.package.shiny>, R package version 1.10.0,
<https://CRAN.R-project.org/package=shiny>.

- **slider**: Vaughan D (2024). _slider: Sliding Window Functions_. doi:10.32614/CRAN.package.slider <https://doi.org/10.32614/CRAN.package.slider>, R package
version 0.3.2, <https://CRAN.R-project.org/package=slider>.
