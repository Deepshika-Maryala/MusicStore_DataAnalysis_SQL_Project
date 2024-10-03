# Music Store Data Analysis Using SQL
### Project Overview:
This project aims to analyze the performance and trends of a music store using SQL to extract, analyze, and visualize key insights from the store's database. By leveraging SQL queries, we will dive into various aspects such as customer behavior, sales performance, inventory management, and genre popularity. The primary goal is to uncover meaningful patterns that can help improve decision-making, optimize inventory, and enhance customer satisfaction.
### Data Source:
The dataset used for analysis are “music_store_data” Zip file, containing detail information of the music store(i.e., music, artist, customers, employees, playlists, invoices, etc).
### Tools:
- SQL (Data Analysis) 
- Excel (Data cleaning and Visualization)
### Exploratory Data Analysis:
EDA involved exploring the data to answer the key questions such as:
- Who is the senior most employee based on job title?
- Which countries have the most Invoices?
- What are top 3 values of total invoice?
- Which city has the best customers? We would like to throw a promotional Music Festival in the city we made the most money. Write a query that returns one city that has the highest sum of invoice totals.
- Who is the best customer? The customer who has spent the most money will be declared the best customer. Write a query that returns the person who has spent the most money.
- Let's invite the artists who have written the most rock music in our dataset. Write a query that returns the Artist name and total track count of the top 10 rock bands.
- Find how much amount spent by each customer on artists? Write a query to return customer name, artist name and total spent
- We want to find out the most popular music Genre for each country. We determine the most popular genre as the genre with the highest amount of purchases. Write a query that returns each country along with the top Genre. For countries where the maximum number of purchases is shared return all Genres.
### Data Analysis:
- Who is the senior most employee based on job title? 
```sql
select title, first_name, last_name from employee order by levels desc limit 1;
```
![Screenshot 2024-10-02 235103](https://github.com/user-attachments/assets/8f9514cb-13f4-4cce-8937-b7f9393f07c3)
![Screenshot 2024-10-02 235117](https://github.com/user-attachments/assets/43c45ae2-5a7f-4709-80d2-1a5234f1f2c9)


- Which countries have the most Invoices?
```sql
select count(*) as Country_most_invoices, billing_country from invoice  group by billing_country 
order by Country_most_invoices desc;
```
![Screenshot 2024-10-03 170120](https://github.com/user-attachments/assets/cec10760-e580-455a-a817-5f99539f89ac)
![Screenshot 2024-10-02 235217](https://github.com/user-attachments/assets/9516daa0-2539-4ae8-9cef-17a6ff454c39)


- What are top 3 values of total invoice?
```sql
select total from invoice order by total desc;
```

![Screenshot 2024-10-02 235428](https://github.com/user-attachments/assets/53a077c4-c405-4e39-ac96-f7e1e821dd86)

![Screenshot 2024-10-02 235442](https://github.com/user-attachments/assets/62fb652a-6e07-4343-8feb-5c7aa901cef4)

- Which city has the best customers? We would like to throw a promotional Music Festival in the city we made the most money. Write a query that returns one city that has the highest sum of invoice totals. Return both the city name & sum of all invoice totals  
```sql
select billing_city, sum(total) as Invoice_total from invoice group by billing_city order by Invoice_total 
limit 1;
```

![Screenshot 2024-10-02 235454](https://github.com/user-attachments/assets/478b36bb-4764-467c-98bf-a03883e0d711)
![Screenshot 2024-10-02 235504](https://github.com/user-attachments/assets/b01f0615-4288-42c6-b112-07e54c19fd86)


- Who is the best customer? The customer who has spent the most money will be declared the best customer. Write a query that returns the person who has spent the most money.
```sql
set SQL_MODE ='';
select customer.customer_id, customer.first_name, customer.last_name, sum(invoice.total) as total_spending
from customer 
join invoice on customer.customer_id= invoice.customer_id
group by customer.customer_id
order by total_spending desc LIMIT 1;
```
![Screenshot 2024-10-03 000202](https://github.com/user-attachments/assets/3993a6fa-10d9-46a2-9acc-d738dfdb3ca3)

![Screenshot 2024-10-03 000212](https://github.com/user-attachments/assets/2e4e5c6d-15f0-419a-90c2-92dec89d7eeb)

- Let's invite the artists who have written the most rock music in our dataset. Write a query that returns the Artist name and total track count of the top 10 rock bands.
```sql
select artist.artist_id, artist.name,  count(artist.artist_id) as Number_of_songs
from track
join album2 on album2.album_id= track.album_id
join artist on artist.artist_id= album2.artist_id
join genre on genre.genre_id= track.genre_id
group by artist.artist_id
order by Number_of_songs desc
limit 10;
```
![Screenshot 2024-10-03 000225](https://github.com/user-attachments/assets/d2aee3ba-46cb-4e79-a208-0956770a472d)

![Screenshot 2024-10-03 000459](https://github.com/user-attachments/assets/efa08639-e201-437b-933e-b22d1991afab)


- Find how much amount spent by each customer on artists? Write a query to return customer name, artist name and total spent 
```sql
select concat(c.first_name," ", c.last_name) as customer_name, a.name, 
sum(il.unit_price*il.quantity) as total_sales
from customer c
join invoice i on c.customer_id= i.customer_id
join invoice_line il on i.invoice_id=il.invoice_id
join track t on il.track_id= t.track_id
join album2 al on al.album_id= t.album_id
join artist a on a.artist_id=al.artist_id
group by customer_name
order by total_sales desc
limit 5;
```
![Screenshot 2024-10-03 000742](https://github.com/user-attachments/assets/7d07f074-321f-4046-b0b7-2c936cfc8642)

![Screenshot 2024-10-03 000752](https://github.com/user-attachments/assets/a147f342-6253-496f-9b72-c8fe370a5ce8)

- We want to find out the most popular music Genre for each country. We determine the most popular genre as the genre with the highest amount of purchases. Write a query that returns each country along with the top Genre. For countries where the maximum number of purchases is shared return all Genres.
```sql
WITH popular_genre AS 
(
    SELECT COUNT(invoice_line.quantity) AS purchases, customer.country, genre.name, genre.genre_id, 
	ROW_NUMBER() OVER(PARTITION BY customer.country ORDER BY COUNT(invoice_line.quantity) DESC) AS RowNo 
    FROM invoice_line 
	JOIN invoice ON invoice.invoice_id = invoice_line.invoice_id
	JOIN customer ON customer.customer_id = invoice.customer_id
	JOIN track ON track.track_id = invoice_line.track_id
	JOIN genre ON genre.genre_id = track.genre_id
	GROUP BY 2,3,4
	ORDER BY 2 ASC, 1 DESC
)  
SELECT * FROM popular_genre WHERE RowNo <= 1
```


![Screenshot 2024-10-03 000829](https://github.com/user-attachments/assets/e17900e4-cfeb-48ef-b9eb-a81a577c41ae)


![Screenshot 2024-10-03 000906](https://github.com/user-attachments/assets/d8cc0048-bde1-46d8-8190-6f61448bf4f2)


### Insights and Findings:
- Number of employees are less at senior level as compared to junior level.
- Maximum customers are from  Europe only( ~70%).
- Maximum total invoice amount is from Prague city also top two customers in terms of money they spend also belongs to the this city.
- Rock is the only music genre which is popular in all country except Argentina in terms of purchase.
### Recommendations:
- Focus on maintaining adequate inventory levels of the best-selling products and categories. Ensure high-demand items are never out of stock.
- Organize events such as live music sessions, artist Q&A, or album release parties in the countries where maximum customers.
### Project Phases and Milestones:
- Data Analysis and Visualization: April 2024- May 2024












