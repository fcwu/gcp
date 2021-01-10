# BigQuery Basics for Data Analysts

## Introduction to SQL for BigQuery and Cloud SQL

## BigQuery: Qwik Start - Console

## Troubleshooting Common SQL Errors with BigQuery

```bash
#standardSQL
SELECT
geoNetwork_city,
SUM(totals_transactions) AS total_products_ordered,
COUNT( DISTINCT fullVisitorId) AS distinct_visitors,
SUM(totals_transactions) / COUNT( DISTINCT fullVisitorId) AS avg_products_ordered
FROM
`data-to-insights.ecommerce.rev_transactions`
GROUP BY geoNetwork_city
HAVING avg_products_ordered > 20
ORDER BY avg_products_ordered DESC
```

## Big Data Analysis to a Slide Presentation

Apps Script

http://script.google.com/

### Query BigQuery and log results to Sheet

```javascript
/**
 * Copyright 2018 Google LLC
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at apache.org/licenses/LICENSE-2.0.
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

// Filename for data results
var QUERY_NAME = "Most common words in all of Shakespeare's works";
// Replace this value with your Google Cloud API project ID
var PROJECT_ID = '<YOUR_PROJECT_ID>';
if (!PROJECT_ID) throw Error('Project ID is required in setup');

/**
 * Runs a BigQuery query; puts results into Sheet. You must enable
 * the BigQuery advanced service before you can run this code.
 * @see http://developers.google.com/apps-script/advanced/bigquery#run_query
 * @see http://github.com/gsuitedevs/apps-script-samples/blob/master/advanced/bigquery.gs
 *
 * @returns {Spreadsheet} Returns a spreadsheet with BigQuery results
 * @see http://developers.google.com/apps-script/reference/spreadsheet/spreadsheet
 */
function runQuery() {
  // Replace sample with your own BigQuery query.
  var request = {
    query:
        'SELECT ' +
            'LOWER(word) AS word, ' +
            'SUM(word_count) AS count ' +
        'FROM [bigquery-public-data:samples.shakespeare] ' +
        'GROUP BY word ' +
        'ORDER BY count ' +
        'DESC LIMIT 10'
  };
  var queryResults = BigQuery.Jobs.query(request, PROJECT_ID);
  var jobId = queryResults.jobReference.jobId;

  // Wait for BQ job completion (with exponential backoff).
  var sleepTimeMs = 500;
  while (!queryResults.jobComplete) {
    Utilities.sleep(sleepTimeMs);
    sleepTimeMs *= 2;
    queryResults = BigQuery.Jobs.getQueryResults(PROJECT_ID, jobId);
  }

  // Get all results from BigQuery.
  var rows = queryResults.rows;
  while (queryResults.pageToken) {
    queryResults = BigQuery.Jobs.getQueryResults(PROJECT_ID, jobId, {
      pageToken: queryResults.pageToken
    });
    rows = rows.concat(queryResults.rows);
  }

  // Return null if no data returned.
  if (!rows) {
    return Logger.log('No rows returned.');
  }

  // Create the new results spreadsheet.
  var spreadsheet = SpreadsheetApp.create(QUERY_NAME);
  var sheet = spreadsheet.getActiveSheet();

  // Add headers to Sheet.
  var headers = queryResults.schema.fields.map(function(field) {
    return field.name.toUpperCase();
  });
  sheet.appendRow(headers);

  // Append the results.
  var data = new Array(rows.length);
  for (var i = 0; i < rows.length; i++) {
    var cols = rows[i].f;
    data[i] = new Array(cols.length);
    for (var j = 0; j < cols.length; j++) {
      data[i][j] = cols[j].v;
    }
  }

  // Start storing data in row 2, col 1
  var START_ROW = 2;      // skip header row
  var START_COL = 1;
  sheet.getRange(START_ROW, START_COL, rows.length, headers.length).setValues(data);

  Logger.log('Results spreadsheet created: %s', spreadsheet.getUrl());
}
```

### Create a chart in Google Sheets

```javascript
/**
 * Uses spreadsheet data to create columnar chart.
 * @param {Spreadsheet} Spreadsheet containing results data
 * @returns {EmbeddedChart} visualizing the results
 * @see http://developers.google.com/apps-script/reference/spreadsheet/embedded-chart
 */
function createColumnChart(spreadsheet) {
  // Retrieve the populated (first and only) Sheet.
  var sheet = spreadsheet.getSheets()[0];
  // Data range in Sheet is from cell A2 to B11
  var START_CELL = 'A2';  // skip header row
  var END_CELL = 'B11';
  // Place chart on Sheet starting on cell E5.
  var START_ROW = 5;      // row 5
  var START_COL = 5;      // col E
  var OFFSET = 0;

  // Create & place chart on the Sheet using above params.
  var chart = sheet.newChart()
     .setChartType(Charts.ChartType.COLUMN)
     .addRange(sheet.getRange(START_CELL + ':' + END_CELL))
     .setPosition(START_ROW, START_COL, OFFSET, OFFSET)
     .build();
  sheet.insertChart(chart);
}
```

