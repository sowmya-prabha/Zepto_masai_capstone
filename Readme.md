# Data Pipeline

## Overview

This module scrapes book data from the **Books to Scrape** website using Python, cleans and transforms the scraped data, converts prices from GBP to INR, and stores the cleaned data in a normalized SQLite database.

The pipeline collects books from **8 different categories** and produces a dataset containing at least **100 books**.

## Project Structure

```text
data_pipeline/
│
├── scrape_and_load.py
├── queries.py
├── books.db
├── books_dataset.csv
├── books_dataset_cleaned.csv
│
├── query_outputs/
│   ├── query1.csv
│   ├── query2.csv
│   ├── query3.csv
│   ├── query4.csv
│   └── query5.csv
│
└── README.md
```

## Requirements

Python 3.x is required.

Install the required Python libraries:

```bash
pip install requests beautifulsoup4 pandas
```

SQLite is provided through Python's built-in `sqlite3` module, so no separate SQLite package is required.

## How to Run

From the project root directory, run:

```bash
python data_pipeline/scrape_and_load.py
```

The pipeline performs the following steps:

1. Scrapes books from 8 selected categories.
2. Extracts:

   * `title`
   * `price`
   * `star_rating`
   * `availability`
   * `category`
3. Cleans the scraped fields.
4. Converts prices from GBP text to numeric `price_gbp`.
5. Converts star ratings from text to numeric `rating`.
6. Converts availability to boolean `in_stock`.
7. Converts GBP prices to INR.
8. Creates the SQLite database and normalized tables.
9. Inserts the cleaned data into SQLite.
10. Executes the required SQL queries.
11. Saves the query results.

## Web Scraping

The `requests` library is used to send HTTP requests to the website, while `BeautifulSoup` parses the returned HTML.

The scraper follows category pagination and collects books from these categories:

* Travel
* Mystery
* Historical Fiction
* Classics
* Science Fiction
* Music
* Business
* Poetry

The scraper does not bypass authentication, CAPTCHAs, paywalls, or other access controls.

## Data Cleaning

### Price

The original price is scraped as text, for example:

```text
£45.17
```

The currency symbol is removed and the value is converted to a floating-point number:

```text
45.17
```

The cleaned value is stored in:

```text
price_gbp
```

### Star Rating

The text ratings are converted as follows:

```text
One   → 1
Two   → 2
Three → 3
Four  → 4
Five  → 5
```

The resulting column is:

```text
rating
```

### Availability

The availability text is converted into a boolean column:

```text
In stock → True
```

The resulting column is:

```text
in_stock
```

### Handling Invalid Values

Unexpected numeric values are converted to missing values using safe numeric parsing rather than causing the pipeline to crash.

Missing numeric values are handled using **median imputation**.

This approach keeps the affected rows while preventing malformed numeric values from stopping the pipeline.

## Currency Conversion

The project uses the required fixed conversion rate:

**1 GBP = 105.50 INR**

The conversion is:

```text
price_inr = price_gbp × 105.50
```

This is an artificial, project-defined baseline for this assignment. It is not a live or historical market exchange rate.

No external currency API or network request is used for this conversion.

## Database Design

The SQLite database contains two normalized tables.

### `categories`

```text
category_id    INTEGER PRIMARY KEY
category_name  TEXT UNIQUE NOT NULL
```

### `books`

```text
book_id       INTEGER PRIMARY KEY
title         TEXT
price_gbp     REAL
price_inr     REAL
rating        INTEGER
in_stock      INTEGER
category_id   INTEGER FOREIGN KEY
```

The relationship is:

```text
categories
    │
    │ category_id
    │
    ▼
books.category_id
```

The category name is stored only once in the `categories` table rather than being repeated for every book.

## SQL Queries

Five SQL queries are executed against the database.

They demonstrate:

1. `SELECT` and `WHERE`
2. `ORDER BY` and `LIMIT`
3. `DISTINCT`
4. `BETWEEN`
5. `JOIN`

The executed query strings and their outputs are saved in the `query_outputs` directory.

## Pandas Verification

At least two SQL query results are read back into pandas using:

```python
pd.read_sql()
```

The SQL `JOIN` result is also independently reproduced using:

```python
pd.merge()
```

The SQL and pandas results are compared to verify that they produce equivalent output.

## Outputs

The pipeline produces:

* `books_dataset.csv` — scraped dataset
* `books_dataset_cleaned.csv` — cleaned and converted dataset
* `books.db` — SQLite database
* `query_outputs/` — outputs from the executed SQL queries

## Design Decisions

* `requests` and `BeautifulSoup` are used for web scraping.
* Pandas is used for cleaning and data manipulation.
* SQLite is used for persistent local storage.
* Categories and books are separated into two tables to maintain a normalized database structure.
* A primary/foreign key relationship connects books to their categories.
* Median imputation is used for invalid numeric values to avoid losing otherwise usable book records.
* The fixed project-defined GBP-to-INR rate of **105.50** is used without an external API.
