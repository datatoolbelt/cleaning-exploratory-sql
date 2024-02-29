# Exploration of laptop dataset

## Getting head
```sql
SELECT *
FROM laptops
ORDER BY `index`
LIMIT 5;
```

## Getting tail
```sql
SELECT *
FROM laptops
ORDER BY `index` DESC
LIMIT 5;
```

## Take a look at random 5 values
```sql
SELECT *
FROM laptops
ORDER BY RAND()
LIMIT 5;
```

## Numerical Univariate Analysis
```sql
-- 8 number summary [count, min, max, mean, std, q1, q2, q3]
	SELECT COUNT(Price),
			MIN(Price),
			MAX(Price),
			AVG(Price),
			STD(Price)
	FROM laptops;

-- Missing values
	SELECT COUNT(Price)
	FROM laptops
	WHERE Price IS NULL;
```

## Categorical Univariate Analysis
```sql
-- Value counts, taking 'Company'
	SELECT Company, COUNT(Company)
	FROM laptops
	GROUP BY Company;
	
-- Missing values
	SELECT COUNT(Company)
	FROM laptops
	WHERE Company IS NULL;
```

## Numerical-Categorical Bivariate Analysis
```sql
-- Compare distributions across categories
	SELECT Company,
		  AVG(Price) AS 'avg_price',
	    MIN(Price) AS 'min_price',
	    MAX(Price) AS 'max_price',
	    STD(Price) AS 'std_dev'
	FROM laptops
	GROUP BY Company;
```

## Filling missing values in `Price` column
Filling nulls with average price by company

```sql
	UPDATE laptops
	SET Price = (SELECT AVG(Price)
				FROM laptops
				GROUP BY Company)
  WHERE Price IS NULL;
```

## Feature Engineering
**Add a new column that shows ppi for each laptop**

```sql
-- Add a new column namely, ppi
	ALTER TABLE laptops
	ADD COLUMN ppi INTEGER AFTER screen_height;

-- Fill in with ppi formula
UPDATE laptops
SET ppi = (ROUND(
					      SQRT((screen_width*screen_width) + (screen_height*screen_height)) / Inches)
);
```

**Add category by screen size; small, medium and large**
```sql
-- Add a new column
ALTER TABLE laptops
ADD COLUMN screen_type VARCHAR(255) AFTER Inches;
	
-- Fill the column
UPDATE laptops
SET screen_type = (CASE
                      WHEN Inches <= 14 THEN 'small'
                      WHEN Inches > 14 AND Inches <17 THEN 'medium'
                      WHEN Inches >= 17  THEN 'large'
                   END);
```

Thanks!
