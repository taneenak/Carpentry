# Introduction

Hello everyone! Today, we're diving into an exciting data analysis project using R. Our focus will be on analyzing goat-related agricultural data and visualizing it geographically using geospatial tools.

This session is hands-on, and I'll guide you through the entire process step-by-step. By the end, you’ll understand how to explore datasets, clean and join them, and create visualizations.

# Overview of the Project
# Objective:
To analyze goat-related data, specifically focusing on operations with sales at the county level in the United States. We'll visualize this data on a map to uncover regional trends.

# Questions We’re Answering:

What do goat sales look like across the U.S. at a county level?
How can we clean and prepare this data for geospatial analysis?
How do different regions, like Missouri and Mississippi, compare?
Setting Up the Environment
Instructor:
Before we start coding, let’s load the necessary libraries. Each library has a specific purpose:

```r 
library(sf)
library(ggplot2)
library(dplyr)
library(viridis)
library(readr)
library(tigris)
library(stringr)
``` 
sf: This library is used for geospatial data handling. It helps us work with maps and shapefiles.
ggplot2: A fantastic tool for creating visualizations, including geospatial maps.
dplyr: Makes data manipulation like filtering and joining very easy.
viridis: Provides beautiful, accessible color scales for our visualizations.
readr: Reads CSV files into R quickly and efficiently.
tigris: Downloads shapefiles, which define geographic boundaries like counties and states.
stringr: Helps with string manipulations, like cleaning and formatting text.
Expected Output: No errors should appear when these libraries are loaded.

# Step 1: Load the Dataset
We’ll start by loading the dataset into R. Here’s how to set the working directory and read the CSV file:

```r 
setwd("~/Lab R/Cattle Supporting Data")
goat <- read_csv("goat.csv")
```
setwd(): Tells R where to find your dataset.
read_csv(): Reads the goat.csv file into a data frame called goat.
Expected Output: A message summarizing the dataset, such as column names and row counts.

# Step 2: Examine the Dataset
Let’s explore the dataset by listing all column names and understanding what they represent:

```r 
colnames(goat)
``` 
colnames(): Displays all the column names in the dataset.
Expected Output: A list of column names like State, County, Data Item, etc.
Next, we’ll check unique values for each column:

```r 
columns <- c("Program", "Year", "Period", ...)
for (col in columns) {
  cat("Unique values in column:", col, "\n")
  print(unique(goat[[col]]))
  cat("\n")
}
```
Purpose: Helps us understand the range of data in each column.
Expected Output: Lists of unique values for the specified columns.

# Step 3: Clean the Data
Now, we’ll select only the columns we need for analysis:

```r 
goat <- subset(goat, select = c("State", "State ANSI", "County", "County ANSI", "Data Item", "Value"))
```
subset(): Retains only the specified columns.
Expected Output: A smaller dataset with fewer columns.
# Step 4: Load County Shapefiles
Instructor:
To create a map, we need U.S. county boundaries. We’ll download and clean this data:

```r 
counties_sf <- counties(cb = TRUE, resolution = "20m", year = 2021) %>%
  st_as_sf() %>%
  mutate(
    NAME = str_to_title(NAME),
    STATEFP = sprintf("%02d", as.numeric(STATEFP))
  )
```
counties(): Downloads U.S. county shapefiles.
st_as_sf(): Converts the data into a spatial object.
mutate(): Cleans and formats the county names (NAME) and state codes (STATEFP).
Expected Output: A geospatial object with county boundaries.

# Step 5: Filter for Mainland U.S.
We’ll remove Hawaii and Alaska from our analysis since they distort maps:

```r 
counties_sf_mainland <- counties_sf %>%
  filter(!STATEFP %in% c("02", "15"))
```
filter(): Excludes rows with STATEFP codes for Alaska (02) and Hawaii (15).
Expected Output: A dataset with only mainland counties.

# Step 6: Join Data
Next, we’ll merge our goat data with the county shapefiles to prepare for mapping:

```r 
joined_data <- counties_sf %>%
  left_join(goat, by = c("STATEFP" = "State ANSI", "COUNTYFP" = "County ANSI"))
```
left_join(): Combines the two datasets based on matching state and county codes.
Expected Output: A merged dataset with both geospatial and goat data.

# Step 7: Visualize the Data
Instructor:
Let’s create a map showing operations with goat sales:

```r 
ggplot(data = joined_data) +
  geom_sf(aes(fill = Value)) +
  scale_fill_viridis_c(option = "inferno", trans = "log10") +
  theme_minimal() +
  labs(title = "County-Level GOATS - OPERATIONS WITH SALES", fill = "Sales")
geom_sf(): Adds geospatial shapes to the plot, with colors based on Value.
scale_fill_viridis_c(): Applies a color scale with log transformation for better visibility.
theme_minimal(): Simplifies the plot's appearance.
labs(): Adds titles and labels.
``` 
Expected Output: A map of the U.S., with counties colored based on goat sales.

# Step 8: Focus on Specific States
Finally, let’s zoom into individual states like Missouri and Mississippi. Here’s how we filter for Missouri:

```r 
joined_data %>%
  filter(State == "MISSOURI") %>%
  ggplot() +
  geom_sf(aes(fill = Value)) +
  ...
```
Expected Output: A map of Missouri showing goat sales by county.

# Repeat this for Mississippi or any other state.

# Conclusion
We’ve successfully:

Explored and cleaned the dataset.
Merged it with geospatial data.
Created maps to visualize regional trends.
Remember, the tools and techniques you’ve learned today are versatile and can be applied to many datasets.
