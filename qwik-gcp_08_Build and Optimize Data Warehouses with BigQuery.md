# Build and Optimize Data Warehouses with BigQuery

[qwiki page](https://www.qwiklabs.com/quests/147)

## BigQuery: Qwik Start - Command Line

```bash
bq show bigquery-public-data:samples.shakespeare
bq help query
bq query --use_legacy_sql=false \
'SELECT
   word,
   SUM(word_count) AS count
 FROM
   `bigquery-public-data`.samples.shakespeare
 WHERE
   word LIKE "%raisin%"
 GROUP BY
   word'
bq query --use_legacy_sql=false \
'SELECT
   word
 FROM
   `bigquery-public-data`.samples.shakespeare
 WHERE
   word = "huzzah"'
bq ls
bq ls bigquery-public-data:
bq mk babynames
wget http://www.ssa.gov/OACT/babynames/names.zip
unzip names.zip
bq load babynames.names2010 yob2010.txt name:string,gender:string,count:integer
bq ls babynames
bq show babynames.names2010
bq query "SELECT name,count FROM babynames.names2010 WHERE gender = 'F' ORDER BY count DESC LIMIT 5"
bq query "SELECT name,count FROM babynames.names2010 WHERE gender = 'M' ORDER BY count ASC LIMIT 5"
bq rm -r babynames
```

## Creating a Data Warehouse Through Joins and Unions

https://console.cloud.google.com/bigquery?p=data-to-insights&page=ecommerce

### Enrich ecommerce data with Machine Learning

[natural language](https://cloud.google.com/natural-language/) to find document

1. entities
2. sentiment with score and magnitude

```bash
#standardSQL
SELECT
  SKU,
  name,
  sentimentScore,
  sentimentMagnitude
FROM
  `data-to-insights.ecommerce.products`
ORDER BY
  sentimentScore DESC
LIMIT 5
```

```bash
# pull what sold on 08/01/2017
CREATE OR REPLACE TABLE ecommerce.sales_by_sku_20170801 AS
SELECT DISTINCT
  productSKU,
  SUM(IFNULL(productQuantity,0)) AS total_ordered
FROM
  `data-to-insights.ecommerce.all_sessions_raw`
WHERE date = '20170801'
GROUP BY productSKU
ORDER BY total_ordered DESC #462 skus sold
# standardSQL
# join against product inventory to get name
SELECT DISTINCT
  website.productSKU,
  website.total_ordered,
  inventory.name,
  inventory.stockLevel,
  inventory.restockingLeadTime,
  inventory.sentimentScore,
  inventory.sentimentMagnitude
FROM
  ecommerce.sales_by_sku_20170801 AS website
  LEFT JOIN `data-to-insights.ecommerce.products` AS inventory
  ON website.productSKU = inventory.SKU
ORDER BY total_ordered DESC
#standardSQL
# calculate ratio and filter
SELECT DISTINCT
  website.productSKU,
  website.total_ordered,
  inventory.name,
  inventory.stockLevel,
  inventory.restockingLeadTime,
  inventory.sentimentScore,
  inventory.sentimentMagnitude,

  SAFE_DIVIDE(website.total_ordered, inventory.stockLevel) AS ratio
FROM
  ecommerce.sales_by_sku_20170801 AS website
  LEFT JOIN `data-to-insights.ecommerce.products` AS inventory
  ON website.productSKU = inventory.SKU

# gone through more than 50% of inventory for the month
WHERE SAFE_DIVIDE(website.total_ordered,inventory.stockLevel) >= .50

ORDER BY total_ordered DESC
```

create new table

```bash
#standardSQL
CREATE OR REPLACE TABLE ecommerce.sales_by_sku_20170802
(
productSKU STRING,
total_ordered INT64
);
#standardSQL
SELECT * FROM ecommerce.sales_by_sku_20170801
UNION ALL
SELECT * FROM ecommerce.sales_by_sku_20170802
#standardSQL
SELECT * FROM `ecommerce.sales_by_sku_2017*`
WHERE _TABLE_SUFFIX = '0802'
```

## Creating Date-Partitioned Tables in BigQuery

```bash
#standardSQL
SELECT DISTINCT
  fullVisitorId,
  date,
  city,
  pageTitle
FROM `data-to-insights.ecommerce.all_sessions_raw`
WHERE date = '20170708'
LIMIT 5
```

Additionally, the LIMIT 5 does not reduce the total amount of data processed, which is a common misconception.

```bash
#standardSQL
CREATE OR REPLACE TABLE ecommerce.partition_by_day
PARTITION BY date_formatted
OPTIONS(
description="a table partitioned by date"
) AS

SELECT DISTINCT
PARSE_DATE("%Y%m%d", date) AS date_formatted,
fullvisitorId
FROM `data-to-insights.ecommerce.all_sessions_raw`
```

```bash
#standardSQL
SELECT *
FROM `data-to-insights.ecommerce.partition_by_day`
WHERE date_formatted = '2016-08-01'
```

```bash
#standardSQL
 SELECT
   DATE(CAST(year AS INT64), CAST(mo AS INT64), CAST(da AS INT64)) AS date,
   (SELECT ANY_VALUE(name) FROM `bigquery-public-data.noaa_gsod.stations` AS stations
    WHERE stations.usaf = stn) AS station_name,  -- Stations may have multiple names
   prcp
 FROM `bigquery-public-data.noaa_gsod.gsod*` AS weather
 WHERE prcp < 99.9  -- Filter unknown values
   AND prcp > 0      -- Filter stations/days with no precipitation
   AND CAST(_TABLE_SUFFIX AS int64) >= 2018
 ORDER BY date DESC -- Where has it rained/snowed recently
 LIMIT 10
```

```bash
#standardSQL
 CREATE OR REPLACE TABLE ecommerce.days_with_rain
 PARTITION BY date
 OPTIONS (
   partition_expiration_days=60,
   description="weather stations with precipitation, partitioned by day"
 ) AS


 SELECT
   DATE(CAST(year AS INT64), CAST(mo AS INT64), CAST(da AS INT64)) AS date,
   (SELECT ANY_VALUE(name) FROM `bigquery-public-data.noaa_gsod.stations` AS stations
    WHERE stations.usaf = stn) AS station_name,  -- Stations may have multiple names
   prcp
 FROM `bigquery-public-data.noaa_gsod.gsod*` AS weather
 WHERE prcp < 99.9  -- Filter unknown values
   AND prcp > 0      -- Filter
   AND CAST(_TABLE_SUFFIX AS int64) >= 2018
```

verify 60 days expired

```bash
#standardSQL
# avg monthly precipitation
SELECT
  AVG(prcp) AS average,
  station_name,
  date,
  CURRENT_DATE() AS today,
  DATE_DIFF(CURRENT_DATE(), date, DAY) AS partition_age,
  EXTRACT(MONTH FROM date) AS month
FROM ecommerce.days_with_rain
WHERE station_name = 'WAKAYAMA' #Japan
GROUP BY station_name, date, today, month, partition_age
ORDER BY date DESC; # most recent days first
```

## Troubleshooting and Solving Data Join Pitfalls

- Cross join: combines each row of the first dataset with each row of the second dataset, where every combination is represented in the output.
- Inner join: requires that key values exist in both tables for the records to appear in the results table. Records appear in the merge only if there are matches in both tables for the key values.
- Left join: Each row in the left table appears in the results, regardless of whether there are matches in the right table.
- Right join: the reverse of a left join. Each row in the right table appears in the results, regardless of whether there are matches in the left table.

```bash
#standardSQL
# how many products are on the website?
SELECT DISTINCT
productSKU,
v2ProductName
FROM `data-to-insights.ecommerce.all_sessions_raw`

SELECT
  v2ProductName,
  COUNT(DISTINCT productSKU) AS SKU_count,
  STRING_AGG(DISTINCT productSKU LIMIT 5) AS SKU
FROM `data-to-insights.ecommerce.all_sessions_raw`
  WHERE productSKU IS NOT NULL
  GROUP BY v2ProductName
  HAVING SKU_count > 1
  ORDER BY SKU_count DESC
# STRING_AGG -> ARRAY_AGG
WITH inventory_per_sku AS (
  SELECT DISTINCT
    website.v2ProductName,
    website.productSKU,
    inventory.stockLevel
  FROM `data-to-insights.ecommerce.all_sessions_raw` AS website
  JOIN `data-to-insights.ecommerce.products` AS inventory
    ON website.productSKU = inventory.SKU
    WHERE productSKU = 'GGOEGPJC019099'
)

SELECT
  productSKU,
  SUM(stockLevel) AS total_inventory
FROM inventory_per_sku
GROUP BY productSKU
#standardSQL
SELECT DISTINCT
productSKU,
v2ProductCategory,
discount
FROM `data-to-insights.ecommerce.all_sessions_raw` AS website
CROSS JOIN ecommerce.site_wide_promotion
WHERE v2ProductCategory LIKE '%Clearance%'
AND productSKU = 'GGOEGOLC013299'
```

LEFT JOIN + RIGHT JOIN = FULL JOIN

## Working with JSON, Arrays, and Structs in BigQuery

```bash
#standardSQL
SELECT
['raspberry', 'blackberry', 'strawberry', 'cherry'] AS fruit_array
#standardSQL
SELECT person, fruit_array, total_cost FROM `data-to-insights.advanced.fruit_store`;
```

### Loading semi-structured JSON into BigQuery

Create a new table fruit_details in the dataset.

Add the following details for the table:

- Source: Choose Cloud Storage in the Create table from dropdown.
- Select file from Cloud Storage bucket: gs://cloud-training/gsp416/shopping_cart.json
- File format: JSONL (Newline delimited JSON)

```bash
SELECT
  fullVisitorId,
  date,
  ARRAY_AGG(v2ProductName) AS products_viewed,
  ARRAY_LENGTH(ARRAY_AGG(v2ProductName)) AS num_products_viewed,
  ARRAY_AGG(pageTitle) AS pages_viewed,
  ARRAY_LENGTH(ARRAY_AGG(pageTitle)) AS num_pages_viewed
  FROM `data-to-insights.ecommerce.all_sessions`
WHERE visitId = 1501570398
GROUP BY fullVisitorId, date
ORDER BY date
SELECT
  fullVisitorId,
  date,
  ARRAY_AGG(DISTINCT v2ProductName) AS products_viewed,
  ARRAY_LENGTH(ARRAY_AGG(DISTINCT v2ProductName)) AS distinct_products_viewed,
  ARRAY_AGG(DISTINCT pageTitle) AS pages_viewed,
  ARRAY_LENGTH(ARRAY_AGG(DISTINCT pageTitle)) AS distinct_pages_viewed
  FROM `data-to-insights.ecommerce.all_sessions`
WHERE visitId = 1501570398
GROUP BY fullVisitorId, date
ORDER BY date
```

### Querying datasets that already have ARRAYs

```bash
SELECT
  visitId,
  hits.page.pageTitle
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`
WHERE visitId = 1501570398
-->
SELECT DISTINCT
  visitId,
  h.page.pageTitle
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`,
UNNEST(hits) AS h
WHERE visitId = 1501570398
LIMIT 10
```

### Let's explore a dataset with STRUCTs

https://console.cloud.google.com/bigquery?p=bigquery-public-data&d=google_analytics_sample&t=ga_sessions_20170801&page=table

type: struct -> record
mode: array -> repeated

### Practice with STRUCTs and ARRAYs

```bash
#standardSQL
SELECT race, participants.name
FROM racing.race_results
CROSS JOIN
race_results.participants # full STRUCT name
```

```bash
#standardSQL
SELECT COUNT(p.name) AS racer_count
FROM racing.race_results AS r, UNNEST(r.participants) AS p
```

```bash
#standardSQL
SELECT
  p.name,
  SUM(split_times) as total_race_time
FROM racing.race_results AS r
, UNNEST(r.participants) AS p
, UNNEST(p.splits) AS split_times
WHERE p.name LIKE 'R%'
GROUP BY p.name
ORDER BY total_race_time ASC;
```

```bash
#standardSQL
SELECT
  p.name,
  split_time
FROM racing.race_results AS r
, UNNEST(r.participants) AS p
, UNNEST(p.splits) AS split_time
WHERE split_time = 23.2;
```

## Build and Execute MySQL, PostgreSQL, and SQLServer to Data Catalog Connectors

Data Catalog is a fully managed and scalable metadata management service that empowers organizations to quickly discover, understand, and manage all their data.

It offers a simple and easy-to-use search interface for data discovery, a flexible and powerful cataloging system for capturing both technical and business metadata, and a strong security and compliance foundation with Cloud Data Loss Prevention (DLP) and Cloud Identity and Access Management (IAM) integrations.

### Create the SQLServer Database

```bash
gcloud config set project qwiklabs-gcp-01-020f046aef10
git clone https://github.com/mesmacosta/cloudsql-sqlserver-tooling
cd cloudsql-sqlserver-tooling
source init-db.sh
...
CREATE TABLE factory_warehouse15797.employees53b82dc5 ( school80581 REAL, reason91250 DATETIME, randomdata32431 BINARY, phone_number52754 REAL, person66471 REAL, credit_card75527 DATETIME )

COMPLETED
...
export PROJECT_ID=qwiklabs-gcp-01-020f046aef10
gcloud iam service-accounts create sqlserver2dc-credentials \
    --display-name  "Service Account for SQLServer to Data Catalog connector" \
    --project $PROJECT_ID
gcloud iam service-accounts keys create "sqlserver2dc-credentials.json" \
    --iam-account "sqlserver2dc-credentials@$PROJECT_ID.iam.gserviceaccount.com"
gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member "serviceAccount:sqlserver2dc-credentials@$PROJECT_ID.iam.gserviceaccount.com" \
    --quiet \
    --project $PROJECT_ID \
    --role "roles/datacatalog.admin"
# Execute SQLServer to Data Catalog connector.
docker run --rm --tty -v \
    "$PWD":/data mesmacosta/sqlserver2datacatalog:stable \
    --datacatalog-project-id=$PROJECT_ID \
    --datacatalog-location-id=us-central1 \
    --sqlserver-host=$public_ip_address \
    --sqlserver-user=$username \
    --sqlserver-pass=$password \
    --sqlserver-database=$database
...
============End sqlserver-to-datacatalog============
...
./cleanup-db.sh
docker run --rm --tty -v \
    "$PWD":/data mesmacosta/sqlserver-datacatalog-cleaner:stable \
    --datacatalog-project-ids=$PROJECT_ID \
    --rdbms-type=sqlserver \
    --table-container-type=schema
./delete-db.sh
...
 Cloud SQL Instance deleted
 COMPLETED
```

### PostgreSQL to Data Catalog

```bash
cd
git clone https://github.com/mesmacosta/cloudsql-postgresql-tooling
cd cloudsql-postgresql-tooling
source init-db.sh
...
CREATE TABLE factory_warehouse69945.home17e97c57 ( house57588 DATE, paragraph64180 SMALLINT, ip_address61569 JSONB, date_time44962 REAL, food19478 JSONB, state8925 VARCHAR(25), cpf75444 REAL, date_time96090 SMALLINT, reason7955 CHAR(5), phone_number96292 INT, size97593 DATE, date_time609 CHAR(5), location70431 DATE )
 COMPLETED
...
gcloud iam service-accounts create postgresql2dc-credentials \
    --display-name  "Service Account for PostgreSQL to Data Catalog connector" \
    --project $PROJECT_ID
gcloud iam service-accounts keys create "postgresql2dc-credentials.json" \
    --iam-account "postgresql2dc-credentials@$PROJECT_ID.iam.gserviceaccount.com"
gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member "serviceAccount:postgresql2dc-credentials@$PROJECT_ID.iam.gserviceaccount.com" \
    --quiet \
    --project $PROJECT_ID \
    --role "roles/datacatalog.admin"
docker run --rm --tty -v \
    "$PWD":/data mesmacosta/postgresql2datacatalog:stable \
    --datacatalog-project-id=$PROJECT_ID \
    --datacatalog-location-id=us-central1 \
    --postgresql-host=$public_ip_address \
    --postgresql-user=$username \
    --postgresql-pass=$password \
    --postgresql-database=$database
...
============End postgresql-to-datacatalog============
...
./cleanup-db.sh
docker run --rm --tty -v \
    "$PWD":/data mesmacosta/postgresql-datacatalog-cleaner:stable \
    --datacatalog-project-ids=$PROJECT_ID \
    --rdbms-type=postgresql \
    --table-container-type=schema
./delete-db.sh
```

### MySQL to Data Catalog

skip

## Build and Optimize Data Warehouses with BigQuery: Challenge Lab

Topics tested

- Use BigQuery to access public COVID and other demographic datasets.
- Create a new BigQuery dataset which will store your tables.
- Add a new date partitioned table to your dataset.
- Add new columns to this table with appropriate data types.
- Run a series of JOINS to populate these new columns with data drawn from other tables.

https://console.cloud.google.com/bigquery?p=bigquery-public-data&d=covid19_govt_response&page=dataset

- `oxford_policy_tracker` table in your newly created dataset, with an expiry time set to 90 days.
- You have also been instructed to exclude the United Kingdom ( alpha_3_code='GBR') & the United States of America (alpha_3_code='USA) as these will be subject to more in-depth analysis through nation and state specific analysis.
- `mobility_report` table from the Google COVID 19 Mobility public dataset, https://console.cloud.google.com/bigquery?p=bigquery-public-data&d=covid19_govt_response&page=dataset
- `covid_19_geographic_distribution_worldwide` table from the European Center for Disease Control COVID 19 public dataset https://console.cloud.google.com/bigquery?p=bigquery-public-data&d=covid19_ecdc&page=dataset

```bash
UPDATE
    `covid.oxford` t0
SET
    ecdc_new_cases = t2.daily_confirmed_cases
FROM
    covid.oxford t1
    LEFT JOIN
    (SELECT DISTINCT country_territory_code, daily_confirmed_cases FROM `bigquery-public-data.covid19_ecdc.covid_19_geographic_distribution_worldwide`) AS t2
     ON t1.alpha_3_code = t2.country_territory_code
WHERE CONCAT(t0.country_name, t0.date) = CONCAT(t1.country_name, t1.date);
```

The above template updates a daily new case column so you must modify it before you can use it to populate the population data from the European Center for Disease Control COVID 19 public dataset

```text
New Column Name          SQL Data Type
population               INTEGER
country_area             FLOAT
mobility                 RECORD
mobility.avg_retail      FLOAT
mobility.avg_grocery     FLOAT
mobility.avg_parks       FLOAT
mobility.avg_transit     FLOAT
mobility.avg_workplace   FLOAT
mobility.avg_residential FLOAT
```

```bash
 SELECT country_region, date,
      AVG(retail_and_recreation_percent_change_from_baseline) as avg_retail,
      AVG(grocery_and_pharmacy_percent_change_from_baseline)  as avg_grocery,
      AVG(parks_percent_change_from_baseline) as avg_parks,
      AVG(transit_stations_percent_change_from_baseline) as avg_transit,
      AVG( workplaces_percent_change_from_baseline ) as avg_workplace,
      AVG( residential_percent_change_from_baseline)  as avg_residential
      FROM `bigquery-public-data.covid19_google_mobility.mobility_report`
      GROUP BY country_region, date
```

### Task 1: Create a table partitioned by date

```bash
CREATE OR REPLACE TABLE mycovid.partition_by_day
PARTITION BY date
OPTIONS(
    partition_expiration_days=90,
description="a table partitioned by date"
) AS

SELECT *
FROM `bigquery-public-data.covid19_govt_response.oxford_policy_tracker`
WHERE alpha_3_code NOT IN ("GBR", "USA")
```

### Task 2: Add new columns to your table

```bash
[
    {
        "name": "population",
        "type": "INTEGER"
    },
    {
        "name": "country_area",
        "type": "FLOAT"
    },
    {
        "name": "mobility",
        "type": "RECORD",
        "fields": [
            {
                "name": "avg_retail",
                "type": "FLOAT"
            },
            {
                "name": "avg_grocery",
                "type": "FLOAT"
            },
            {
                "name": "avg_parks",
                "type": "FLOAT"
            },
            {
                "name": "avg_transit",
                "type": "FLOAT"
            },
            {
                "name": "avg_workplace",
                "type": "FLOAT"
            },
            {
                "name": "avg_residential",
                "type": "FLOAT"
            }
        ]
    }
]
```

### Task 3: Add country population data to the population column

```bash
UPDATE `qwiklabs-gcp-00-acf08aca0012.mycovid.partition_by_day` as t1
SET population = (select pop_data_2019 FROM `bigquery-public-data.covid19_ecdc.covid_19_geographic_distribution_worldwide` as t2 where t1.alpha_3_code = t2.country_territory_code AND t1.date = t2.date)
WHERE TRUE
```

### Task 4: Add country area data to the country_area column

```bash
UPDATE `qwiklabs-gcp-00-acf08aca0012.mycovid.partition_by_day` as t1
SET country_area = (select country_area FROM `bigquery-public-data.census_bureau_international.country_names_area` as t2 where t1.country_name = t2.country_name)
WHERE TRUE
```

### Task 5: Populate the mobility record data

SELECT country_region, date,
AVG(retail_and_recreation_percent_change_from_baseline) as avg_retail,
AVG(grocery_and_pharmacy_percent_change_from_baseline) as avg_grocery,
AVG(parks_percent_change_from_baseline) as avg_parks,
AVG(transit_stations_percent_change_from_baseline) as avg_transit,
AVG( workplaces_percent_change_from_baseline ) as avg_workplace,
AVG( residential_percent_change_from_baseline) as avg_residential
FROM `bigquery-public-data.covid19_google_mobility.mobility_report`
where country_region = "Brazil"
GROUP BY country_region, date
order by country_region

```bash
UPDATE `qwiklabs-gcp-00-acf08aca0012.mycovid.partition_by_day` as t1
SET
    mobility.avg_retail = (
        SELECT AVG(retail_and_recreation_percent_change_from_baseline)
        FROM `bigquery-public-data.covid19_google_mobility.mobility_report` as t2
        where t1.country_name = t2.country_region AND t1.date = t2.date
    ),
    mobility.avg_grocery = (
        SELECT AVG(grocery_and_pharmacy_percent_change_from_baseline)
        FROM `bigquery-public-data.covid19_google_mobility.mobility_report` as t2
        where t1.country_name = t2.country_region AND t1.date = t2.date
    ),
    mobility.avg_parks = (
        SELECT AVG(parks_percent_change_from_baseline)
        FROM `bigquery-public-data.covid19_google_mobility.mobility_report` as t2
        where t1.country_name = t2.country_region AND t1.date = t2.date
    ),
    mobility.avg_transit = (
        SELECT AVG(transit_stations_percent_change_from_baseline)
        FROM `bigquery-public-data.covid19_google_mobility.mobility_report` as t2
        where t1.country_name = t2.country_region AND t1.date = t2.date
    ),
    mobility.avg_workplace = (
        SELECT AVG(workplaces_percent_change_from_baseline)
        FROM `bigquery-public-data.covid19_google_mobility.mobility_report` as t2
        where t1.country_name = t2.country_region AND t1.date = t2.date
    ),
    mobility.avg_residential = (
        SELECT AVG(residential_percent_change_from_baseline)
        FROM `bigquery-public-data.covid19_google_mobility.mobility_report` as t2
        where t1.country_name = t2.country_region AND t1.date = t2.date
    )
WHERE TRUE
```

### Task 6: Query missing data in population & country_area columns

Run a query to find the missing countries in the population and country_area data. The query should list countries that do not have any population data and countries that do not have country area information, ordered by country name. If a country has neither population or country area it must appear twice.

```bash
WITH table1 AS
    (SELECT distinct(country_name) FROM `qwiklabs-gcp-00-acf08aca0012.mycovid.partition_by_day` WHERE population is NULL UNION ALL
     SELECT distinct(country_name) FROM `qwiklabs-gcp-00-acf08aca0012.mycovid.partition_by_day` WHERE country_area is NULL)
SELECT country_name FROM table1 ORDER BY country_name
```
