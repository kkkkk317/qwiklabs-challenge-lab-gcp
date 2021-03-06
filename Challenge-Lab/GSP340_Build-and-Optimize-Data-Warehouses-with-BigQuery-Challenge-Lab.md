# Build and Optimize Data Warehouses with BigQuery: [Challenge Lab](https://www.qwiklabs.com/focuses/14341?parent=catalog)

## Challenge scenario
You are part of an international public health organization which is tasked with developing a machine learning model to predict the daily case count for countries during the Covid-19 pandemic. As a junior member of the Data Science team you've been assigned to use your data warehousing skills to develop a table containing the features for the machine learning model. You receive the following request to complete the following tasks:

- Create a new dataset and table
- Add columns for population, country_area and a record column (named mobility) 
- Modify SQL template before populate the population data 
- Extract average values for the the six component fields that comprise the mobility 
- Run a query that returns a combined list of the DISTINCT countries that do not have any population data and countries which do not have country area information, ordered by country name

Some standards and tips you should follow:
- Exclude United Kingdom **(GBR)** and the United States **(USA)** data from your initial table.
- When updating the schema for a BigQuery table you can use the console to add the columns and record elements or you can use the command line **bq** utility to update the schema by providing a JSON file.
- The **covid19_ecdc** table contains a **population** column that you can use to populate the populationcolumn in your dataset.
- The **country_names_area** table does not contain a three letter country code column, but you can join it to your table using the full text **country_name** column that exists in both tables.
- When updating the mobility record remember that you must select (and average) a number of records for each country and date combination so that you get a single average of each child column in the mobility record.

### Task 1: Create a table partitioned by date
Create a new dataset and create a table in that dataset partitioned by date, with an expiry of 90 days. The table should initially use the schema defined for the **oxford_policy_tracker** table in the [COVID 19 Government Response public dataset](https://console.cloud.google.com/bigquery?p=bigquery-public-data&d=covid19_govt_response&page=dataset).

You must also populate the table with the data from the source table for all countries except the United Kingdom **(GBR)** and the United States **(USA)**.

Navigate to the BigQuery console, select **Navigation menu > BigQuery**. Select **GCP Project ID** then click **CREATE DATASET** button.

![Screenshot](https://github.com/kkkkk317/qwiklabs-challenge-lab-gcp/blob/main/img/Build-and-Optimize-BigQuery-1.png)

Write **covid** in the Dataset ID field. And click **Create dataset** button.

![Screenshot](https://github.com/kkkkk317/qwiklabs-challenge-lab-gcp/blob/main/img/Build-and-Optimize-BigQuery-2.png)

You can see **covid dataset** is created. Now put SQL query on the blank field.

![Screenshot](https://github.com/kkkkk317/qwiklabs-challenge-lab-gcp/blob/main/img/Build-and-Optimize-BigQuery-3.png)

SQL Query:
```
 CREATE OR REPLACE TABLE covid.oxford
 PARTITION BY date
 OPTIONS(
   partition_expiration_days=90
 )
 AS
 SELECT *
 FROM `bigquery-public-data.covid19_govt_response.oxford_policy_tracker`
 WHERE alpha_3_code NOT IN ('GBR', 'USA')
```

### Task 2: Add new columns to your table
Update your table to add new columns to your table with the appropriate data types to ensure alignment with the specification provided to you.

SQL Query:
```
 ALTER TABLE covid.oxford
 ADD COLUMN population INT64,
 ADD COLUMN country_area FLOAT64,
 ADD COLUMN mobility STRUCT<
  avg_retail FLOAT64,
  avg_grocery FLOAT64,
  avg_parks FLOAT64,
  avg_transit FLOAT64,
  avg_workplace FLOAT64,
  avg_residential FLOAT64
  >
```

### Task 3: Add country population data to the population column
Add the country population data to the population column in your table with **covid_19_geographic_distribution_worldwide** table data from the [European Center for Disease Control COVID 19 public dataset](https://console.cloud.google.com/bigquery?p=bigquery-public-data&d=covid19_ecdc&page=dataset) table.

SQL Query:
```
UPDATE covid.oxford A
SET A.population = B.pop_data_2019
FROM `bigquery-public-data.covid19_ecdc.covid_19_geographic_distribution_worldwide` B
WHERE A.date = B.date AND A.alpha_3_code = B.country_territory_code
```

### Task 4: Add country area data to the country_area column
Add the country area data to the country_area column in your table with **country_names_area** table data from the [Census Bureau International public dataset](https://console.cloud.google.com/bigquery?p=bigquery-public-data&d=census_bureau_international&page=dataset).

SQL Query:
```
UPDATE covid.oxford A
SET A.country_area = B.country_area
FROM `bigquery-public-data.census_bureau_international.country_names_area` B
WHERE A.country_name = B.country_name
```

### Task 5: Populate the mobility record data
Populate the mobility record in your table with data from the [Google COVID 19 Mobility public dataset](https://console.cloud.google.com/bigquery?p=bigquery-public-data&d=covid19_govt_response&page=dataset).

SQL Query:
```
UPDATE covid.oxford A
SET A.mobility = STRUCT<
  avg_retail FLOAT64,
  avg_grocery FLOAT64,
  avg_parks FLOAT64,
  avg_transit FLOAT64,
  avg_workplace FLOAT64,
  avg_residential FLOAT64>
  (B.avg_retail, B.avg_grocery, B.avg_parks, B.avg_transit, B.avg_workplace, B.avg_residential)
FROM (SELECT country_region, date, 
      AVG(retail_and_recreation_percent_change_from_baseline) as avg_retail,
      AVG(grocery_and_pharmacy_percent_change_from_baseline) as avg_grocery,
      AVG(parks_percent_change_from_baseline) as avg_parks,
      AVG(transit_stations_percent_change_from_baseline) as avg_transit,
      AVG(workplaces_percent_change_from_baseline) as avg_workplace,
      AVG(residential_percent_change_from_baseline) as avg_residential
      FROM `bigquery-public-data.covid19_google_mobility.mobility_report`
      GROUP BY country_region, date) AS B
WHERE A.country_name = B.country_region AND A.date = B.date
```

### Task 6: Query missing data in population & country_area columns
Run a query to find the missing countries in the population and country_area data. The query should list countries that do not have any population data and countries that do not have country area information, ordered by country name. If a country has neither population or country area it must appear twice.

SQL Query:
```
SELECT DISTINCT country_name
FROM covid.oxford
WHERE population IS NULL
UNION ALL
SELECT DISTINCT country_name
FROM covid.oxford
WHERE country_area IS NULL
ORDER BY country_name ASC
```


## Congratulations!
<img src="https://github.com/kkkkk317/qwiklabs-challenge-lab-gcp/blob/main/img/BigQuery-for-Data-Warehousing.png" height="150" /> <img src="https://github.com/kkkkk317/qwiklabs-challenge-lab-gcp/blob/main/img/Build-and-Optimize-Data-Warehouses-with-BigQuery.png" height="150" />
