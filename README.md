# Spotify-data-analysis-using-SQL  End-to-End Data Exploration & Querying

This project is an end-to-end SQL analysis of Spotify track data using PostgreSQL.  
It covers data exploration, easy-to-advanced SQL querying, and performance optimization using indexing and execution plans.

---

##  Project Steps

### **1. Data Exploration**
Before writing SQL queries, the dataset was explored to understand the attributes:

- **Artist** – Performer of the track  
- **Track** – Song name  
- **Album** – Album title  
- **Album_type** – Single / Album  
- **Audio metrics** – danceability, energy, loudness, tempo, etc.

This helped in identifying patterns and planning the analysis structure.

---

### **2. Querying the Data**
Once the data was inserted into PostgreSQL, SQL queries were written at increasing complexity levels.

####  **Easy Queries**
- Tracks with more than **1B streams**
  ```sql
  SELECT track 
  FROM spotify
  WHERE stream>1000000000;

- List all albums with their artists
  ```sql
  SELECT 
  distinct artist,album 
  FROM spotify;
- Total comments where `licensed = TRUE`
  ```sql
  SELECT sum(comments) as total_comments
  FROM spotify 
  WHERE licensed='true'; 
  
- Tracks that belong to the album type **single**
  ```sql
  SELECT track ,album_type
  FROM potify 
  WHERE album_type = 'single;'
  

- Count total tracks per artist
  ```sql
  SELECT  count(track) as total_no_of_tracks,artist
  FROM spotify 
  GROUP BY  artist
  ORDER BY total_no_of_tracks asc;

####  **Medium Queries**
- Average **danceability** of tracks per album
  ```sql
  SELECT avg(danceability),track
  FROM spotify
  GROUP BY  track
  ORDER BY 1 DESC;
  
- Top **5 tracks** with highest energy
  ```sql
  SELECT track,
  max(energy) as maximum_energy
  FROM spotify 
  GROUP BY  track
  ORDER BY max(energy) DESC
  LIMIT 5;
- Tracks with views and likes where `official_video = TRUE`
  ```sql
  SELECT DISTINCT track,
  sum(likes) as total_likes,
  sum(views) as total_views
  FROM spotify 
  WHERE  official_video ='true'
  GROUP BY 1;

- Total views for each album
  ```sql
    SELECT album,track,
  sum(views) as total_no_of_views
  FROM spotify 
  GROUP BY 1 ,2 
  ORDER BY 3 DESC;

- Tracks streamed more on **Spotify than YouTube**
  ```sql
     SELECT * 
  FROM
  (SELECT track,
   COALESCE(SUM(CASE WHEN most_played_on='Spotify' THEN stream end),0) as streamed_on_Spotify,
  COALESCE(SUM(CASE WHEN most_played_on='Youtube' THEN stream end),0) as streamed_on_Youtube
  FROM  spotify
  GROUP BY 1) sub
  WHERE streamed_on_Spotify>streamed_on_Youtube
  AND
  streamed_on_Youtube<>0;


####  **Advanced Queries**
- Top 3 **most-viewed tracks per artist** (window function)
  ```sql
        
  WITH ranking_artist
  AS
  ( 
   SELECT 
   artist,
  track,
  SUM(views) as total_view,
  DENSE_RANK() OVER(PARTITION BY artist ORDER BY SUM(views) DESC ) as rank
  FROM spotify 
  GROUP BY 1,2
  ORDER BY 1,3 DESC
  )
  SELECT * FROM ranking_artist
  WHERE rank<=3;
  
- Tracks with **liveness above average**
   ```sql
     SELECT DISTINCT artist,album,track,most_played_on,liveness
  FROM spotify 
  WHERE liveness >(SELECT AVG(liveness)
				FROM spotify );
				
- Difference between highest & lowest **energy** per album (CTE)
  
```sql
WITH cte
AS
(SELECT 
	album,
	MAX(energy) as highest_energy,
	MIN(energy) as lowest_energery
FROM spotify
GROUP BY 1
)
SELECT 
	album,
	highest_energy - lowest_energery as energy_diff
FROM cte
ORDER BY 2 DESC;
```

##  Query Optimization Technique

To improve query performance in our Spotify SQL Analysis project, we carried out a systematic optimization process focused on indexing and execution planning.

---

###  Initial Query Performance Analysis (Using EXPLAIN)

We first analyzed the performance of a query that retrieved tracks based on the **artist** column.  
The `EXPLAIN` output showed the following metrics:

- **Execution Time (E.T.):** 7 ms  
- **Planning Time (P.T.):** 0.17 ms  


---

###  Index Creation on the `artist` Column

To optimize query speed, we created an index on the `artist` column.  
This helps PostgreSQL quickly locate relevant rows, drastically improving performance.

**SQL Command:**

```sql
CREATE INDEX idx_artist ON spotify_tracks(artist);
