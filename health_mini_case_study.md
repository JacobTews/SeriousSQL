### Cleaned code and answered questions:

([buggy code](#code-needing-cleaning) for reference)

<br/>

*1. How many unique users exist in the logs dataset?*
```sql
SELECT
  COUNT(DISTINCT id) AS unique_users
FROM health.user_logs
;
```

| unique_users |
| --- |
| 554 |

* debugged:
  * missing parenthesis
  * incorrect column name

<br/>

For the next series of questions, we create a temp table to reference:
```sql
DROP TABLE IF EXISTS user_measure_count;
CREATE TEMP TABLE user_measure_count AS
  SELECT
    id
    , COUNT(*) AS measure_count
    , COUNT(DISTINCT measure) AS unique_measures
  FROM health.user_logs
  GROUP BY 1
  ;
```

* debugged:
  * fixed typo in temp table name
  * added `AS`
  * tabbed `SELECT`

<br/>

*2. How many total measurements do we have per user on average?*
```sql
SELECT
  ROUND(AVG(measure_count)) AS measures_per_user
FROM user_measure_count
;
```

| measures_per_user |
| --- |
| 79 |

* debugged:
  * the function name is `AVG`, not `MEAN`

<br/>

*3. What about the median number of measurements per user?*
```sql
SELECT
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY measure_count) AS median_value
FROM user_measure_count
;
```

| median_value |
| --- |
| 2 |

* debugged:
  * `ORDER BY` within the CTE needs to be by `measure_count`; it doesn't make sense to find the median of id #s

<br/>

*4. How many users have 3 or more measurements?*
```sql
SELECT
  COUNT(*) AS users_with_3_plus
FROM user_measure_count
WHERE measure_count >= 3
;
```

| users_with_3_plus |
| --- |
| 209 |

* debugged:
  * the column we need to reference in the temp table is `measure_count`, not `measure`
  * conditional statement needs to go in a `WHERE` clause vs a `HAVING`

<br/>

*5. How many users have 1,000 or more measurements?*
```sql
SELECT
  COUNT(*) AS users_with_1000_plus
FROM user_measure_count
WHERE measure_count >= 1000
;
```

| users_with_1000_plus |
| --- |
| 5 |

* debugged:
  * the correct function is `COUNT()`

<br/>

*6. How many have logged blood glucose measurements?*
```sql
SELECT
  COUNT(DISTINCT id) AS glucose_users
FROM health.user_logs
WHERE measure = 'blood_glucose'
;
```

| glucose_users |
| --- |
| 325 |

* debugged:
  * added parentheses for `COUNT()` function
  * checking for measure type needs an `=`, not `is`
  * measure label is `'blood_glucose'`, not `'blood_sugar'`

<br/>

*7. Have at least 2 types of measurements?*
```sql
SELECT
  COUNT(*) AS two_plus_users
FROM user_measure_count
WHERE unique_measures >= 2
;
```

| two_plus_users |
| --- |
| 204 |

* debugged:
  * since we are querying a temp table, we've already `SELECT`ed `COUNT(DISTINCT measure)` as its own column, `unique_measures`, so we can use that in the `WHERE` clause

<br/>

*8. Have all 3 measures - blood glucose, weight and blood pressure?*
```sql
SELECT
  COUNT(*) AS all_measures_users
FROM user_measure_count
WHERE unique_measures = 3
;
```

| all_measures_users |
| --- |
| 50 |

* debugged:
  * fixed typo in temp table name

<br/>

*9.  What is the median systolic/diastolic blood pressure values?*
```sql
SELECT
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY systolic) AS median_systolic
  , PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY diastolic) AS median_diastolic
FROM health.user_logs
WHERE measure = 'blood_pressure'
;
```

| median_systolic | median_diastolic |
| --- | --- |
| 126 | 79 |

* debugged:
  * conditional `WHERE` clause needs `=`, not `is`;
  * column name needs single quotes around it;
  * the CTE for median (`PERCENTILE_CONT()`) requires `WITHIN GROUP`;
  * missing comma to separate the `SELECT` columns;

<br/>
<br/>

### Code needing cleaning:
(back up to [clean code](#cleaned-code-and-answered-questions))

```sql
-- 1. How many unique users exist in the logs dataset?
SELECT
  COUNT DISTINCT user_id
FROM health.user_logs;

-- for questions 2-8 we need a temporary table
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
