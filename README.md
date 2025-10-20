# Guild Take-Home Project – Principal Data Engineer

This project processes *The Movies Dataset* to create analytics-ready tables for analyzing movie genres and production companies.  
It includes a Python-based ETL pipeline, reproducible SQL models, and validation queries using DuckDB.

---

## 1 Setup

```bash
cd ~/Projects
unzip ~/Downloads/guild_takehome_project_v14.zip -d guild_takehome_project
cd guild_takehome_project

python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

---

## 2 Run the ETL Pipeline
The ETL pipeline accepts either an S3 URI or a local file path as input, automatically downloading S3 files using Boto3 before processing.

```bash
python src/main.py s3://com.guild.us-west-2.public-data/project-data/the-movies-dataset.zip \
  --out ./output --log ./error.log

> Alternatively, you can provide a local file path:
> Ensure `the-movies-dataset.zip` is placed in the **project root** (same folder as `src/`, `sql/`, `docs/`).

python src/main.py the-movies-dataset.zip --out ./output --log ./error.log

```
This generates six analytics-ready tables under `./output/`:

```
tblMovie.csv
tblGenre.csv
tblCompany.csv
tblMovieGenres.csv
tblMovieCompanies.csv
tblMovieMetrics.csv
```

---

### About the Data Transformation Program

The ETL pipeline implemented in `src/main.py` and `src/transform.py` transforms *The Movies Dataset* into analytics-ready tables that support all required queries.  

- **Extract:** Reads compressed CSVs from `the-movies-dataset.zip`.  
- **Transform:**  
  - Parses genre and company JSON arrays.  
  - Derives `release_year` from `release_date`.  
  - Cleans invalid IDs, null values, and missing genre tags.  
  - Computes `profit = revenue - budget`.  
- **Load:**  
  Writes six normalized CSVs (`tblMovie`, `tblGenre`, `tblCompany`, `tblMovieGenres`, `tblMovieCompanies`, `tblMovieMetrics`) into the `/output` directory.  
- **Validate:**  
  The `sql/*.sql` queries verify that the data model answers each required analytical question.  

This implementation fulfills **Deliverable #2 – Implement a program that transforms the input data into a form usable by the data model.**

---
## 3 Validation Queries (DuckDB)

After generating outputs, you can validate the model using DuckDB from the terminal.  
Each query auto-registers CSVs from `./output/` as tables and prints the results.

### Movie Genre Details
```bash
python src/run_query.py sql/most_popular_genre_by_year.sql
python src/run_query.py sql/budget_by_genre_by_year.sql
python src/run_query.py sql/revenue_by_genre_by_year.sql
python src/run_query.py sql/profit_by_genre_by_year.sql
```

### Production Company Details
```bash
python src/run_query.py sql/budget_by_company_by_year.sql
python src/run_query.py sql/revenue_by_company_by_year.sql
python src/run_query.py sql/profit_by_company_by_year.sql
python src/run_query.py sql/releases_by_genre_per_company_per_year.sql
python src/run_query.py sql/average_popularity_by_company_by_year.sql
```

All queries sort results by their **primary metric (descending)** within each `release_year`, so each report clearly surfaces top performers.

---

### Why “Most Popular Genre by Year” returns one row per year

The prompt asks for *most popular genre by year* (singular).  
This query ranks genres by average popularity within each year and returns only the top one (`ROW_NUMBER() = 1`).  
To view all genres per year, simply remove the `WHERE rn = 1` filter.

---

## 4 Control the Number of Rows Displayed

`run_query.py` supports the `--limit` argument:

```bash
# Default (10 rows)
python src/run_query.py sql/revenue_by_genre_by_year.sql

# Top 50 rows
python src/run_query.py sql/revenue_by_genre_by_year.sql --limit 50

# All rows
python src/run_query.py sql/revenue_by_genre_by_year.sql --limit 0
```

---

## 5 Documentation

- `docs/data_model.md` – table definitions and modeling decisions  
- `docs/erd.png` – Entity-Relationship Diagram  
- `docs/design_scaling_ai.md` – scaling strategy, backfills, monitoring, and AI/ML plan  

---

## 6 Deliverables

```
src/
sql/
docs/
output/
requirements.txt
README.md
error.log (optional)
```

---

## Notes & Assumptions

1. **Movies with missing or invalid data**  
   • Non-numeric `movie_id` rows are dropped.  
   • Only movies with valid `release_year` are analyzed.  

2. **Numeric formatting**  
   • Integer-like fields are displayed as integers (no “.0” suffix).  

3. **Genre filtering**  
   • Only movies with genre tags are included.  
   • Ensures company lists remain consistent across metrics.  

4. **Join strategy**  
   • All joins are `INNER JOIN`; no `LEFT JOIN` used.  

5. **Sorting**  
   • Each query orders results by the **main metric (descending)** within each year   
     (e.g., `ORDER BY release_year, SUM(f.revenue) DESC`).  

6. **Result display**  
   • The `--limit` flag controls how many rows are shown (default 10; `0 = all`).  

---

## Submission Summary

- All SQL validated and syntax-safe  
- `ORDER BY` uses raw aggregates (no alias errors)  
- Supports `--limit` parameter  
- Clean integer output  
- Genre-less movies excluded  
- Sorted results by key metric per year  
- “Most Popular Genre” correctly returns one per year  
- Full documentation for review and submission