### Put the results data into a slide deck

```javascript
// Filename for data results
var QUERY_NAME = "Most common words in all of Shakespeare's works";
// Replace this value with your Google Cloud API project ID
var PROJECT_ID = '<YOUR_PROJECT_ID>';
if (!PROJECT_ID) throw Error('Project ID is required in setup');

/**
 * Runs a BigQuery query; puts results into Sheet. You must enable
 * the BigQuery advanced service before you can run this code.
 * @see http://developers.google.com/apps-script/advanced/bigquery#run_query
 * @see http://github.com/gsuitedevs/apps-script-samples/blob/master/advanced/bigquery.gs
 *
 * @returns {Spreadsheet} Returns a spreadsheet with BigQuery results
 * @see http://developers.google.com/apps-script/reference/spreadsheet/spreadsheet
 */
function runQuery() {
  // Replace sample with your own BigQuery query.
  var request = {
    query:
        'SELECT ' +
            'LOWER(word) AS word, ' +
            'SUM(word_count) AS count ' +
        'FROM [bigquery-public-data:samples.shakespeare] ' +
        'GROUP BY word ' +
        'ORDER BY count ' +
        'DESC LIMIT 10'
  };
  var queryResults = BigQuery.Jobs.query(request, PROJECT_ID);
  var jobId = queryResults.jobReference.jobId;

  // Wait for BQ job completion (with exponential backoff).
  var sleepTimeMs = 500;
  while (!queryResults.jobComplete) {
    Utilities.sleep(sleepTimeMs);
    sleepTimeMs *= 2;
    queryResults = BigQuery.Jobs.getQueryResults(PROJECT_ID, jobId);
  }

  // Get all results from BigQuery.
  var rows = queryResults.rows;
  while (queryResults.pageToken) {
    queryResults = BigQuery.Jobs.getQueryResults(PROJECT_ID, jobId, {
      pageToken: queryResults.pageToken
    });
    rows = rows.concat(queryResults.rows);
  }

  // Return null if no data returned.
  if (!rows) {
    return Logger.log('No rows returned.');
  }

  // Create the new results spreadsheet.
  var spreadsheet = SpreadsheetApp.create(QUERY_NAME);
  var sheet = spreadsheet.getActiveSheet();

  // Add headers to Sheet.
  var headers = queryResults.schema.fields.map(function(field) {
    return field.name.toUpperCase();
  });
  sheet.appendRow(headers);

  // Append the results.
  var data = new Array(rows.length);
  for (var i = 0; i < rows.length; i++) {
    var cols = rows[i].f;
    data[i] = new Array(cols.length);
    for (var j = 0; j < cols.length; j++) {
      data[i][j] = cols[j].v;
    }
  }

  // Start storing data in row 2, col 1
  var START_ROW = 2;      // skip header row
  var START_COL = 1;
  sheet.getRange(START_ROW, START_COL, rows.length, headers.length).setValues(data);

  // Return the spreadsheet object for later use.
  return spreadsheet;
}

/**
 * Uses spreadsheet data to create columnar chart.
 * @param {Spreadsheet} Spreadsheet containing results data
 * @returns {EmbeddedChart} visualizing the results
 * @see http://developers.google.com/apps-script/reference/spreadsheet/embedded-chart
 */
function createColumnChart(spreadsheet) {
  // Retrieve the populated (first and only) Sheet.
  var sheet = spreadsheet.getSheets()[0];
  // Data range in Sheet is from cell A2 to B11
  var START_CELL = 'A2';  // skip header row
  var END_CELL = 'B11';
  // Place chart on Sheet starting on cell E5.
  var START_ROW = 5;      // row 5
  var START_COL = 5;      // col E
  var OFFSET = 0;

  // Create & place chart on the Sheet using above params.
  var chart = sheet.newChart()
     .setChartType(Charts.ChartType.COLUMN)
     .addRange(sheet.getRange(START_CELL + ':' + END_CELL))
     .setPosition(START_ROW, START_COL, OFFSET, OFFSET)
     .build();
  sheet.insertChart(chart);

  // Return the chart object for later use.
  return chart;
}

/**
 * Create presentation with spreadsheet data & chart
 * @param {Spreadsheet} Spreadsheet with results data
 * @param {EmbeddedChart} Sheets chart to embed on slide
 * @returns {Presentation} Returns a slide deck with results
 * @see http://developers.google.com/apps-script/reference/slides/presentation
 */
function createSlidePresentation(spreadsheet, chart) {
  // Create the new presentation.
  var deck = SlidesApp.create(QUERY_NAME);

  // Populate the title slide.
  var [title, subtitle] = deck.getSlides()[0].getPageElements();
  title.asShape().getText().setText(QUERY_NAME);
  subtitle.asShape().getText().setText('via GCP and G Suite APIs:\n' +
    'Google Apps Script, BigQuery, Sheets, Slides');

  // Data range to copy is from cell A1 to B11
  var START_CELL = 'A1';  // include header row
  var END_CELL = 'B11';
  // Add the table slide and insert an empty table on it of
  // the dimensions of the data range; fails if Sheet empty.
  var tableSlide = deck.appendSlide(SlidesApp.PredefinedLayout.BLANK);
  var sheetValues = spreadsheet.getSheets()[0].getRange(
      START_CELL + ':' + END_CELL).getValues();
  var table = tableSlide.insertTable(sheetValues.length, sheetValues[0].length);

  // Populate the table with spreadsheet data.
  for (var i = 0; i < sheetValues.length; i++) {
    for (var j = 0; j < sheetValues[0].length; j++) {
      table.getCell(i, j).getText().setText(String(sheetValues[i][j]));
    }
  }

  // Add a chart slide and insert the chart on it.
  var chartSlide = deck.appendSlide(SlidesApp.PredefinedLayout.BLANK);
  chartSlide.insertSheetsChart(chart);

  // Return the presentation object for later use.
  return deck;
}

/**
 * Runs a BigQuery query, adds data and a chart in a Sheet,
 * and adds the data and chart to a new slide presentation.
 */
function createBigQueryPresentation() {
  var spreadsheet = runQuery();
  Logger.log('Results spreadsheet created: %s', spreadsheet.getUrl());
  var chart = createColumnChart(spreadsheet);
  var deck = createSlidePresentation(spreadsheet, chart);
  Logger.log('Results slide deck created: %s', deck.getUrl());
}
```

