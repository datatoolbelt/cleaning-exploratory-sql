# 💻 Data Cleaning and EDA with SQL
Here's where you'll find all the code lines associated with the data cleaning and exploratory data analysis (EDA) project for the laptops dataset.
This repository alongside my medium [post](https://medium.com/@datatoolbelt/data-cleaning-and-eda-with-sql-4e70e84ef3b2) could serve as a very efficient resource for a someone starting their journey with SQL.

## What you'll find?
- Data Cleaning
- Exploratory Data Analysis
- PostgreSQL

## Data Cleaning

### Step 1: Creating Backup
As a best practice, it's always necessary to have backup of your data in it's raw form. This helps us go back/ refer to the original version in case there's a mishap.

```sql
-- create empty table of the same format
CREATE TABLE laptop_backup LIKE laptop;

-- fill the table with all the data in current table
INSERT INTO new_backup
SELECT *
FROM laptop;
```
### Step 2: Check the data
Every dataset has it's unique quirks even if they all look the same. In order to understand what is located where, what are the visible queues we should always have a brief look at it before we start making tweaks.

```sql
-- check number of rows
SELECT COUNT(*)
FROM laptop;
```
### Step 3: Check null values
This should be done with every column individually in order to check for nulls.

```sql
SELECT *
FROM laptop
WHERE <column_name> IS NULL;
```
### Step 4: Drop columns
Irrelevant columns can be dropped from our dataset

```sql
ALTER TABLE laptop
DROP COLUMN `Unnamed: 0`;
```

#### `Ram` should be converted to integer after removing GB
```sql
-- Remove 'GB' from column Ram
UPDATE laptop
SET Ram = REPLACE(Ram, 'GB', '');

-- Convert into integer
ALTER TABLE laptop
MODIFY COLUMN Ram INTEGER;

-- Rename Ram to Ram(GB)
ALTER TABLE laptop
RENAME COLUMN Ram TO `Ram(GB)`;
```

#### `Weight` column should be integer
```sql
-- Remove 'kg' from column Weight
UPDATE laptop
SET Weight = REPLACE(Weight, 'kg', '');

-- Convert into integer
ALTER TABLE laptop
MODIFY COLUMN Weight INTEGER;

Error Code: 1366. Incorrect integer value: '?' for column 'Weight' at row 202
```

#### `Weight` column further
```sql
SELECT *
FROM laptop
WHERE Weight = '?';
```

#### `Price` should be rounded to 2 digits after decimal
```sql
UPDATE laptop
SET Price = ROUND(Price, 2);
```

#### `ScreenResolution` — multiple levels of information
```sql
-- Add 3 columns namely screen_width, screen_height, touchscreen
ALTER TABLE laptop
ADD COLUMN screen_width INTEGER AFTER ScreenResolution,
ADD COLUMN screen_height INTEGER AFTER screen_width,
ADD COLUMN touchscreen VARCHAR(255) AFTER screen_height;

-- fill screen_width
UPDATE laptop
SET screen_width = SUBSTRING_INDEX(
                    SUBSTRING_INDEX(ScreenResolution, ' ', -1), 'x',1);

-- fill screen_height
UPDATE laptop
SET screen_height = (SUBSTRING_INDEX(ScreenResolution, 'x', -1));

-- fill touchscreen with 1, where touch else 0
UPDATE laptop
SET touchscreen = (CASE WHEN ScreenResolution LIKE '%touch%' THEN 1
                   ELSE 0
                   END);

-- drop column ScreenResolution
ALTER TABLE laptop
DROP COLUMN ScreenResolution;
```

#### `Cpu` — multiple levels of information
```sql
-- split information into 3 columns, cpu_brand, cpu_type, cpu_speed(GHz)
ALTER TABLE laptop
ADD COLUMN cpu_brand VARCHAR(255) AFTER Cpu,
ADD COLUMN cpu_type VARCHAR(255) AFTER cpu_brand,
ADD COLUMN `cpu_speed(GHz)` FLOAT(8) AFTER cpu_type;

-- fill cpu_brand
UPDATE laptop
SET cpu_brand = SUBSTRING_INDEX(Cpu, ' ', 1);

-- fill cpu_type
UPDATE laptop
SET cpu_type = SUBSTRING_INDEX(SUBSTRING_INDEX(Cpu, ' ', 3), ' ', -2);

-- fill cpu_speed(GHz)
UPDATE laptop
SET `cpu_speed(GHz)` = REPLACE(SUBSTRING_INDEX(Cpu, ' ', -1),'GHz', '');

-- drop Cpu
ALTER TABLE laptop
DROP COLUMN Cpu;
```

#### `Memory` — multiple levels of information
```sql
-- split into 3 columns, drive_type, primary_storage(GB) and 
-- secondary_storage(GB)

ALTER TABLE laptop
ADD COLUMN drive_type VARCHAR(255) AFTER Memory,
ADD COLUMN `primary_storage(GB)` INTEGER AFTER drive_type,
ADD COLUMN `secondary_storage(GB)` INTEGER AFTER `primary_storage(GB)`;

-- fill drive_type
UPDATE laptop
SET drive_type =  (CASE WHEN Memory LIKE '%SSD%' AND Memory LIKE '%HDD%' 
                        THEN 'Hybrid'
                        WHEN Memory LIKE '%SSD%' THEN 'SSD'
                        WHEN Memory LIKE '%HDD%' THEN 'HDD'
                        WHEN Memory LIKE '%Flash%' THEN 'Flash Storage'
                        WHEN Memory LIKE '%Hybrid%' THEN 'Hybrid'
                        ELSE NULL
                        END
                       );

-- fill primary_storage(GB)
UPDATE laptop
SET `primary_storage(GB)` =  (CASE WHEN Memory LIKE '%Flash%' 
                              OR Memory LIKE '%SSD%' 
                              THEN REGEXP_SUBSTR(SUBSTRING_INDEX(
                                        Memory, ' ', 1), '[0-9]+')
                              WHEN Memory LIKE '%HDD%' THEN 0
                              END
                              );

-- fill secondary_storage(GB)
UPDATE laptop
SET `secondary_storage(GB)` =  (CASE WHEN Memory LIKE '%+%' 
                              THEN REGEXP_SUBSTR(SUBSTRING_INDEX(
                                         Memory, ' ', -2), '[0-9]+')
                              WHEN Memory LIKE '%HDD%' 
                              THEN REGEXP_SUBSTR(SUBSTRING_INDEX(
                                         Memory, ' ', 1), '[0-9]+')
                              ELSE 0
                              END
                              );
                      
-- convert secondary_storage TB values to GB
UPDATE laptop
SET `secondary_storage(GB)` = (CASE WHEN `secondary_storage(GB)` <= 5 THEN 1024*`secondary_storage(GB)`
        ELSE `secondary_storage(GB)`
                                END);
-- drop Memory column
ALTER TABLE laptop
DROP COLUMN Memory;
```

#### `Gpu` — multiple levels of information
```sql
-- split into 2 columns namely, gpu_brand, gpu_name
ALTER TABLE laptop
ADD COLUMN gpu_brand VARCHAR(255) AFTER Gpu,
ADD COLUMN gpu_name VARCHAR(255) AFTER gpu_brand;

-- fill gpu_brand
UPDATE laptop
SET gpu_brand = SUBSTRING_INDEX(Gpu, ' ', 1);

-- fill gpu_name
UPDATE laptop
SET gpu_name = TRIM(REPLACE(Gpu, gpu_brand, ''));

-- drop Gpu
ALTER TABLE laptop
DROP COLUMN Gpu;
```
#### `Opsys`
```sql
-- create a new column
ALTER TABLE laptop
ADD COLUMN OpSystem VARCHAR(255) AFTER OpSys;

-- fill OpSystem column
UPDATE laptop
SET OpSystem = (CASE WHEN OpSys LIKE '%mac%' THEN 'Mac'
                     WHEN OpSys LIKE '%window%' THEN 'Windows'
                     WHEN OpSys LIKE '%linux%' THEN 'Linux'
                     WHEN OpSys LIKE '%No OS' THEN 'No-OS'
                     ELSE 'Others'
                     END);
                     
-- drop OpSys column
ALTER TABLE laptop
DROP COLUMN OpSys;
```

The final cleaned file can now be exported for further exploration on any of the visualisation softwares available. Our dataset went from 12 messy columns to 18 columns ready for analysis. Next step is exploration which you'll find [here](https://github.com/datatoolbelt/cleaning-exploratory-sql/blob/4f5c6b529580481742a49eff94aed5f6ac24f627/exploration.md).

Cheers!
