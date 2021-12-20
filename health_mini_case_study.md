Here's the code that needs to be debugged:
```sql
-- 1. How many unique users exist in the logs dataset?
SELECT
  COUNT DISTINCT user_id
FROM health.user_logs;

-- for questions 2-8 we created a temporary table
DROP TABLE IF EXISTS user_measure_count;
CREATE TEMP TABLE user_measure_cout
SELECT
    id,
    COUNT(*) AS measure_count,
    COUNT(DISTINCT measure) as unique_measures
  FROM health.user_logs
  GROUP BY 1; 

-- 2. How many total measurements do we have per user on average?
SELECT
  ROUND(MEAN(measure_count))
FROM user_measure_count;

-- 3. What about the median number of measurements per user?
SELECT
  PERCENTILE_CONTINUOUS(0.5) WITHIN GROUP (ORDER BY id) AS median_value
FROM user_measure_count;

-- 4. How many users have 3 or more measurements?
SELECT
  COUNT(*)
FROM user_measure_count
HAVING measure >= 3;

-- 5. How many users have 1,000 or more measurements?
SELECT
  SUM(id)
FROM user_measure_count
WHERE measure_count >= 1000;

-- 6. Have logged blood glucose measurements?
SELECT
  COUNT DISTINCT id
FROM health.user_logs
WHERE measure is 'blood_sugar';

-- 7. Have at least 2 types of measurements?
SELECT
  COUNT(*)
FROM user_measure_count
WHERE COUNT(DISTINCT measures) >= 2;

-- 8. Have all 3 measures - blood glucose, weight and blood pressure?
SELECT
  COUNT(*)
FROM usr_measure_count
WHERE unique_measures = 3;

-- 9.  What is the median systolic/diastolic blood pressure values?
SELECT
  PERCENTILE_CONT(0.5) WITHIN (ORDER BY systolic) AS median_systolic
  PERCENTILE_CONT(0.5) WITHIN (ORDER BY diastolic) AS median_diastolic
FROM health.user_logs
WHERE measure is blood_pressure;
```

And here's my debugging and question answering, step by step:
1. How many unique users exist in the logs dataset?
```sql
SELECT
  COUNT(DISTINCT id)
FROM health.user_logs;

-- debugged: missing parenthesis, incorrect column name
-- 554 unique users exist in the dataset
```

* For the next series of questions, we create a temp table to reference:
```sql
DROP TABLE IF EXISTS user_measure_count;
CREATE TEMP TABLE user_measure_count AS
  SELECT
    id,
    COUNT(*) AS measure_count,
    COUNT(DISTINCT measure) as unique_measures
  FROM health.user_logs
  GROUP BY 1;

-- debugged: fixed typo in temp table name, added AS, tabbed SELECT
```

2. How many total measurements do we have per user on average?
```sql
SELECT
  ROUND(AVG(measure_count))
FROM user_measure_count;

-- debugged: the function name is AVG, not MEAN
-- there are an average of 79 measures per user
```

3. What about the median number of measurements per user?
```sql
SELECT
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY measure_count) AS median_value
FROM user_measure_count;

-- debugged: ORDER BY within the CTE needs to be by measure_count; it doesn't make sense to find the median of id #s
-- median number of measurements per user is 2
```

4. How many users have 3 or more measurements?
```sql
SELECT
  COUNT(*)
FROM user_measure_count
WHERE measure_count >= 3;

-- debugged: the column we need to reference in the temp table is measure_count, not measure; it also needs to go in a WHERE clause vs a HAVING
-- 209 users have 3 or more measurements
```

5. How many users have 1,000 or more measurements?
```sql
SELECT
  COUNT(*)
FROM user_measure_count
WHERE measure_count >= 1000;

-- debugged: the correct function is COUNT()
-- 5 users have 1000+ measurements
```

6. How many have logged blood glucose measurements?
```sql
SELECT
  COUNT(DISTINCT id)
FROM health.user_logs
WHERE measure = 'blood_glucose';

-- debugged: added parentheses for COUNT() function; checking for measure type needs an =, not "is"; measure label is 'blood_glucose', not 'blood_sugar'
-- 325 users logged blood glucose measurements
```

7. Have at least 2 types of measurements?
```sql
SELECT
  COUNT(*)
FROM user_measure_count
WHERE unique_measures >= 2;

-- since we are querying a temp table, we've already SELECTed COUNT(DISTINCT measure) as its own column, unique measures, so we can use that in the WHERE clause
-- 204 users recorded measurements in at least 2 categories
```

8. Have all 3 measures - blood glucose, weight and blood pressure?
```sql
SELECT
  COUNT(*)
FROM user_measure_count
WHERE unique_measures = 3;

-- debugged: fixed typo in temp table name
-- 50 users recorded measurements in all three categories
```

9.  What is the median systolic/diastolic blood pressure values?
```sql
SELECT
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY systolic) AS median_systolic
  , PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY diastolic) AS median_diastolic
FROM health.user_logs
WHERE measure = 'blood_pressure'
;

-- debugged: conditional WHERE clause needs =, not "is"; column name needs single quotes around it; the CTE for median (PERCENTILE_CONT() requires WITHIN GROUP;
-- missing comma to separate the SELECT columns;
-- median diastolic measurement is 79, median systolic is 126
```
