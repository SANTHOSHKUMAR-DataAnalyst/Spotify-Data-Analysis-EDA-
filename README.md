🎶 Spotify Data Analysis Using SQL
📋 Objective
This project aims to analyze and explore Spotify track data using SQL. The primary focus is on insights into high-stream tracks, artist productivity, and track characteristics. By cleaning and querying the dataset, we uncover patterns in streaming behavior, popular music attributes, and artist trends to support data-driven insights for decision-making and enhancing user experience on the platform.

📂 Dataset
The dataset contains information on various Spotify tracks, including:

Track name
Artist
Album type
Streams, likes, views, comments
Track features like energy, danceability, and liveness
🔧 Tools Used
SQL: MySQL for data analysis and querying
Data Cleaning: Handling duplicates, standardizing formats, removing null values

⭐ Key Features
1.High-Stream Tracks Analysis: Identification of tracks with more than 1 billion streams.
2.Album and Artist Pairing: Lists unique combinations of albums and their respective artists.
3.Comment Count for Licensed Tracks: Calculation of total comments on tracks with licensing information.
4.Single Track Filtering: Extraction of tracks belonging to single-type albums.
5.Artist Track Count: Provides total track count per artist to gauge productivity.
6.Album Danceability Insights: Calculation of average danceability scores per album.
7.High-Energy Track Identification: Retrieval of the top 5 tracks with the highest energy levels.
8.Official Video Insights: Listing of tracks with associated views and likes for official videos.
9.Album View Totals: Aggregation of total views for each album’s tracks.
10.Platform Streaming Comparison: Identifies tracks with higher streams on Spotify compared to YouTube.
11.Top Tracks per Artist: Ranking of the top 3 most-viewed tracks for each artist using window functions.
12.Above-Average Liveness Tracks: Tracks with a liveness score above the dataset’s average.
13.Energy Range per Album: Calculation of the range between highest and lowest energy levels within each album.
14.Energy-to-Liveness Ratio Analysis: Tracks with energy-to-liveness ratios greater than 1.2.
15Cumulative Likes by Views: Calculation of cumulative likes for tracks ordered by views.
📊 Results
This analysis provides insights into popular music trends and track characteristics on Spotify, 
making it a valuable resource for understanding user preferences and improving data-driven music recommendations.

solutions : 

-- 1.Retrive the names of all tracks that have more than 1 billion streams 

select track 
from spotify
where stream > 1000000000
order by 1 DESC
LIMIT 5;

-- 2. list all albums along with their respective artists

select distinct album,artist
from spotify
where album_type = 'album'
LIMIT 10;

-- 3.get the total number of comments for track where licedsed = 'True'.

select sum(comments) as Total_Comments
from spotify
where Licensed = 'True';

-- 4.find all tracks that belong to the album type single

select track
from spotify
where album_type = 'Single'
LIMIT 10;

-- 5.count the total number of tracks by each artist

select artist,count(track) as Total_No_Songs
from spotify
group by artist
ORDER BY 2
LIMIT 10;

-- 6.calculate the average danceability of tracks in each album.

select album,round(avg(Danceability),6)
from spotify
group by album
ORDER BY 2 DESC
LIMIT 10;

-- 7.find the top 5 tracks with the highest energy values;

select track,max(energyLiveness) as energy
from spotify
group by track
order by energy desc
limit 10;

-- 8.list all tracks along with their views and likes where official_video = True 

select track,sum(views) as total_Views,
	   sum(likes) as total_Likes
from spotify
where official_video = 'True'
group by track
limit 5;

-- 9.for each album,calculate the total views of all associated tracks

select album,track, sum(views)
from spotify
group by album,track
order by 3 DESC
LIMIT 10;

-- 10.retrive the track names that have been streamed on spotify more than youtube.

select * from (
select track ,
coalesce(sum(case when most_playedon = 'Youtube' then stream end),0) as streamed_in_youtube,
coalesce(sum(case when most_playedon = 'Spotify' then stream end),0) as streamed_in_spotify
from spotify  
group by track) santy
where streamed_in_spotify > streamed_in_youtube and streamed_in_youtube <> 0
limit 5;

-- 11.find the top 3 most-viewed tracks for each artist using windows function.

select * from (
select track 
,artist
,dense_rank() over(partition by artist order by sum(views)) as dns_rnk
from spotify 
group by 1,2) dense
where dns_rnk < 4;

-- 12.write a query to find tracks where the liveness score is above the average 

select track,artist,liveness from
(
select avg(liveness) as a
from spotify 
)as average,spotify
where liveness > a;

-- 13.use with clause to calculate the difference between the highest and lowest energy values for tracks in each albums

with cte as 
( 
select album,
	max(energy) as Highest_energy_value,
	min(energy) as Lowest_energy_value 
    from spotify 
group by album
)
select album,
	   Highest_energy_value - Lowest_energy_value as Differnce 
from cte
order by 2 Desc
LIMIT 10;

-- 14.find tracks where the energy-to-liveness ratio is greater than 1.2.

select * from (
select distinct track ,
	   case when liveness = 0 then null 
	   Else energy / liveness end ratio
from spotify ) as r
where ratio > 1.2
order by 2 desc
limit 10;       


-- 15 calcuate the cummulative sum of likes for tracks ordered by the number of views.

with cte as
(
select track
,sum(likes) over(partition by track 
                 order by views desc) as likes
from spotify
)
select track ,likes
from cte
order by 2 desc
LIMIT 10;
















