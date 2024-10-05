# A Data Cleaning Project in MySQL

## Project Overview

**Project Title**: Data Cleaning in MySQL
**Database**: `world_layoffs`

This project is designed to demonstrate SQL skills and techniques typically used by BI and data analysts to clean the data. This project involves setting up a world_layoffs database and cleaning the data using SQL queries.

## Objectives

1. **Set up a world layoffs database**
2. **Data Cleaning**

## DATA CLEANING STEPS
1. **Remove Duplicates**
2. **Standardize the data**
3. **Deal with Null values and blank values**
4. **Remove any columns (if required)**

**Viewing all records**
```sql
SELECT *
FROM layoffs;
```

**Creates a staging table to keep the original table unmodified (only schema)**
```sql
CREATE TABLE layoffs_staging 			                      
LIKE layoffs;
```

**Inserts values**
```sql
INSERT layoffs_staging				                          
Select * 
From layoffs;
```

**Partinioning the data based on all columns(change according to every projects need) to assign a row number in order to identify duplicate records**
```sql
SELECT *,
ROW_NUMBER () Over (
	PARTITION BY company, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions ) AS row_num
FROM layoffs_staging;                                   
```

**Identifying duplicate records**
```sql
WITH duplicate_cte AS (
	SELECT *,
		ROW_NUMBER () Over (
		PARTITION BY company, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions) AS row_num
		FROM layoffs_staging)
SELECT * 
FROM duplicate_cte
WHERE row_num > 1;  						                      
```

**Verifying duplicate record**
```sql
SELECT *
FROM layoffs_staging
WHERE company = 'Casper' 				                    	
```

**Creating another staging table with an additional column row_num**
```sql
CREATE TABLE `layoffs_staging2` (
  `company` text,
  `location` text,
  `industry` text,
  `total_laid_off` int DEFAULT NULL,
  `percentage_laid_off` text,
  `date` text,
  `stage` text,
  `country` text,
  `funds_raised_millions` int DEFAULT NULL,
  `row_num` int
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```

```sql
SELECT *
FROM layoffs_staging2;
```

**Inserting all rows and a new row_num column from layoffs_staging table**
```sql
INSERT INTO layoffs_staging2 		                                    
SELECT *,
		ROW_NUMBER () Over (
		PARTITION BY company, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions) AS row_num
		FROM layoffs_staging;
```

**Checking for Duplicates again**
```sql
SELECT * 						                                                 
FROM layoffs_staging2
WHERE row_num > 1;
```

**Deleting duplicates**
```sql
DELETE							                                                 
FROM layoffs_staging2
WHERE row_num > 1;
```

## STANDARDIZING DATA (Finding issues in your data and standardizing them)

**Trimming extra spaces in company column**
```sql
UPDATE layoffs_staging2
SET company= TRIM(company);
```

**Checking Industry column for issues**
```sql
SELECT DISTINCT industry
FROM layoffs_staging2
ORDER by industry
```

**Issues: NULL, BLANK, (Crypto (majorly), Crypto Currency, CryptoCurrency)**
```sql
SELECT * 
FROM layoffs_staging2 
WHERE industry LIKE 'Crypto%';
```

**UPDATING the industry**
```sql
UPDATE layoffs_staging2
SET industry= 'Crypto'
WHERE industry IN ('Crypto Currency','CryptoCurrency');
```

**Checking Location column for issues (no issues found)**
```sql
SELECT DISTINCT location
FROM layoffs_staging2
ORDER by location;
```

**Checking country column for issues (Issue: United States with a period)**
```sql
SELECT DISTINCT country
FROM layoffs_staging2
ORDER by country;
```

```sql
SELECT * 
FROM layoffs_staging2 
WHERE country LIKE 'United States.%';
```

**Updating into the correct format**
```sql
UPDATE layoffs_staging2   
SET country = 'United States'
WHERE country = 'United States.'
```

**Changing the datatype of date column**
```sql
SELECT date, STR_TO_DATE(date, '%m/%d/%Y') -- deciding the correct date format to update
FROM layoffs_staging2;
```

**Updating the date column format**
```sql
UPDATE layoffs_staging2
SET date= STR_TO_DATE(date, '%m/%d/%Y');
```

**Changing the datatype of the date column**
```sql
ALTER TABLE layoffs_staging2
MODIFY COLUMN `date` DATE;
```


**Checking for NULL/blank values and Populating NULL Values**
```sql
SELECT * 
FROM layoffs_staging2 
WHERE industry IS NULL
OR industry = '';
```

**Identifying the columns with blank/null industries that can be updated**
```sql
SELECT t1.industry, t2.industry
FROM layoffs_staging2 t1
JOIN layoffs_staging2 t2
	ON  t1.company = t2.company
WHERE (t1.industry IS NULL OR t1.industry = '')
AND t2.industry IS NOT NULL;
```

**Updating all the blank to NUll first**
```sql
UPDATE layoffs_staging2
SET industry = NULL
WHERE industry = '';
```

**Updating all the NULL industries to the desired industries**
```sql
UPDATE layoffs_staging2 t1
JOIN layoffs_staging2 t2
	ON  t1.company = t2.company
SET t1.industry=t2.industry
WHERE (t1.industry IS NULL OR t1.industry = '')
AND t2.industry IS NOT NULL;
```

**Identifying Null values and checking if it will be helpful with our EDA**
```sql
SELECT * 
FROM layoffs_staging2 
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;
```

**Deleting this data as it wont be helpful in the analysis**
```sql
DELETE FROM layoffs_staging2 
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;
```

**Dropping the row_number column**
```sql 
ALTER TABLE layoffs_staging2
DROP COLUMN row_num;
```

**Cleaned Data**
```sql
SELECT *
FROM layoffs_staging2;
```


## End of the data cleaning project
