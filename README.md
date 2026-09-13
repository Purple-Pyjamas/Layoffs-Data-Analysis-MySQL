# :purple_square: Data Analysis of a Layoff Dataset With MySQL

This project demonstrates an end-to-end SQL data analysis workflow using a global layoffs dataset. The project was completed in two main stages: Data Cleaning and Exploratory Data Analysis. 

I first cleaned and transformed the raw dataset in MySQL to improve its consistency and reliability. I then analyzed the cleaned data to investigate layoff variations across companies, industries, countries, company stages, and time or seasonality.

This task was not done only to enhance productivity and save time by preparing messy real-world data, reducing the risk of chasing the wrong targets and to make visualization of data findings accurate and seamless; It also shows how SQL can be used to uncover patterns and structure data for meaningful analysis.
 
---

## :clipboard: Project Overview

### Data cleaning

Raw data is rarely ready for analysis. In this project, I used **MySQL** to clean and prepare a layoffs dataset for analysis, focusing on improving data quality, consistency, and usability. The data showed varying numbers of employees who were laid off across various companies, as well as what stage those companies were in when the layoffs happened, and how much revenue was made in the same year. The cleaning process included:

* Creating staging tables to preserve the original dataset
* Identifying and removing duplicate records
* Standardizing inconsistent text values
* Cleaning country and industry categories
* Converting date values into the correct SQL `DATE` format
* Identifying and handling null and blank values
* Populating missing industry information using existing company records
* Removing records that contained insufficient layoff information
* Removing temporary helper columns after cleaning

The final result is a cleaner, more structured dataset that can be used for exploratory data analysis, visualization, or further business analysis.

### Data Analysis Objectives

The project focuses on answering the following questions:

* Which companies recorded the highest total layoffs?
* Which industries experienced the most layoffs?
* Which countries were most affected?
* How did layoffs change from year to year?
* Which company stages experienced the highest layoffs?
* How did layoffs evolve month by month?
* What does the cumulative trend in layoffs look like?
* Which five companies recorded the highest layoffs in each year?
* Which companies laid off 100% of their reported workforce?

---

## :hammer_and_wrench: Tools & Technologies

* **MySQL**
* **SQL**
* MySQL Workbench

### SQL concepts used: 

* `CREATE TABLE`
* `INSERT INTO`
* `UPDATE`
* `DELETE`
* `ALTER TABLE`
* `ROW_NUMBER()`
* Window functions
* Common Table Expressions — CTEs
* Self joins
* `PARTITION BY`
* `TRIM()`
* `STR_TO_DATE()`
* `IS NULL`
* Pattern matching with `LIKE`
* Data type conversion
* `SELECT`
* `GROUP BY`
* `ORDER BY`
* `SUM()`
* `DENSE_RANK()`
* `OVER()` 

---

## :microscope: Methodology and Project Workflow

## 1. Data Cleaning

The first phase focused on transforming the raw layoffs dataset into a clean and reliable dataset suitable for analysis.

### 1.1 Creating a Staging Table

Instead of modifying the original dataset directly, I created a copy of the dataset, titled it "layoffs_staging," and performed the cleaning process on that copy. This preserved the raw dataset while allowing transformations to be performed safely on the copy.

`CREATE TABLE layoffs_staging
LIKE layoffs;`

`INSERT layoffs_staging
SELECT *
FROM layoffs;`

### 1.2 Removing Duplicate Records

I used ROW_NUMBER() to identify records containing identical information across the relevant columns. Rows with a row_num greater than 1 were classified as duplicates and removed.

`ROW_NUMBER() OVER(
    PARTITION BY company,
                 location,
                 industry,
                 total_laid_off,
                 percentage_laid_off,
                 date,
                 stage,
                 country,
                 funds_raised_millions
) AS row_num`

### 1.3 Standardizing Data

Several inconsistencies were corrected to make categorical data more reliable. Examples include:

`Company names:
" Airbnb " → "Airbnb"`

`Industry:
CryptoCurrency → Crypto`

`Country:
United States. → United States`

Company names were cleaned using TRIM(), similar industry categories were standardized, and unnecessary punctuation was removed from country names.

### 1.4 Converting Dates

The date column was originally stored as text. I converted it to MySQL's DATE datatype using:

`UPDATE layoffs_staging2
SET date = STR_TO_DATE(date, '%m/%d/%Y')`

`ALTER TABLE layoffs_staging2
MODIFY COLUMN date DATE`

### 1.5 Handling Missing Values

Blank industry values were converted to NULL. Instead of immediately deleting records with missing industries, I investigated whether another record belonging to the same company contained the information. A self join was then used to populate the missing values:

`UPDATE layoffs_staging2 t1
JOIN layoffs_staging2 t2
    ON t1.company = t2.company
SET t1.industry = t2.industry
WHERE t1.industry IS NULL
AND t2.industry IS NOT NULL`

### 1.6 Removing Unusable Records

Records where both total_laid_off and percentage_laid_off were missing contained insufficient information for the analysis and were therefore removed. The temporary row_num helper column was also removed.

