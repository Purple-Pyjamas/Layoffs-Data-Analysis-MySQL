# :purple_square: Data Cleaning with MySQL

End-to-end data cleaning of a layoffs dataset using MySQL, while including duplicate removal, standardization, null handling, date transformation, and self joins to prepare a clean, consistent, and reliable data for smoother operations and better decision making. This task was done to enhance productivity and save time for the team, reduce risk of chasing the wrong targets and to make visualization of data findings accurate and seamless.

---

## :clipboard: Project Overview

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

---

## :microscope: Methodology

### 1. Created a Staging Table

Instead of modifying the raw dataset directly, I created a duplicate staging table. This helps to preserve the original data while allowing transformations to be carried out safely. It is especially useful when there is a need to reference the original data or when massive errors are made in the process.

```sql
CREATE TABLE layoffs_staging
LIKE layoffs;

INSERT layoffs_staging
SELECT *
FROM layoffs;
```


### 2. Removed Duplicate Records

I used the `ROW_NUMBER()` window function to assign a number to records sharing the same values across relevant columns.

```sql
ROW_NUMBER() OVER(
    PARTITION BY company,
                 location,
                 industry,
                 total_laid_off,
                 percentage_laid_off,
                 date,
                 stage,
                 country,
                 funds_raised_millions
) AS row_num
```

Records with a `row_num` greater than `1` were identified as duplicates and removed.

This approach provides more control than simply using `SELECT DISTINCT`, especially when inspecting duplicate records before deleting them.


### 3. Standardized Company Names

Extra spaces in company names can cause the same company to appear as different categories during analysis. So, I removed unnecessary spaces using:

```sql
UPDATE layoffs_staging2
SET company = TRIM(company);
```


### 4. Standardized Industry Categories

I inspected the distinct industry values to identify inconsistencies. For example, multiple industry labels beginning with **Crypto** were standardized into one category, instead of having some of them labelled "Cryptocurrency" and some labelled "Crypto":

```sql
UPDATE layoffs_staging2
SET industry = 'Crypto'
WHERE industry LIKE 'Crypto%';
```

This prevents similar categories from being treated as separate industries during analysis.


### 5. Standardized Country Names

Some country values contained unnecessary trailing punctuation. These were cleaned to create consistent country categories.

```sql
UPDATE layoffs_staging2
SET country = TRIM(TRAILING '.' FROM country)
WHERE country LIKE 'United States%';
```


### 6. Converted the Date Column

The date field was initially stored as text. I converted the values into MySQL's date format using:

```sql
STR_TO_DATE(date, '%m/%d/%Y')
```

Then changed the column datatype:

```sql
ALTER TABLE layoffs_staging2
MODIFY COLUMN date DATE;
```

Using the correct datatype makes future time-series analysis, filtering, sorting, and aggregation much easier.


### 7. Handled Null and Blank Values

I inspected columns containing missing information, particularly the `industry` field. Blank industry values were first converted to `NULL`:

```sql
UPDATE layoffs_staging2
SET industry = NULL
WHERE industry = '';
```


### 8. Populated Missing Industry Data

Some companies had multiple records where one record contained the industry, and another did not. I used a **self-join** to match companies with themselves and populate the missing industry values from records where that information already existed. This solved the missing data problem encountered in the previous step, and it allowed useful information already present within the dataset to be used instead of unnecessarily deleting those records.

```sql
UPDATE layoffs_staging2 t1
JOIN layoffs_staging2 t2
    ON t1.company = t2.company
SET t1.industry = t2.industry
WHERE t1.industry IS NULL
AND t2.industry IS NOT NULL;
```


### 9. Removed Unnecessary Data

Records where both:

* `total_laid_off` was NULL
* `percentage_laid_off` was NULL

contained insufficient information for meaningful layoff analysis.

These records were removed:

```sql
DELETE
FROM layoffs_staging2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;
```


### 10. Removed Temporary Columns

The `row_num` column was created only to help identify duplicates. After it had served its purpose, I removed it:

```sql
ALTER TABLE layoffs_staging2
DROP COLUMN row_num;
```


## :bulb: Key Lessons

This project reinforced an important lesson:

> Data cleaning is not simply about deleting missing values. It requires understanding what each record represents and deciding how inconsistencies should be handled without unnecessarily losing useful information.

I also gained practical experience using SQL techniques such as:

* Window functions for duplicate detection
* Self joins for filling missing information
* String functions for standardization
* Data transformations
* Null-value handling
* Safe staging-table workflows

---

## :dart: Skills Demonstrated

**Data Cleaning • SQL • MySQL • Data Quality • Data Transformation • Window Functions • CTEs • Joins • Data Standardization • Missing Data Handling**

---

## ❓ Possible Next Steps

The cleaned dataset can now be used for exploratory data analysis to answer questions such as:

* Which industries recorded the most layoffs?
* Which companies had the largest layoffs?
* How have layoffs changed over time?
* Which countries experienced the highest layoffs?
* Are layoffs concentrated in particular company stages?
* How do layoffs vary by year and month?

---

## :open_file_folder: Repository Structure

Data-Cleaning-mySQL/
│
├── README.md
│
│
├── layoffs.csv               dataset
│   
│
├── data_cleaning.sql         cleaned SQL data

---
