# SQL-Data-Cleaning-Project

-- Removing Duplicates


SELECT * FROM layoffs;


CREATE TABLE layoffs_staging
(LIKE layoffs);


SELECT * 
FROM layoffs_staging;


INSERT INTO layoffs_staging
SELECT * 
FROM layoffs;


SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company, industry, total_laid_off,percentage_laid_off,date)
FROM layoffs_staging;


WITH duplicate_cte AS
(SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company, location, 
industry, total_laid_off,percentage_laid_off,date, stage, 
country, funds_raised_millions) AS row_number
FROM layoffs_staging
)
SELECT * 
FROM duplicate_cte
WHERE row_number > 1;


SELECT * 
FROM layoffs_staging
WHERE company = 'Casper';



WITH duplicate_cte AS
(SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company, location, 
industry, total_laid_off,percentage_laid_off,date, stage, 
country, funds_raised_millions) 
AS row_number
FROM layoffs_staging
)
DELETE  
FROM duplicate_cte
WHERE row_number > 1;



CREATE TABLE IF NOT EXISTS world_layoffs.layoffs_staging2
(
    company character varying COLLATE pg_catalog."default",
    location character varying COLLATE pg_catalog."default",
    industry character varying COLLATE pg_catalog."default",
    total_laid_off numeric,
    percentage_laid_off character varying COLLATE pg_catalog."default",
    date character varying COLLATE pg_catalog."default",
    stage character varying COLLATE pg_catalog."default",
    country character varying COLLATE pg_catalog."default",
    funds_raised_millions numeric,
	row_number numeric 
);


SELECT * 
FROM layoffs_staging2
	WHERE row_number > 1;

INSERT INTO layoffs_staging2
SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company, location, 
industry, total_laid_off,percentage_laid_off,date, stage, 
country, funds_raised_millions) AS row_number
FROM layoffs_staging;


DELETE
FROM layoffs_staging2
	WHERE row_number > 1;

SELECT *
FROM layoffs_staging2;


	--Standardizing Data 

SELECT company, TRIM(company)
FROM layoffs_staging2;

UPDATE layoffs_staging2
SET company = TRIM(company);

SELECT DISTINCT industry
FROM layoffs_staging2
;


UPDATE layoffs_staging2
SET industry = 'Crypto'
WHERE industry LIKE 'Crypto%';


SELECT DISTINCT country, TRIM(TRAILING '.' FROM country)
FROM layoffs_staging2
ORDER BY 1;

UPDATE layoffs_staging2 
SET country = TRIM(TRAILING '.' FROM country)
WHERE country LIKE 'United States%';


SELECT date,
	TO_DATE(date, 'MM-DD-YYYY')
FROM layoffs_staging2;


UPDATE layoffs_staging2
SET date = TO_DATE(date, 'MM-DD-YYYY');


SELECT date
FROM layoffs_staging2;

ALTER TABLE layoffs_staging2
ALTER COLUMN date TYPE DATE USING date::DATE;


--Addressing NULL or blank values


UPDATE layoffs_staging2
SET industry = NULL
	WHERE industry = '';


SELECT *
FROM layoffs_staging2
WHERE industry IS NULL
OR industry = '';


SELECT *
FROM layoffs_staging2
WHERE company = 'Airbnb';

SELECT * 
FROM layoffs_staging2 AS t1
JOIN layoffs_staging2 AS t2
	ON t1.company = t2.company
	AND t1.location = t2.location 
WHERE (t1.industry IS NULL OR t1.industry = '')
AND t2.industry IS NOT NULL;


UPDATE layoffs_staging2 AS t1
SET industry = t2.industry
FROM layoffs_staging2 AS t2
WHERE t1.company = t2.company
AND (t1.industry IS NULL OR t1.industry = '')
AND t2.industry IS NOT NULL;


SELECT *
FROM layoffs_staging2;


SELECT * 
FROM layoffs_staging2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;


--Removing Rows and Columns


DELETE 
FROM layoffs_staging2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;

SELECT * 
FROM layoffs_staging2;


ALTER TABLE layoffs_staging2
DROP COLUMN row_number;


SELECT * 
FROM layoffs_staging2;