`DELETE
FROM layoffs_staging2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL`

## 2. Exploratory Data Analysis

After cleaning the dataset, I used SQL to explore patterns in the layoffs data.

### 2.1 Maximum Layoffs

I began by examining the maximum number of employees laid off and the maximum percentage of employees laid off. I also ordered these companies by the amount of funding they had raised to explore whether heavily funded companies were among the businesses that reported complete workforce layoffs.

`SELECT MAX(total_laid_off),
       MAX(percentage_laid_off)
FROM layoffs_staging2;`

`SELECT *
FROM layoffs_staging2
WHERE percentage_laid_off = 1
ORDER BY total_laid_off DESC;`

`Total Layoffs by Company
SELECT company,
       SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY company
ORDER BY 2 DESC;`

This identifies companies with the largest cumulative layoffs in the dataset.

`Layoffs by Industry
SELECT industry,
       SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY industry
ORDER BY 2 DESC;`

This allows for comparison of the impact of layoffs across industries.

`Layoffs by Country
SELECT country,
       SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY country
ORDER BY 2 DESC;`

This identifies countries with the highest reported layoffs.

`Layoffs by Year
SELECT YEAR(date),
       SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY YEAR(date)
ORDER BY 1 DESC;`

This shows how layoffs changed over time.

`Layoffs by Company Stage
SELECT stage,
       SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY stage
ORDER BY 2 DESC;`

This helps investigate whether layoffs were concentrated among startups, established companies, public companies, or businesses at other funding stages.

### 2.3 Monthly Layoff Trends

I aggregated layoffs by month to examine shorter-term trends.

`SELECT SUBSTRING(date,1,7) AS MONTH,
       SUM(total_laid_off)
FROM layoffs_staging2
WHERE SUBSTRING(date,1,7) IS NOT NULL
GROUP BY MONTH
ORDER BY 1;`

### 2.4 Rolling Total of Layoffs

To understand how layoffs accumulated over time, I combined a Common Table Expression with a window function.

`WITH Rolling_Total AS
(
    SELECT SUBSTRING(date,1,7) AS MONTH,
           SUM(total_laid_off) AS total_off
    FROM layoffs_staging2
    WHERE SUBSTRING(date,1,7) IS NOT NULL
    GROUP BY MONTH
)
SELECT MONTH,
       total_off,
       SUM(total_off) OVER(ORDER BY MONTH) AS rolling_total
FROM Rolling_Total;`

This produces both monthly layoffs and their cumulative total.

### 2.5 Top Companies by Layoffs Each Year

One of the more advanced parts of the project was identifying the companies with the highest layoffs in each year. First, layoffs were aggregated by company and year. Then, DENSE_RANK() was used to rank companies independently within each year.

`WITH Company_Year (company, years, total_laid_off) AS
(
    SELECT company,
           YEAR(date),
           SUM(total_laid_off)
    FROM layoffs_staging2
    GROUP BY company, YEAR(date)
),
Company_Year_Rank AS
(
    SELECT *,
           DENSE_RANK() OVER(
               PARTITION BY years
               ORDER BY total_laid_off DESC
           ) AS Ranking
    FROM Company_Year
    WHERE years IS NOT NULL
)`

`SELECT *
FROM Company_Year_Rank
WHERE Ranking <= 5;`

This returns the top five companies by total layoffs for each year represented in the dataset.

---

## :bulb: Key Lessons

This project reinforced an important lesson:

> Data cleaning is not simply about deleting missing values. It requires understanding what each record represents and deciding how inconsistencies should be handled without unnecessarily losing useful information.

---

## :dart: Skills Demonstrated in this Project

| Area	              | Skills                                                                 |
|--------------------|------------------------------------------------------------------------|
| Data Cleaning	      | Duplicate removal, null handling, standardization and string functions |
| Data Transformation |	Date conversion, categorical normalization                             |
| SQL Analysis	      | Aggregation, grouping, filtering                                       | 
| Advanced SQL	      | CTEs, self joins, window functions                                     | 
| Trend Analysis	  | Monthly and yearly analysis                                            |
| Ranking Analysis	  | DENSE_RANK() and PARTITION BY                                          | 
| Data Quality	      | Staging-table workflow and missing-value investigation                 |

**Data Cleaning • SQL • MySQL • Data Quality • Data Transformation • Window Functions • CTEs • Joins • Data Standardization • Missing Data Handling**

---

## ❓ Possible Next Steps

The next stage of this project could include:

* Building an interactive Power BI or Tableau dashboard
* Visualizing monthly layoff trends
* Comparing industries across time
* Examining geographic concentration of layoffs
* Investigating relationships between company funding and layoffs
* Creating additional KPIs and business-focused insights

---

## :mailbox: Contact

**Uchechukwu Esther Okwudili**  

:e-mail: [Send me an email](ucokwudili27@gmail.com)

:briefcase: [My Linkedin profile](www.linkedin.com/in/uchechukwu-okwudili-a7437933a)

:globe_with_meridians: [View my Portfolio]()
