# 🎵 Spotify Dataset SQL Analysis

![Spotify's](https://storage.googleapis.com/pr-newsroom-wp/1/2023/12/Generic-FTR-headers_V10-1920x733.jpg)

## 📄 Overview
This project analyzes the Spotify dataset to uncover trends in music popularity, audio features, and user behavior. Using SQL, we explore patterns in song characteristics, regional popularity, and other key insights. The queries range from simple data retrieval to advanced analytical techniques, showcasing the power of SQL for music data analysis.

---

## 📂 Dataset

Access the dataset here: [Spotify’s Dataset on Kaggle](https://www.kaggle.com/datasets/lehaknarnauli/spotify-datasets).

## 🔍 SQL Query Analysis

### 1. Retrieve All Data
```
SELECT *
FROM [tracks]
SELECT *
FROM [artists];
```

#### 2.Modify Column Data Types
```
ALTER TABLE [tracks]
ALTER COLUMN popularity INT;

ALTER TABLE [tracks]
ALTER COLUMN duration_ms INT;

ALTER TABLE [tracks]
ALTER COLUMN [explicit] INT;

UPDATE [tracks]
SET release_date = CONCAT(release_date, '-01-01')
WHERE LEN(release_date) = 4 AND ISNUMERIC(release_date) = 1;
ALTER TABLE [tracks] ADD release_date_temp DATE;
UPDATE [tracks]
SET release_date_temp = TRY_CAST(release_date AS DATE);
ALTER TABLE [tracks] DROP COLUMN release_date;
EXEC sp_rename 'tracks.release_date_temp', 'release_date', 'COLUMN';
ALTER TABLE [tracks] ALTER COLUMN release_date DATE;

ALTER TABLE [tracks]
ALTER COLUMN danceability FLOAT;

ALTER TABLE [tracks]
ALTER COLUMN energy FLOAT;

ALTER TABLE [tracks]
ALTER COLUMN [key] INT;

ALTER TABLE [tracks]
ALTER COLUMN loudness FLOAT;

ALTER TABLE [tracks]
ALTER COLUMN mode INT;

ALTER TABLE [tracks]
ALTER COLUMN speechiness FLOAT;

ALTER TABLE [tracks]
ALTER COLUMN acousticness FLOAT;

ALTER TABLE [tracks]
ALTER COLUMN instrumentalness FLOAT;

ALTER TABLE [tracks]
ALTER COLUMN liveness FLOAT;

ALTER TABLE [tracks]
ALTER COLUMN valence FLOAT;

ALTER TABLE [tracks]
ALTER COLUMN tempo FLOAT;

ALTER TABLE [tracks]
ALTER COLUMN time_signature INT;

ALTER TABLE [artists]
ALTER COLUMN followers FLOAT;

ALTER TABLE [artists]
ALTER COLUMN popularity INT;
```

#### 3.Top 10 Most Popular Tracks
```
SELECT TOP 10 name, popularity, artists
FROM [tracks]
ORDER BY popularity DESC;
```

#### 4.Tracks Containing the Word "Love"
```
SELECT name
FROM [tracks]
WHERE name LIKE '%Love%' OR name LIKE '%love%';
```

#### 5.Tracks Released Before 2000
```
SELECT * 
FROM [tracks]
WHERE release_date < '2000-01-01';
```

#### 6.Join Tracks and Artists for Popularity > 50
```
SELECT a.name AS artist_name,
       t.name AS track_name,
       t.popularity AS track_popularity
FROM [artists] a
JOIN [tracks] t
ON a.id = t.id_artists
WHERE t.popularity > 50;
```

#### 7.Average Popularity of Tracks by Artist
```
SELECT AVG(popularity) AS avg_popularity, artists
FROM [tracks]
GROUP BY artists
ORDER BY avg_popularity DESC;
```

#### 8.Artists with More Than 5 Tracks
```
SELECT COUNT(name) AS num_tracks, artists
FROM [tracks]
GROUP BY artists
HAVING COUNT(name) > 5;
```

#### 9.Artists with More Than 5 Tracks
```
SELECT COUNT(name) AS num_tracks, artists
FROM [tracks]
GROUP BY artists
HAVING COUNT(name) > 5;
```

#### 10.Top 3 Tracks by Popularity Per Artist
```
WITH CTE AS (
    SELECT *,
           ROW_NUMBER() OVER (PARTITION BY artists ORDER BY popularity DESC) AS rank
    FROM [tracks]
)
SELECT *
FROM CTE
WHERE rank <= 3
ORDER BY artists;
```

#### 11.Explicit vs. Non-Explicit Tracks
```
SELECT COUNT(name) AS track_count, [explicit]
FROM [tracks]
GROUP BY [explicit];
```

#### 12.Tracks Released in 2020 (CTE)
```
WITH CTE AS (
    SELECT *
    FROM [tracks]
    WHERE release_date BETWEEN '2020-01-01' AND '2020-12-31'
)
SELECT AVG(danceability) AS avg_danceability,
       AVG(energy) AS avg_energy,
       AVG(valence) AS avg_valence
FROM CTE;
```

#### 13.Cumulative Popularity by Artist
```
SELECT name, artists,
       SUM(popularity) OVER (PARTITION BY artists ORDER BY release_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS cumulative_popularity,
       popularity
FROM [tracks];
```

#### 14.Artists with Danceability > 0.8 and Loudness < -10
```
WITH CTE AS (
    SELECT artists,
           SUM(CASE WHEN danceability > 0.8 AND loudness < -10 THEN 1 ELSE 0 END) AS valid_tracks
    FROM [tracks]
    GROUP BY artists
)
SELECT artists
FROM CTE
WHERE valid_tracks > 0;
```

#### 14.Artists with No Tracks Above Popularity 10
```
WITH CTE AS (
    SELECT artists,
           MAX(popularity) AS max_popularity
    FROM [tracks]
    GROUP BY artists
)
SELECT artists
FROM CTE
WHERE max_popularity <= 10;
```

#### 15.Extract Unique Genres and Count Occurrences
```
WITH SplitGenres AS (
    SELECT TRIM(value) AS genre
    FROM [artists]
    CROSS APPLY STRING_SPLIT(genres, ',')
)
SELECT genre,
       COUNT(*) AS occurrence
FROM SplitGenres
GROUP BY genre
ORDER BY occurrence DESC;
```

#### 16.Artists with Highest Average Track Duration (Post-2000)
```
SELECT artists, AVG(duration_ms) AS avg_duration
FROM [tracks]
WHERE release_date > '2000-01-01'
GROUP BY artists
ORDER BY avg_duration DESC;
```

#### 17.Top Tracks by Popularity in Each Decade
```
WITH CTE1 AS (
    SELECT name, popularity, release_date,
           (DATEPART(YEAR, release_date) / 10) * 10 AS decade
    FROM [tracks]
),
CTE2 AS (
    SELECT *, 
           ROW_NUMBER() OVER (PARTITION BY decade ORDER BY popularity DESC) AS rank
    FROM CTE1
)
SELECT name, decade
FROM CTE2
WHERE rank = 1;
