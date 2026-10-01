# Layoffs Data Cleaning (MySQL)

A SQL project that cleans a raw global tech layoffs dataset so it is ready for exploratory data analysis (EDA). All cleaning is done in MySQL using staging tables, so the original raw data is never modified.

## Dataset

The raw table `layoffs` contains the following columns:

| Column | Description |
|---|---|
| `company` | Company that conducted the layoffs |
| `location` | City / region of the company |
| `industry` | Industry of the company |
| `total_laid_off` | Number of employees laid off |
| `percentage_laid_off` | Percentage of the workforce laid off |
| `date` | Date of the layoff event |
| `stage` | Funding stage of the company |
| `country` | Country of the company |
| `funds_raised_millions` | Total funds raised (in millions USD) |

## Cleaning Workflow

1. **Remove duplicates**
2. **Standardize the data**
3. **Handle NULL and blank values**
4. **Remove unnecessary rows and columns**

### 0. Staging tables

To protect the raw data, the work is done on copies:

- `layoffs_staging` is a copy of `layoffs` (created with `CREATE TABLE ... LIKE` and `INSERT ... SELECT`).
- `layoffs_staging2` is a second staging table that adds a `row_num` helper column, used to identify and delete duplicates.

### 1. Remove duplicates

- The table has no unique ID, so duplicates are identified with `ROW_NUMBER() OVER (PARTITION BY ...)` across all columns.
- Any row with `row_num > 1` is a duplicate.
- MySQL cannot delete directly from a CTE, so the results are inserted into `layoffs_staging2` (which has the `row_num` column) and the duplicates are deleted from there.

### 2. Standardize the data

- **Whitespace:** trimmed `company` names with `TRIM()`.
- **Industry:** merged variants such as `Crypto Currency` and `CryptoCurrency` into a single `Crypto` value.
- **Country:** removed the trailing period from `United States.` using `TRIM(TRAILING '.' FROM country)`.
- **Dates:** converted `date` from text to a real date using `STR_TO_DATE(date, '%m/%d/%Y')`, then changed the column type with `ALTER TABLE ... MODIFY COLUMN date DATE`.

### 3. Null and blank values

- Converted blank `industry` values (`''`) to `NULL`.
- Populated missing `industry` values by self-joining on `company`, using the industry from another row of the same company (for example, Airbnb).
- Rows where both `total_laid_off` and `percentage_laid_off` are `NULL` were considered unusable for analysis.

### 4. Remove rows and columns

- Deleted rows where both `total_laid_off` and `percentage_laid_off` are `NULL`.
- Dropped the temporary `row_num` column.

## Result

The final table, `layoffs_staging2`, contains deduplicated, standardized data with proper data types and is ready for EDA.

## Skills Demonstrated

- CTEs
- Window functions (`ROW_NUMBER`, `PARTITION BY`)
- Self joins and `UPDATE` with `JOIN`
- String functions (`TRIM`, `LIKE`)
- Date conversion (`STR_TO_DATE`)
- `ALTER TABLE` (`MODIFY COLUMN`, `DROP COLUMN`)
- Staging-table best practices for safe data cleaning

## How to Run

1. Import the raw dataset into a MySQL database as a table named `layoffs`.
2. Run the cleaning script from top to bottom in MySQL Workbench (or any MySQL client).
3. Query `layoffs_staging2` for the cleaned data.

## Requirements

- MySQL 8.0 or later (window functions are required)

## Notes

- The original `layoffs` table is left untouched.
- Rows with no layoff figures were deleted. Depending on your analysis, you may prefer to keep them.