## Explore and Create Reports with Data Studio

https://datastudio.google.com/

## Insights from Data with BigQuery: Challenge Lab 

### Query 1: Total Confirmed Cases

```bash
SELECT country_code, subregion1_code, subregion2_code, cumulative_confirmed
FROM `bigquery-public-data.covid19_open_data.covid19_open_data` 
WHERE date = "2020-04-15"
SELECT SUM(cumulative_confirmed) as total_cases_worldwide
FROM `bigquery-public-data.covid19_open_data.covid19_open_data` 
WHERE date = "2020-04-15"
```

### Query 2: Worst Affected Areas

```bash
SELECT
    COUNT(*) AS count_of_states
FROM (
SELECT
    subregion1_name AS state,
    SUM(cumulative_deceased) AS death_count
FROM
  `bigquery-public-data.covid19_open_data.covid19_open_data`
WHERE
  country_name="United States of America"
  AND date='2020-04-10'
  AND subregion1_name IS NOT NULL
GROUP BY
  subregion1_name
)
WHERE death_count > 100
```

### Query 3: Identifying Hotspots

```bash
SELECT * FROM (
SELECT subregion1_name as state, sum(cumulative_confirmed) as total_confirmed_cases
FROM `bigquery-public-data.covid19_open_data.covid19_open_data`
WHERE country_code="US" AND date='2020-04-10' AND subregion1_name is NOT NULL
GROUP BY subregion1_name
ORDER BY total_confirmed_cases DESC ) WHERE total_confirmed_cases > 1000
```

### Query 4: Fatality Ratio

```bash
SELECT 
  SUM(cumulative_confirmed) AS total_confirmed_cases, 
  SUM(cumulative_deceased) AS total_deaths,
  SUM(cumulative_deceased) / SUM(cumulative_confirmed) * 100 AS case_fatality_ratio
FROM `bigquery-public-data.covid19_open_data.covid19_open_data` 
WHERE country_name = "Italy" AND date BETWEEN "2020-04-01" AND "2020-04-30"
```

### Query 5: Identifying specific day

```bash
SELECT 
  date
FROM `bigquery-public-data.covid19_open_data.covid19_open_data` 
WHERE country_name = "Italy" AND cumulative_deceased > 10000
ORDER BY date
LIMIT 1
```

### Query 6: Finding days with zero net new cases

https://docs.microsoft.com/zh-tw/sql/t-sql/functions/lag-transact-sql?view=sql-server-ver15

