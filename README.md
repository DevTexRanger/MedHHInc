# Guide: Obtain Median Household Income for Specific TX Counties Using the 5YR ACS Across Multiple Years Using R & RStudio

This guide walks you through the process of obtaining and visualizing median household income data for specific Texas counties using the 5-year American Community Survey (ACS) data across multiple years.

## Load Required Libraries

First, load the necessary libraries using the `pacman` package. If you haven't installed `pacman`, it's highly recommended, especially if you're working with multiple packages.

```r
# Load pacman
library("pacman")

# Load libraries using p_load
p_load(dplyr, ggplot2, scales, tidyverse, tidycensus)
```

## Set Up Census Data API Key

Enter your Census Data API Key within the parentheses.

```r
# Census Data API Key
census_api_key("your_census_data_api_key_here")
```

## Check for Specific Table in Multiple Years

Check whether the specific table (`B19013A_001`) is available for each year from 2011 to 2023.

```r
# Define the years of interest
yrs <- 2011:2023
table_name <- "B19013A_001"

# Load variables for each year and check for the specific table
table_check <- sapply(yrs, function(year) {
  vars <- load_variables(year, "acs5", cache = TRUE)
  table_name %in% vars$variable
})

# Create a data frame to summarize the results
table_check_df <- data.frame(
  year = yrs,
  table_exists = table_check
)

# Print the results
print(table_check_df)

# Check if the table is present in all years
if (all(table_check)) {
  print(paste("The table", table_name, "is present in all years from 2011 to 2023."))
} else {
  print(paste("The table", table_name, "is not present in all years from 2011 to 2023."))
}
```

## Load Variables for the Latest "acs5"

Load the variables for the latest ACS 5-year estimates.

```r
# Load variables for the latest "acs5"
v23 <- load_variables(2023, "acs5", cache = TRUE)
```

## Identify the Table Name for Median Household Income

Use the `View()` command to inspect the variables and identify the table name for median household income.

```r
# View variables
View(v23)

# Create a variable name for median household income
medHHIncTX <- c(estimate = "B19013A_001")
```

## Specify Counties and Years

Specify the counties and years for which you want to obtain data.

```r
# Specify the counties as the vector 'my_counties'
my_counties <- c("Jones", "Taylor")

# Specify the years as a list using 'lst' from tidyverse
years <- lst(2011, 2012, 2013, 2014, 2015, 2016, 2017, 2018, 2019, 2020, 2021, 2022, 2023)
```

## Request Data for Median Household Income

Request the median household income data for the specified counties and years.

```r
# Request data for 2023 on the median household income for all TX counties
texas_income <- map_dfr(
  years,
  ~ get_acs(
    geography = "county",
    variables = medHHIncTX,
    state = "TX",
    county = my_counties,
    year = .x,
    survey = "acs5",
    geometry = FALSE,
    output = "wide"
  ),
  .id = "year"
)
```

## Rename Columns

Rename the columns for clarity.

```r
# Rename the columns
texas_income <- texas_income %>%
  rename(
    county = NAME,
    estimate = estimateE,
    MOE = estimateM
  )
```

## Visualize the Data

### Filter Data

Filter the data to include only the top 10 and bottom 10 counties based on median household income.

```r
# Filter data
top_bottom_20 <- texas_income %>%
  arrange(desc(estimate)) %>%
  slice(c(1:10, (n() - 9):n()))
```

### Clean Data

Remove duplicate entries.

```r
# Remove duplicates
top_bottom_20_clean <- top_bottom_20 %>%
  distinct(year, county, .keep_all = TRUE)
```

### Scatter Plot

Create a scatter plot to visualize the data.

```r
# Scatter plot
ggplot(top_bottom_20, aes(x = estimate, y = reorder(county, estimate))) + 
  geom_point(aes(color = as.factor(year)), size = 3) + 
  theme_minimal(base_size = 12.5) + 
  labs(title = "Median household income", 
       subtitle = "Counties in Texas", 
       x = "2018-2022 ACS estimate", 
       y = "", 
       color = "Year") + 
  scale_x_continuous(labels = label_dollar())
```

![Scatter plot using ggplot](https://github.com/DevTexRanger/MedHHInc/blob/main/ggplot_scatter_plot.png
)

### Error Bars Plot

Create a plot with error bars to visualize the margins of error.

```r
# Error bars plot
ggplot(top_bottom_20, aes(x = estimate, y = reorder(county, estimate))) + 
  geom_errorbarh(aes(xmin = estimate - MOE, xmax = estimate + MOE, color = as.factor(year))) + 
  geom_point(aes(color = as.factor(year)), size = 3) + 
  theme_minimal(base_size = 12.5) + 
  labs(title = "Median household income", 
       subtitle = "Counties in Texas", 
       x = "2011-2022 ACS 5YR Estimate", 
       y = "", 
       color = "Year") + 
  scale_x_continuous(labels = label_dollar())
```

![Scatter plot using ggplot](https://github.com/DevTexRanger/MedHHInc/blob/main/ggplot_error_bars.png
)

### Arrange Data by Margin of Error

Arrange the data in descending order by the margin of error.

```r
# Arrange data by margin of error
texas_income %>% 
  arrange(desc(MOE))
```

## Save Data to CSV File

Save the cleaned and filtered data to a CSV file. Remember to use double backslashes (\\) as a directory separator in file paths if you're on Windows.

```r
# Save the output as a CSV file
write.csv(texas_income, "C:...\\texasincome.csv")
```

---

This guide should help you understand and execute the code step-by-step.
