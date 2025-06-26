SQL ANALYSIS


1.	Retrieve the jurisdiction residence with the highest number of COVID deaths for the latest data period end date:

SELECT TOP 1 
    Jurisdiction_Residence, 
    COVID_deaths
FROM DA_Data
WHERE data_period_end = (
    SELECT MAX(data_period_end) FROM DA_Data
)
ORDER BY COVID_deaths DESC;


2.	Retrieve the top 5 jurisdictions with the highest % difference in aa_COVID_rate compared to the overall crude COVID rate:

SELECT TOP 5 
    Jurisdiction_Residence,
    aa_COVID_rate,
    crude_COVID_rate,
    ROUND(((aa_COVID_rate - crude_COVID_rate) / crude_COVID_rate) * 100, 2) AS pct_diff
FROM DA_Data
WHERE data_period_end = (
    SELECT MAX(data_period_end) FROM DA_Data
)
ORDER BY pct_diff DESC;


3.	Calculate the average COVID deaths per week for each jurisdiction residence and group, for the latest 4 data period end dates:

WITH LatestPeriods AS (
    SELECT DISTINCT TOP 4 data_period_end
    FROM DA_Data
    ORDER BY data_period_end DESC
)
SELECT 
    Jurisdiction_Residence,
    [Group],
    ROUND(AVG(CAST(COVID_deaths AS FLOAT)), 2) AS avg_weekly_deaths
FROM DA_Data
WHERE data_period_end IN (SELECT data_period_end FROM LatestPeriods)
GROUP BY Jurisdiction_Residence, [Group];

4.	Retrieve the data for the latest data period end date, but exclude any jurisdictions with zero deaths and missing values:

SELECT *
FROM DA_Data
WHERE data_period_end = (
    SELECT MAX(data_period_end) FROM DA_Data
)
AND COVID_deaths > 0
AND Jurisdiction_Residence IS NOT NULL
AND aa_COVID_rate IS NOT NULL
AND crude_COVID_rate IS NOT NULL
AND COVID_pct_of_total IS NOT NULL;


5.	Calculate the week-over-week percentage change in COVID_pct_of_total for all jurisdictions and groups after March 1, 2020:

WITH RankedData AS (
    SELECT *,
           LAG(COVID_pct_of_total) OVER (
               PARTITION BY Jurisdiction_Residence, [Group] 
               ORDER BY data_period_start
           ) AS Prev_pct
    FROM DA_Data
    WHERE data_period_start > '2020-03-01'
)
SELECT 
    Jurisdiction_Residence,
    [Group],
    data_period_start,
    COVID_pct_of_total,
    Prev_pct,
    ROUND(
        CASE 
            WHEN Prev_pct IS NOT NULL AND Prev_pct != 0 THEN 
                ((COVID_pct_of_total - Prev_pct) / Prev_pct) * 100
            ELSE NULL 
        END, 2
    ) AS pct_change_week
FROM RankedData;
