# 🎬 Netflix Content SQL Analysis

## Project Overview

This project analyzes **Netflix's content library** using SQL to extract business insights from a dataset covering movies, TV shows, genres, countries, ratings, and release trends.

> Dataset: 8,800+ titles · 100+ countries · 2008 – 2021

---

## Dataset Schema

```
netflix_titles
──────────────────────────────────────────────────────
show_id       TEXT      -- Unique ID for each title
type          TEXT      -- 'Movie' or 'TV Show'
title         TEXT      -- Title name
director      TEXT      -- Director name(s)
cast          TEXT      -- Cast members (comma-separated)
country       TEXT      -- Country of production
date_added    TEXT      -- Date added to Netflix
release_year  INTEGER   -- Year of original release
rating        TEXT      -- Content rating (PG, R, TV-MA, etc.)
duration      TEXT      -- '90 min' or '2 Seasons'
listed_in     TEXT      -- Genres (comma-separated)
description   TEXT      -- Short description
```

---

## Business Questions

| # | Business Question |
|---|---|
| 1 | What is the overall content breakdown? |
| 2 | How has Netflix grown its content over the years? |
| 3 | Which countries produce the most content? |
| 4 | What genres dominate the platform? |
| 5 | Who are the most featured directors and actors? |
| 6 | What are the YoY growth trends? |

---

## Section 1 — Overview KPIs

> **Business Question:** What is the overall health and breakdown of Netflix content?

### 1.1 Total Titles, Movies vs TV Shows

```sql
SELECT
    COUNT(*)                                              AS total_titles,
    COUNT(CASE WHEN type = 'Movie'   THEN 1 END)         AS total_movies,
    COUNT(CASE WHEN type = 'TV Show' THEN 1 END)         AS total_tvshows,
    ROUND(COUNT(CASE WHEN type = 'Movie' THEN 1 END) * 100.0
          / COUNT(*), 2)                                 AS movie_pct,
    ROUND(COUNT(CASE WHEN type = 'TV Show' THEN 1 END) * 100.0
          / COUNT(*), 2)                                 AS tvshow_pct
FROM netflix_titles;
```




**Result:**

Netflix focuses on movies, accounting for nearly 70% of all content, but TV shows also make up 30% because they retain engagement for longer periods, as viewers tend to return for multiple episodes.

---


### 1.2 Rating Distribution

```sql
SELECT
    rating,
    COUNT(*)                                              AS total_titles,
    ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) 
          FROM netflix_titles), 2)                       AS pct
FROM netflix_titles
WHERE rating IS NOT NULL
GROUP BY rating
ORDER BY total_titles DESC;
```

---

## Section 2 — Content Trend Analysis

> **Business Question:** How has Netflix grown its content library over the years?

### 2.1 Titles Added per Year

```sql
SELECT
    SUBSTR(date_added, -4)                               AS year_added,
    COUNT(*)                                             AS total_titles,
    COUNT(CASE WHEN type = 'Movie'   THEN 1 END)         AS movies,
    COUNT(CASE WHEN type = 'TV Show' THEN 1 END)         AS tvshows
FROM netflix_titles
WHERE date_added IS NOT NULL
GROUP BY year_added
ORDER BY year_added;
```

---

### 2.2 Month with Most Content Added

```sql
SELECT
    SUBSTR(date_added, 1, INSTR(date_added, ' ') - 1)   AS month_added,
    COUNT(*)                                             AS total_titles
FROM netflix_titles
WHERE date_added IS NOT NULL
GROUP BY month_added
ORDER BY total_titles DESC;
```

This reveals seasonal patterns in Netflix's content acquisition strategy.

---

## Section 3 — Country Analysis

> **Business Question:** Which countries produce the most content on Netflix?

### 3.1 Top 10 Countries by Content

```sql
SELECT
    TRIM(country)                                        AS country,
    COUNT(*)                                             AS total_titles,
    COUNT(CASE WHEN type = 'Movie'   THEN 1 END)         AS movies,
    COUNT(CASE WHEN type = 'TV Show' THEN 1 END)         AS tvshows,
    ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) 
          FROM netflix_titles 
          WHERE country IS NOT NULL), 2)                 AS pct
FROM netflix_titles
WHERE country IS NOT NULL
  AND INSTR(country, ',') = 0
GROUP BY TRIM(country)
ORDER BY total_titles DESC
LIMIT 10;
```

