# Data-Cleaning-in-MySQL
Data cleaning in MySQL involves identifying and correcting errors, inconsistencies, and inaccuracies in a database to ensure data quality. This process is crucial for accurate analysis and decision-making.
IMPORT THE DATA (Use SQL Server Import Wizard or BULK INSERT)
-- Example BULK INSERT (modify path as needed)
BULK INSERT layoffs
FROM 'C:\path\to\layoffs.csv'
WITH (
    FIELDTERMINATOR = ',',
    ROWTERMINATOR = '\n',
    FIRSTROW = 2,
    FORMAT = 'CSV'
);

-- 3. CHECK FOR MISSING VALUES
SELECT 
    SUM(CASE WHEN company IS NULL THEN 1 ELSE 0 END) AS missing_company,
    SUM(CASE WHEN industry IS NULL THEN 1 ELSE 0 END) AS missing_industry,
    SUM(CASE WHEN total_laid_off IS NULL THEN 1 ELSE 0 END) AS missing_total_laid_off,
    SUM(CASE WHEN percentage_laid_off IS NULL THEN 1 ELSE 0 END) AS missing_percentage_laid_off,
    SUM(CASE WHEN date IS NULL THEN 1 ELSE 0 END) AS missing_date,
    SUM(CASE WHEN stage IS NULL THEN 1 ELSE 0 END) AS missing_stage,
    SUM(CASE WHEN country IS NULL THEN 1 ELSE 0 END) AS missing_country,
    SUM(CASE WHEN funds_raised_millions IS NULL THEN 1 ELSE 0 END) AS missing_funds
FROM layoffs;

-- 4. HANDLE MISSING VALUES
-- Option 1: Fill NULLs with 'Unknown' for categorical data
UPDATE layoffs SET industry = 'Unknown' WHERE industry IS NULL;
UPDATE layoffs SET stage = 'Unknown' WHERE stage IS NULL;

-- Option 2: Fill missing numeric values with 0 (or use an average if applicable)
UPDATE layoffs SET total_laid_off = 0 WHERE total_laid_off IS NULL;
UPDATE layoffs SET funds_raised_millions = 0 WHERE funds_raised_millions IS NULL;

-- 5. REMOVE DUPLICATES
WITH CTE AS (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY company, date ORDER BY (SELECT NULL)) AS row_num
    FROM layoffs
)
DELETE FROM CTE WHERE row_num > 1;

-- 6. STANDARDIZE TEXT FORMATTING
UPDATE layoffs SET company = LTRIM(RTRIM(company));
UPDATE layoffs SET industry = UPPER(industry);
UPDATE layoffs SET country = UPPER(country);

-- 7. CONVERT DATE COLUMN TO PROPER FORMAT
ALTER TABLE layoffs ALTER COLUMN date DATE;

-- 8. VALIDATE CLEANED DATA
SELECT * FROM layoffs WHERE total_laid_off < 0 OR funds_raised_millions < 0;
