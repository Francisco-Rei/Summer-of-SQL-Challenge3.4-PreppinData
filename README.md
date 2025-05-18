# Preppin Data 2023 Week 3.4
This is a challenge from [Summer of SQL](https://github.com/wjsutton/the_summer_of_sql) to help gain skills and confidence in SQL.

Skills tested in challenge 3.4:
- Common Table Expressions (CTEs)
- Union All
- Pivot
- -Using ROW_NUMBER for deduplication


## Challenge: Preppin Data 2023 Week 3.4

Data Source Bank acquires new customers every month. They are stored in separate tabs of an Excel workbook so it's "easy" to see which customers joined in which month. However, it's not so easy to do any comparisons between months. Therefore, we'd like to consolidate all the months into one dataset. There's an extra twist as well. The customer demographics are stored as rows rather than columns, which doesn't make for very easy reading. So we'd also like to restructure the data.

Inputs
- One excel file, with 12 different tabs, one for each month:

![Input](https://blogger.googleusercontent.com/img/a/AVvXsEiaT8Ijorc_YYWPq3od7zn-iyjOm2qzEdvgd0QHLYL7VTuJc5kiNM-SnT3da40PRBJYbNSK1pZs2mp13ieNm2gLXxg1QGI4QXYSYI0FepBFkEl-k25mBRALEfzecQkU16UcUnnWhjFPlg5ZJn0vUFzC9oXTAMN2MzDR6JC4Rcxjx2CNHnJ2kF4U0fQOEQ).

Requirements:

- We want to stack the tables on top of one another, since they have the same fields in each sheet
- Some of the fields aren't matching up as we'd expect, due to differences in spelling. Merge these fields together
- Make a Joining Date field based on the Joining Day, Table Names and the year 2023
- Reshape our data so we have a field for each demographic, for each new customer
- Make sure all the data types are correct for each field
- Remove duplicates. If a customer appears multiple times take their earliest joining date

## Output
````sql
WITH uniontables AS (
SELECT *, 'pd2023_wk04_january' as tablename FROM pd2023_wk04_january
UNION ALL 
SELECT *, 'pd2023_wk04_february' as tablename FROM pd2023_wk04_february
UNION ALL 
SELECT *, 'pd2023_wk04_march' as tablename FROM pd2023_wk04_march
UNION ALL 
SELECT *, 'pd2023_wk04_april' as tablename FROM pd2023_wk04_april
UNION ALL
SELECT *, 'pd2023_wk04_may' as tablename FROM pd2023_wk04_may
UNION ALL
SELECT *, 'pd2023_wk04_june' as tablename FROM pd2023_wk04_june
UNION ALL
SELECT *, 'pd2023_wk04_july' as tablename FROM pd2023_wk04_july
UNION ALL
SELECT *, 'pd2023_wk04_august' as tablename FROM pd2023_wk04_august
UNION ALL
SELECT *, 'pd2023_wk04_september' as tablename FROM pd2023_wk04_september
UNION ALL
SELECT *, 'pd2023_wk04_october' as tablename FROM pd2023_wk04_october
UNION ALL
SELECT *, 'pd2023_wk04_november' as tablename FROM pd2023_wk04_november
UNION ALL
SELECT *, 'pd2023_wk04_december' as tablename FROM pd2023_wk04_december
)

, transformations AS (
SELECT 
id,
date_from_parts(2023,DATE_PART('month',DATE(SPLIT_PART(tablename,'_',3),'MMMM')),joining_day) as joining_date,
demographic,
value
FROM uniontables
)
,table_pivot AS (
SELECT 
id,
joining_date,
ethnicity,
account_type,
date_of_birth::date as date_of_birth,
ROW_NUMBER() OVER(PARTITION BY id ORDER BY joining_date ASC) as rn
FROM transformations
PIVOT(MAX(value) FOR demographic IN ('Ethnicity','Account Type','Date of Birth')) AS P
(id,
joining_date,
ethnicity,
account_type,
date_of_birth)
)
SELECT 
id,
joining_date,
account_type,
date_of_birth,
ethnicity
FROM table_pivot
WHERE rn = 1
````
Sample

| ID     | JOINING_DATE | ACCOUNT_TYPE | DATE_OF_BIRTH | ETHNICITY |
|--------|--------------|--------------|----------------|-----------|
| 467374 | 2023-04-24   | Gold         | 2006-01-03     | Black     |
| 213378 | 2023-09-15   | Basic        | 1987-07-03     | Black     |
| 356495 | 2023-09-20   | Platinum     | 1981-02-02     | White     |
| 808528 | 2023-11-26   | Gold         | 1992-04-05     | White     |
| 721246 | 2023-12-13   | Gold         | 2002-06-24     | Asian     |
| 388013 | 2023-10-27   | Gold         | 2018-07-13     | Black     |
| 201221 | 2023-08-06   | Gold         | 1941-11-10     | Black     |
| 784400 | 2023-11-26   | Gold         | 2002-12-10     | Asian     |
| 938419 | 2023-08-17   | Basic        | 2003-10-04     | White     |
| 512357 | 2023-09-25   | Basic        | 2009-11-19     | White     |
| 221128 | 2023-06-29   | Basic        | 1978-08-16     | Black     |
| 673773 | 2023-08-26   | Gold         | 1951-09-22     | White     |
| 845293 | 2023-01-02   | Platinum     | 1946-05-28     | Black     |
| 942724 | 2023-10-09   | Platinum     | 1979-05-23     | Black     |
| 748491 | 2023-11-20   | Platinum     | 1957-12-07     | Asian     |
