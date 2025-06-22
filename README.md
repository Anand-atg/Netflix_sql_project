# Netflix Movies and TV Shows Data Analysis using SQL

![Netflix Logo](https://github.com/najirh/netflix_sql_project/blob/main/logo.png)

## 📘 Overview

This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset.

## 🎯 Objectives

* Analyze the distribution of content types (movies vs TV shows).
* Identify the most common ratings for movies and TV shows.
* List and analyze content based on release years, countries, and durations.
* Explore and categorize content based on specific criteria and keywords.

## 📊 Dataset

The data for this project is sourced from the Kaggle dataset:

* **Dataset Link:** [Netflix Movies and TV Shows Dataset](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)

---

## 🧱 Schema

```sql
DROP TABLE IF EXISTS netflix;
CREATE TABLE netflix
(
    show_id      VARCHAR(5),
    type         VARCHAR(10),
    title        VARCHAR(250),
    director     VARCHAR(550),
    casts        VARCHAR(1050),
    country      VARCHAR(550),
    date_added   VARCHAR(55),
    release_year INT,
    rating       VARCHAR(15),
    duration     VARCHAR(15),
    listed_in    VARCHAR(250),
    description  VARCHAR(550)
);
```

### 🔍 Schema Explanation

* Ensures any existing `netflix` table is dropped to avoid conflicts.
* Creates a new `netflix` table with metadata about each show including type (Movie/TV Show), title, director, cast, country, release year, duration, genres, and description.
* This structure supports rich querying for insights.

---

## 💼 Business Problems and Solutions

### 1. Count the Number of Movies vs TV Shows

```sql
SELECT 
    type,
    COUNT(*)
FROM netflix
GROUP BY 1;
```

**Explanation:**
Groups data by `type` and counts the number of items in each group. Helps assess the overall distribution of content types.

---

### 2. Find the Most Common Rating for Movies and TV Shows

```sql
WITH RatingCounts AS (
    SELECT type, rating, COUNT(*) AS rating_count
    FROM netflix
    GROUP BY type, rating
),
RankedRatings AS (
    SELECT type, rating, rating_count,
           RANK() OVER (PARTITION BY type ORDER BY rating_count DESC) AS rank
    FROM RatingCounts
)
SELECT type, rating AS most_frequent_rating
FROM RankedRatings
WHERE rank = 1;
```
## other way to solve
```
SELECT 
	TYPE,
	RATING
FROM 
(
select 
	type,
	rating,
	count(*),
 	rank()over(partition by type order by count(*) DESC)as ranking
from netflix
group by 1,2) AS T

WHERE RANKING = 1

```






**Explanation:**
Counts how often each rating appears per type, ranks them, and returns the top one. Useful to understand audience targeting.

---

### 3. List All Movies Released in a Specific Year (e.g., 2020)

```sql
SELECT * 
FROM netflix
WHERE release_year = 2020;
```

**Explanation:**
Filters the dataset to return all content released in 2020. Useful for year-wise trend analysis.

---

### 4. Find the Top 5 Countries with the Most Content on Netflix

```sql
SELECT * 
FROM (
    SELECT UNNEST(STRING_TO_ARRAY(country, ',')) AS country,
           COUNT(*) AS total_content
    FROM netflix
    GROUP BY 1
) AS t1
WHERE country IS NOT NULL
ORDER BY total_content DESC
LIMIT 5;
```
## without subquery 
```
select  
	UNNEST(STRING_TO_ARRAY(COUNTRY,','))as new_country,
	count (show_id)as total_count 
from netflix
group by new_country 
order by 2 desc
limit 5;
```
**Explanation:**
Splits multiple countries and counts occurrences to find the top 5 countries producing or hosting Netflix content.

---

### 5. Identify the Longest Movie

```sql
SELECT * 
FROM netflix
WHERE type = 'Movie'
ORDER BY SPLIT_PART(duration, ' ', 1)::INT DESC;
```

**Explanation:**
Filters movies and sorts them by numeric duration to find the longest one.

---

### 6. Find Content Added in the Last 5 Years

```sql
SELECT * 
FROM netflix
WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years';
```

**Explanation:**
Converts `date_added` string into a proper date and filters for the past 5 years. Helps identify recent additions.

---

### 7. Find All Movies/TV Shows by Director 'Rajiv Chilaka'

```sql
SELECT * 
FROM (
    SELECT *, UNNEST(STRING_TO_ARRAY(director, ',')) AS director_name
    FROM netflix
) AS t
WHERE director_name = 'Rajiv Chilaka';
```

**Explanation:**
Handles multiple directors, filters content specifically directed by Rajiv Chilaka.

---

### 8. List All TV Shows with More Than 5 Seasons

```sql
SELECT * 
FROM netflix
WHERE type = 'TV Show'
  AND SPLIT_PART(duration, ' ', 1)::INT > 5;
```
##  other way to solve it 
```
SELECT * FROM netflix
WHERE LOWER(TYPE)  = 'tv show'
  AND SPLIT_PART(duration, ' ', 1) ~ '^[0-9]+'
  AND SPLIT_PART(duration, ' ', 1)::INT > 5;
```

**Explanation:**
Checks `duration` (number of seasons) for TV Shows and filters those with more than 5.

---

### 9. Count the Number of Content Items in Each Genre

```sql
SELECT 
    UNNEST(STRING_TO_ARRAY(listed_in, ',')) AS genre,
    COUNT(*) AS total_content
FROM netflix
GROUP BY 1;
```

**Explanation:**
Splits the genre list using commas and counts how many times each genre appears across all content items.

---

### 10. Find Top 5 Years with Highest Average Content Released in India

```sql
SELECT 
    country,
    release_year,
    COUNT(show_id) AS total_release,
    ROUND(
        COUNT(show_id)::numeric /
        (SELECT COUNT(show_id) FROM netflix WHERE country = 'India')::numeric * 100, 2
    ) AS avg_release
FROM netflix
WHERE country = 'India'
GROUP BY country, release_year
ORDER BY avg_release DESC
LIMIT 5;
```

**Explanation:**
Calculates the percentage of India's total content released in each year and returns the top 5 years with the highest average content share.

---

### 11. List All Movies That Are Documentaries

```sql
SELECT * 
FROM netflix
WHERE listed_in LIKE '%Documentaries';
```

**Explanation:**
Filters and lists content where the genre contains "Documentaries".

---

### 12. Find All Content Without a Director

```sql
SELECT * 
FROM netflix
WHERE director IS NULL;
```

**Explanation:**
Lists all content items where the director information is missing.

---

### 13. Find Movies Actor 'Salman Khan' Appeared in the Last 10 Years

```sql
SELECT * 
FROM netflix
WHERE casts LIKE '%Salman Khan%'
  AND release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10;
```

**Explanation:**
Filters content featuring Salman Khan and released within the past 10 years.

---

### 14. Top 10 Actors in Indian Movies

```sql
SELECT 
    UNNEST(STRING_TO_ARRAY(casts, ',')) AS actor,
    COUNT(*)
FROM netflix
WHERE country = 'India'
GROUP BY actor
ORDER BY COUNT(*) DESC
LIMIT 10;
```

**Explanation:**
Identifies the top 10 actors with the most appearances in Netflix content from India.

---

### 15. Categorize Content Based on Keywords 'Kill' and 'Violence'

```sql
SELECT CATEGORY,	
			TYPE,	
			COUNT(SHOW_ID) AS TOTAL_CONTENT
			FROM (
SELECT 
		*,
        CASE 
            WHEN description ILIKE '%kill%' OR description ILIKE '%violence%' THEN 'Bad'
            ELSE 'Good'
        END AS category
    FROM netflix)AS CATEGORY_CONTENT
	GROUP BY 1,2
	ORDER BY 2 DESC

```

**Explanation:**
Scans the content descriptions and categorizes content as 'Bad' if it contains violence-related terms, otherwise 'Good'.

###Findings and Conclusion

Content Distribution: The dataset contains a diverse range of movies and TV shows with varying ratings and genres.
Common Ratings: Insights into the most common ratings provide an understanding of the content's target audience.
Geographical Insights: The top countries and the average content releases by India highlight regional content distribution.
Content Categorization: Categorizing content based on specific keywords helps in understanding the nature of content available on Netflix.

This analysis provides a comprehensive view of Netflix's content and can help inform content strategy and decision-making.
