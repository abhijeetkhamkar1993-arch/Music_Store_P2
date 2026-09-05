# Music_Store_P2
This is a SQL database end to end project and some query's... 
----------------------------MUSIC STORE DB PROJECT--------------------

1. Who is the senior most employee based on job title?

select * from employee
order by levels desc
limit 1;

2. Which countries have the most Invoices?

select count(invoice_id) as totalc, billing_country from invoice
group by billing_country
order by totalc desc

3. What are top 3 values of total invoice?

select * from invoice
order by total desc
limit 3


4. Which city has the best customers? We would like to throw a promotional Music Festival in the city we made the
 most money. Write a query that returns one city that has the highest sum of invoice totals. Return both the city name 
 & sum of all invoice totals
 
select sum(total) as invctotal, billing_city from invoice
group by billing_city
order by invctotal desc
 
5. Who is the best customer? The customer who has spent the most money will be declared the best customer.
 Write a query that returns the person who has spent the most money

select c.customer_id, c.first_name,c.last_name, sum(i.total) as total  from customer as c inner join invoice as i on
c.customer_id=i.customer_id
group by c.customer_id
order by total desc 
limit 1


--------------------------------Question Set 2 – Moderate----------------------------------------------

1. Write query to return the email, first name, last name, & Genre of all Rock Music listeners. Return 
your list ordered alphabetically by email starting with A


select c.email, c.first_name, c.last_name from customer as c 
join invoice as i on c.customer_id=i.customer_id
join invoice_line on i.invoice_id=invoice_line.invoice_id
where track_id IN
(select track_id from track
join genre on track.genre_id= genre.genre_id
where genre.name LIKE 'Rock')
order by email;


--2. Let's invite the artists who have written the most rock music in our dataset. 
--Write a query that returns the Artist name and total track count of the top 10 rock bands

select artist.artist_id,artist.name, count(artist.artist_id) as numberofsong from track
join album on track.album_id=album.album_id
join artist on album.artist_id=artist.artist_id 
join genre on track.genre_id=genre.genre_id
where genre.name LIKE 'Rock'
group by artist.artist_id
order by numberofsong desc
limit 10

--3. Return all the track names that have a song length longer than the average song length. 
Return the Name and Milliseconds for each track. Order by the song length with the longest songs listed first

select name, milliseconds from track
where milliseconds >
(select avg(milliseconds) as avgmill from track)
order by milliseconds desc


----------------------Question Set 3 – Advance--------------------------------------------

1. Find how much amount spent by each customer on artists? Write a query to return customer name, 
artist name and total spent


WITH best_selling_artist AS (
select artist.artist_id as atist_id, artist.name as artist_name,
sum(invoice_line.unit_price * invoice_line.quantity) as totalspent from invoice_line 
join track on invoice_line.track_id=track.track_id
join album on track.album_id=album.album_id
join artist on album.artist_id=artist.artist_id
group by 1
order by 3 desc
limit 1
)
select c.customer_id,c.customer_name, bsa.artist_name,
sum(il.unit_price * il.quantity) as total_spent from invoice i
join customer c on c.customer_id=i.customer_id
join invoice_line il on il.invoice_id=i.invoice_id
join track t on il.track_id=t.track_id
join album ab on ab.album_id=t.album_id
join best_selling_artist bsa on bsa.artist_id=ab.artist_id
group by 1,2,3,4
order by 5 desc;



2. We want to find out the most popular music Genre for each country. 
We determine the most popular genre as the genre with the highest amount of purchases. 
Write a query that returns each country along with the top Genre. 
For countries where the maximum number of purchases is shared return all Genres

WITH RECURSIVE Sale_per_country AS(
select count(*) as purchasepergenre , customer.country, genre.name, genre.genre_id from invoice_line
join invoice on invoice.invoice_id=invoice_line.invoice_id
Join customer on customer.customer_id=invoice.customer_id
Join track on track.track_id=invoice_line.track_id
join genre on genre.genre_id=track.genre_id
group by 2,3,4
order by 2
),
max_genre_per_country as (select max(purchasepergenre) as max_genre_number , country 
from Sale_per_country
group by 2
order by 2)

select Sale_per_country.* from Sale_per_country
join max_genre_per_country on Sale_per_country.country=max_genre_per_country.country
where Sale_per_country.purchasepergenre = max_genre_per_country.max_genre_number
