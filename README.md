# FamousPainting

```sql
create table artist(
artist_id int,	
full_name varchar(50),
first_name varchar(50),
middle_names varchar(20),
last_name varchar(50),
nationality	varchar(20),
style varchar(20),
birth int,
death int);
```
```sql
select * from artist;
```
----CANVAS SIZE

```sql
create table canvas_size
(
size_id INT,
width INT,
height INT,
label VARCHAR(50)
);
```
```sql
select * from canvas_size;

create table IMAGE_Link
(
work_id int,
url varchar(500),
thumbnail_small_url varchar(500),
thumbnail_large_url varchar(550)
);


create table museum
(
museum_id int,
name varchar(150),
address	varchar(150),
city varchar(150),
	state varchar(50),
	postal	varchar(150),
	country	varchar(150),
	phone varchar(200),
	url varchar(1500)
);

drop table museum;


create table museum_hours
(museum_id int	,
day varchar(50),
open varchar(50),	
close varchar(50)
)
;


create table product_size
(work_id varchar(50),
size_id varchar(50),
sale_price varchar(50),
regular_price varchar(50)
);

drop table product_size;

create table subject
(work_id	int,
subject varchar(50)
);


create table work
(work_id	int,
name varchar(150),
artist_id int,
style varchar(150),
museum_id int
);


select *
from work;
```
-- all files have been loaded
--Answering Business Questions

--Fetch all the paintings which are not displayed on any museums?
```sql
select *
from work
where museum_id is null;
```
--Identify the museums which are open on both Sunday and Monday. Display museum name, city.
```sql
select *
from museum;

select *
from museum_hours;

select distinct(m.name), m.city
from museum as m
inner join museum_hours as mh
using(museum_id)
where mh.day = 'Sunday'
and museum_id in 
(select museum_id
from museum_hours
where day = 'Monday');

```
--Which museum is open for the longest during a day. Dispay museum name, state and hours open and which day?
```sql
select *
from museum_hours;
```
--convert into timestamp-- writing a subquery and then using rank() over() order_by function
```sql
select *
from(
SELECT m.name as museum_name, m.state, mh.day,
to_timestamp(open, 'HH:MI AM') as open_time,
to_timestamp(close, 'HH:MI PM') as close_time,
to_timestamp(close, 'HH:MI AM') - to_timestamp(open, 'HH:MI PM') as duration,
rank() over(order by (to_timestamp(close, 'HH:MI AM') - to_timestamp(open, 'HH:MI PM')) desc) 
from museum_hours mh
join museum m 
using(museum_id)) x
where x.rank = 1;
```
--Display the country and the city with most no of museums 
--Output 2 seperate columns to mention the city and country. If there are multiple value, seperate them with comma.

```sql
with cte_country as 
			(select country, count(1)
			, rank() over(order by count(1) desc) as rnk
			from museum
			group by country),
		cte_city as
			(select city, count(1)
			, rank() over(order by count(1) desc) as rnk
			from museum
			group by city)
	select string_agg(distinct country.country,', '), string_agg(city.city,', ')
	from cte_country country
	cross join cte_city city
	where country.rnk = 1
	and city.rnk = 1;
cte_city.rank=1;
```
-- Are there museuems without any paintings?
```sql
select *
from work
where museum_id is null;
```
--How many paintings have an asking price of more than their regular price? 
```sql
select *
from product_size
where sale_price>regular_price;
```
--Identify the paintings whose asking price is less than 50% of its regular price


--Which canva size costs the most?
```sql
select ps.work_id, ps.regular_price, ps.size_id
from canvas_size as cs
inner join product_size as ps
using(size_id)
order by 1 desc;

select *
from canvas_size;

select * 
from product_size;

SELECT * FROM product_size WHERE size_id !~ '^\d+$';

UPDATE product_size
SET size_id = NULL
WHERE size_id !~ '^\d+$';

ALTER TABLE product_size
ALTER COLUMN size_id TYPE integer USING size_id::integer;
```
--Delete duplicate records from work, product_size, subject and image_link tables
```sql
delete from work 
	where ctid not in (select min(ctid)
						from work
						group by work_id );

delete 
from product_size
where ctid not in(select min(ctid)
from product_size
group by work_id, size_id);

delete
from subject
where ctid not in(select min(ctid)
from subject
group by work_id, subject);

delete
from image_link
where ctid not in(select min(ctid)
from image_link
group by work_id);
```



--Identify the museums with invalid city information in the given dataset
```sql
select *
from museum
where city ~'[0-9]';
```
-- Museum_Hours table has 1 invalid entry. Identify it and remove it.
```sql
delete from museum_hours
where ctid not in (select min(ctid)
from museum_hours
group by museum_id);
```
--Fetch the top 10 most famous painting subject

```sql
select * from
(select s.subject, count(1) as no_of_paintings,
rank() over(order by count(1) desc) as rank
from work as w
inner join subject as s
using(work_id)
group by s.subject) as x
where rank >=10;
```
-- How many museums are open every single day?
```sql
select count(*)
from (select museum_id, count(*)
from museum_hours
group  by 1
having count(*) > 7) x;
```
--Which are the top 5 most popular museums? (Popularity is defined based on most no of paintings in a museum)
```sql
select m.name as museum, m.city,m.country,x.no_of_painintgs
	from (	select m.museum_id, count(1) as no_of_painintgs
			, rank() over(order by count(1) desc) as rnk
			from work w
			join museum m on m.museum_id=w.museum_id
			group by m.museum_id) x
	join museum m on m.museum_id=x.museum_id
	where x.rnk<=5;
```


-- Who are the top 5 most popular artist? (Popularity is defined based on most no of paintings done by an artist)
```sql
select 
a.full_name, a.artist_id
from
(select a.artist_id,  count(*) as no_of_paintings,
rank() over(order by count(*) desc) as rank
from work as w
inner join artist as a 
on a.artist_id = w.artist_id
group by 1) as x
inner join artist as a 
on a.artist_id = x.artist_id
where x.rank<= 5;
```