```bash
WITH india_cases_by_date AS (
  SELECT
    date,
    SUM(cumulative_confirmed) AS cases
  FROM
    `bigquery-public-data.covid19_open_data.covid19_open_data`
  WHERE
    country_name="India"
    AND date between '2020-02-21' and '2020-03-15'
  GROUP BY
    date
  ORDER BY
    date ASC
 )

, india_previous_day_comparison AS
(SELECT
  date,
  cases,
  LAG(cases) OVER(ORDER BY date) AS previous_day,
  cases - LAG(cases) OVER(ORDER BY date) AS net_new_cases
FROM india_cases_by_date
)

SELECT COUNT(*) FROM india_previous_day_comparison WHERE net_new_cases = 0
```

### Query 7: Doubling rate

```bash
WITH us_cases_by_date AS (
  SELECT
    date,
    SUM( cumulative_confirmed ) AS cases
  FROM
    `bigquery-public-data.covid19_open_data.covid19_open_data`
  WHERE
    country_name="United States of America"
    AND date between '2020-03-22' and '2020-04-20'
  GROUP BY
    date
  ORDER BY
    date ASC
 )

, us_previous_day_comparison AS
(SELECT
  date,
  cases,
  LAG(cases) OVER(ORDER BY date) AS previous_day,
  cases - LAG(cases) OVER(ORDER BY date) AS net_new_cases,
  (cases - LAG(cases) OVER(ORDER BY date))*100/LAG(cases) OVER(ORDER BY date) AS percentage_increase
FROM us_cases_by_date
)
SELECT
  Date,
  cases AS Confirmed_Cases_On_Day,
  previous_day AS Confirmed_Cases_Previous_Day,
  percentage_increase AS Percentage_Increase_In_Cases
FROM
  us_previous_day_comparison
WHERE
  percentage_increase > 10
```

### Query 8: Recovery rate

```bash
WITH cases_by_country AS (
  SELECT
    country_name AS country,
    SUM(cumulative_confirmed) AS cases,
    SUM(cumulative_recovered) AS recovered_cases
  FROM
    `bigquery-public-data.covid19_open_data.covid19_open_data`
  WHERE
    date="2020-05-10"
  GROUP BY
    country_name
)

, recovered_rate AS (
  SELECT
    country, cases, recovered_cases,
    (recovered_cases * 100)/cases AS recovery_rate
  FROM
    cases_by_country
)

SELECT country, cases AS confirmed_cases, recovered_cases, recovery_rate
FROM
   recovered_rate
WHERE
   cases > 50000
ORDER BY recovery_rate DESC
LIMIT 10
```

### Query 9: CDGR - Cumulative Daily Growth Rate

- LEAD: https://cloud.google.com/bigquery/docs/reference/standard-sql/functions-and-operators
- https://tivo168.pixnet.net/blog/post/266556215
- LAG OVER: https://cloud.google.com/bigquery/docs/reference/standard-sql/navigation_functions

```bash
WITH
  france_cases AS (
  SELECT
    date,
    SUM(cumulative_confirmed) AS total_cases
  FROM
    `bigquery-public-data.covid19_open_data.covid19_open_data`
  WHERE
    country_name="France"
    AND date IN ('2020-01-24',
      '2020-05-10')
  GROUP BY
    date
  ORDER BY
    date)
, summary as (
SELECT
  total_cases AS first_day_cases,
  LEAD(total_cases) OVER(ORDER BY date) AS last_day_cases,
  DATE_DIFF(LEAD(date) OVER(ORDER BY date),date, day) AS days_diff
FROM
  france_cases
LIMIT 1
)

select first_day_cases, last_day_cases, days_diff, POW((last_day_cases/first_day_cases),(1/days_diff))-1 as cdgr
from summary

```

### Create a Datastudio report

```bash
SELECT
  date, SUM(cumulative_confirmed) AS country_cases,
  SUM(cumulative_deceased) AS country_deaths
FROM
  `bigquery-public-data.covid19_open_data.covid19_open_data`
WHERE
  date BETWEEN '2020-03-15'
  AND '2020-04-30'
  AND country_name='United States of America'
GROUP BY date
```

1. ADD above query
2. Click on EXPLORE DATA > Explore with Data Studio.
3. Give access to Data Studio and authorize it to control BigQuery.
    If you fail to create a report for the very first time login of Data Studio, click + Blank Report option and accept the Terms of Service. Then, go back again to BigQuery page and click Explore with Data Studio again.
4. Create a new Time series chart in the new Data Studio report by selecting Add a chart > Time series Chart.
5. Add country_cases and country_deaths to the Metric field.
6. Click Save to commit the change.