---

## Section 4 — Genre Analysis

> **Business Question:** What genres dominate the Netflix platform?

### 4.1 Top Genres

```sql
WITH genre_split AS (
    SELECT
        show_id,
        type,
        TRIM(SUBSTR(listed_in, 1,
            CASE WHEN INSTR(listed_in, ',') > 0
                 THEN INSTR(listed_in, ',') - 1
                 ELSE LENGTH(listed_in) END))            AS genre
    FROM netflix_titles
    UNION ALL
    SELECT
        show_id,
        type,
        TRIM(SUBSTR(listed_in,
            INSTR(listed_in, ',') + 1))                  AS genre
    FROM netflix_titles
    WHERE INSTR(listed_in, ',') > 0
)
SELECT
    genre,
    COUNT(*)                                             AS total_titles,
    COUNT(CASE WHEN type = 'Movie'   THEN 1 END)         AS movies,
    COUNT(CASE WHEN type = 'TV Show' THEN 1 END)         AS tvshows
FROM genre_split
WHERE genre != ''
GROUP BY genre
ORDER BY total_titles DESC
LIMIT 15;
```

---

## Section 5 — Directors & Cast

> **Business Question:** Who are the most featured directors and actors on Netflix?

### 5.1 Top 10 Directors

```sql
SELECT
    TRIM(director)                                       AS director,
    COUNT(*)                                             AS total_titles,
    COUNT(CASE WHEN type = 'Movie'   THEN 1 END)         AS movies,
    COUNT(CASE WHEN type = 'TV Show' THEN 1 END)         AS tvshows
FROM netflix_titles
WHERE director IS NOT NULL
  AND INSTR(director, ',') = 0
GROUP BY TRIM(director)
ORDER BY total_titles DESC
LIMIT 10;
```

---

## Section 6 — Advanced Analytics

> **Business Question:** What are the running totals and year-over-year growth trends?

### 6.1 Running Total of Titles Added per Year

```sql
SELECT
    year_added,
    yearly_titles,
    SUM(yearly_titles) OVER (
        ORDER BY year_added
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    )                                                    AS running_total
FROM (
    SELECT
        SUBSTR(date_added, -4)                           AS year_added,
        COUNT(*)                                         AS yearly_titles
    FROM netflix_titles
    WHERE date_added IS NOT NULL
    GROUP BY year_added
)
ORDER BY year_added;
```

Running totals show the cumulative scale of Netflix's content investment over time.

---

### 6.2 Year-over-Year Growth Rate

```sql
SELECT
    year_added,
    yearly_titles,
    prev_year_titles,
    CASE
        WHEN prev_year_titles IS NULL THEN NULL
        ELSE ROUND((yearly_titles - prev_year_titles) * 100.0
                   / prev_year_titles, 2)
    END                                                  AS yoy_growth_pct
FROM (
    SELECT
        SUBSTR(date_added, -4)                           AS year_added,
        COUNT(*)                                         AS yearly_titles,
        LAG(COUNT(*)) OVER (
            ORDER BY SUBSTR(date_added, -4)
        )                                                AS prev_year_titles
    FROM netflix_titles
    WHERE date_added IS NOT NULL
    GROUP BY year_added
)
ORDER BY year_added;
```

---

## SQL Techniques Used

| Category | Techniques Used | Used In |
|---|---|---|
| Window Functions | `LAG()`, `SUM() OVER()`, `RANK()` | YoY Growth, Running Totals |
| CTEs | `WITH ... AS ()`, multi-step CTEs | Genre Split, Advanced Analytics |
| Aggregate Functions | `COUNT(DISTINCT)`, `ROUND()`, `SUM()` | KPI Summary, Country Analysis |
| String Functions | `SUBSTR()`, `INSTR()`, `TRIM()` | Date Parsing, Genre Split |
| Conditional Logic | `CASE WHEN`, `NULLIF()` | Type Breakdown, Growth Rate |
| JOINs | `INNER JOIN`, `LEFT JOIN` | Multi-table analyses |

---

*Dataset source: [Netflix Movies and TV Shows — Kaggle](https://www.kaggle.com/datasets/shivamb/netflix-shows)*
