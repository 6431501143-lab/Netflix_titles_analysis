# 🎬 Netflix Content SQL Analysis

---

## Project Overview

This project analyzes **Netflix's content library** using SQL and Python to extract business insights from a dataset covering movies, TV shows, genres, countries, ratings, and release trends.

> Dataset: 8,807 titles · 100+ countries · 2008 – 2021

---

## Business Questions

| # | Business Question |
|---|---|
| 1 | What is the overall content breakdown? |
| 2 | How has Netflix grown its content over the years? |
| 3 | Which countries produce the most content? |
| 4 | What genres dominate the platform? |
| 5 | Who are the most featured directors on Netflix? |
| 6 | What is the content rating distribution? |

---

## Dataset Schema

```
netflix_titles
──────────────────────────────────────────────────────────
show_id        TEXT     -- Unique ID for each title
type           TEXT     -- 'Movie' or 'TV Show'
title          TEXT     -- Title name
director       TEXT     -- Director name(s)
cast           TEXT     -- Cast members (comma-separated)
country        TEXT     -- Country of production
date_added     TEXT     -- Date added to Netflix
release_year   INTEGER  -- Year of original release
rating         TEXT     -- Content rating (PG, R, TV-MA, etc.)
duration       TEXT     -- '90 min' or '2 Seasons'
listed_in      TEXT     -- Genres (comma-separated)
description    TEXT     -- Short description
```

---

## Key Findings

### 🎥 Content Overview
- Total titles: **8,807** — **6,131 Movies (69.6%)** and **2,676 TV Shows (30.4%)**
- Netflix heavily favors Movies over TV Shows at nearly a **7:3 ratio**

### 📈 Growth Trend
- Content additions surged from **429 titles in 2016** to a peak of **2,016 titles in 2019**
- Growth slowed in 2020–2021, likely impacted by COVID-19 production delays
- Highest YoY growth: **+423%** in 2016 — Netflix's aggressive global expansion phase

### 🌍 Geography
- **United States** dominates with **2,818 titles (35.3%)** of all content
- **India** is a strong #2 with **972 titles (12.2%)** — reflecting Netflix's push into South Asia
- **South Korea** ranks #5, confirming the global rise of K-content

### 🎭 Genres
- **International Movies** and **Dramas** are the top genres on the platform
- TV Shows skew heavily toward **International TV Shows** and **Crime TV Shows**

### 🔞 Ratings
- **TV-MA** accounts for **36.4%** of all content — Netflix targets adult audiences
- **TV-14** adds another **24.5%** — combined, mature content makes up over 60% of the library
- Family-friendly content (TV-G, TV-Y, PG) represents less than **10%** of titles

### 🎬 Directors
- **Rajiv Chilaka** leads with 19 titles — primarily children's animation (Chhota Bheem)
- Hollywood legends **Martin Scorsese** and **Steven Spielberg** both appear in the Top 10

---

Dataset source: [Netflix Movies and TV Shows — Kaggle](https://www.kaggle.com/datasets/shivamb/netflix-shows)
