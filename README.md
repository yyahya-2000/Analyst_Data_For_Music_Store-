# Analyst_Data_For_Music_Store-
# Objective
You are the lead data analyst for a popular music store. Help them analyze their sales and service!

# --Which tracks appeared in the most playlists? how many playlist did they appear in?
WITH temp_table AS (
  SELECT tracks.TrackId  AS 'track_id',tracks.Name AS 'name_' 
	FROM playlist_track
  JOIN tracks ON tracks.TrackId=playlist_track.TrackId
)
SELECT temp_table.track_id , temp_table.name_,COUNT(temp_table.name_) AS
  'Number of playlists a track appears in' FROM temp_table
GROUP BY 1
ORDER BY 3 DESC;


# --TOP 10 track generated the most revenue, which album? which genre?
WITH
 temp_table AS(
    SELECT tracks.TrackId AS 'Track_Id', tracks.Name AS 'Track_Name', 
	(SELECT albums.Title FROM albums WHERE albums.AlbumId=tracks.AlbumId) AS 'Album_Name',
	(SELECT genres.Name FROM genres WHERE genres.GenreId=tracks.GenreId) AS 'Genre_Name'
	FROM tracks
),
 temp_sum AS(
	SELECT invoice_items.TrackId AS 'TrackId_1',SUM(invoices.Total) AS 'Total_'
	FROM invoice_items 
	JOIN invoices ON invoices.InvoiceId=invoice_items.InvoiceId
	GROUP BY 1
	ORDER BY 2 DESC
 )

 SELECT temp_table.Track_Id,temp_table.Track_Name,temp_table.Album_Name,
	temp_table.Genre_Name, temp_sum.Total_
	FROM temp_table
 JOIN temp_sum ON temp_table.Track_Id=temp_sum.TrackId_1
 ORDER BY 5 DESC
 LIMIT 10;
 
 
 # -- Which countries have the highest sales revenue? What percent of total revenue does each country make up?
SELECT invoices.BillingCity,SUM(invoice_items.UnitPrice) AS 'Total Country Sales', 
	ROUND((SUM(invoice_items.UnitPrice) * 100) / (SELECT SUM(invoice_items.UnitPrice) 
	FROM invoice_items),2) AS 'Percent of total revenue'
FROM invoices 
JOIN invoice_items ON invoice_items.InvoiceId = invoices.InvoiceId
GROUP BY 1
ORDER BY 3 DESC
LIMIT 10;--it can be what ever user wants!


# --How many customers did each employee support, what is the average revenue for each sale, and what is their total sale?
WITH Sum_Avg_Total AS(
	SELECT CustomerId, SUM(Total) AS 'total_sale' 
	FROM invoices
	GROUP BY 1
)
SELECT employees.EmployeeId,employees.FirstName,employees.LastName,
	COUNT(customers.CustomerId) AS 'Number of customers employee support', 
	ROUND(AVG(Sum_Avg_Total.total_sale),2) AS 'AVG_Sale', SUM(Sum_Avg_Total.total_sale) 
	AS 'Total_Sale' 
FROM employees 
LEFT JOIN customers ON employees.EmployeeId = customers.SupportRepId
LEFT JOIN Sum_Avg_Total ON Sum_Avg_Total.CustomerId = customers.CustomerId
GROUP BY 1
ORDER BY 6 DESC, 4 DESC;


# --Do longer or shorter length albums tend to generate more revenue?
WITH revenue_per_track AS(
	SELECT tracks.TrackId, tracks.AlbumId,tracks.Milliseconds AS 'Milliseconds',
		SUM(invoice_items.Quantity * invoice_items.UnitPrice) 'Revenue'
	FROM tracks 
	JOIN invoice_items ON invoice_items.TrackId = tracks.TrackId
	GROUP BY 1	
)
SELECT albums.AlbumId, albums.Title, SUM(revenue_per_track.Milliseconds) / 1000 / 60 AS
	'Length_Albums_Min', SUM(revenue_per_track.Revenue) AS 'Album_Revenue'
FROM albums 
JOIN revenue_per_track ON albums.AlbumId = revenue_per_track.AlbumId
GROUP BY 1
ORDER BY 4 DESC;


# --Is the number of times a track appear in any playlist a good indicator of sales?
WITH tracks_lists AS (
	SELECT tracks.TrackId, COUNT(playlist_track.TrackId) AS 
		'Times_Appearing_In_Playlist', playlists.Name
	FROM tracks 
	JOIN playlist_track ON tracks.TrackId = playlist_track.TrackId
	LEFT JOIN playlists ON playlists.PlaylistId = playlist_track.PlaylistId
	GROUP BY 1
)
SELECT tracks.TrackId, tracks.Name ,tracks_lists.Name AS 
	'Name_Of_Playlist', tracks_lists.Times_Appearing_In_Playlist, 
	SUM(invoice_items.UnitPrice) AS 'Sales'
FROM tracks
LEFT JOIN tracks_lists ON tracks_lists.TrackId = tracks.TrackId
LEFT JOIN invoice_items ON invoice_items.TrackId = tracks.TrackId
GROUP BY 1
ORDER BY 5 DESC;
--Answer is no
--in the table I added the name of the playlist because it will help the result to be clearer


# --How much revenue is generated each year, and what is its percent change from the previous year?
WITH 
this_year AS(
	SELECT STRFTIME('%Y',InvoiceDate) AS 'This_Year', SUM(invoice_items.UnitPrice) AS
		'Total_Sales' 
	FROM invoices
	JOIN invoice_items ON invoices.InvoiceId=invoice_items.InvoiceId
	GROUP BY 1
),
prev_year AS(
	SELECT STRFTIME('%Y',InvoiceDate) +1 AS 'Prev_Year', SUM(invoice_items.UnitPrice) AS
		'Total_Sales' 
	FROM invoices
	JOIN invoice_items ON invoices.InvoiceId=invoice_items.InvoiceId
	GROUP BY 1
)
SELECT this_year.This_Year, prev_year.Prev_Year, 
	((this_year.Total_Sales - prev_year.Total_Sales) / prev_year.Total_Sales * 100) AS 
	'Percent_Change' FROM this_year
LEFT JOIN prev_year ON CAST(this_year.This_Year AS INTEGER) = CAST(prev_year.Prev_Year AS INTEGER);
