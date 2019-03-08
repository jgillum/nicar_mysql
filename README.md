
# NICAR 2019 intermediate MySQL

By Jack Gillum, ProPublica \
jack.gillum@propublica.org \
Data from: https://data.baltimorecity.gov/Public-Safety/911-Police-Calls-for-Service/xviu-ezkt

Go ahead and copy/paste everything in the codeblock below into your MySQL window:

```
## Let's explore the tables using select. We should make a note of the grade types
SELECT * FROM schools AS s;  # All schools in Baltimore
SELECT * FROM calls AS c LIMIT 1000; # BPD calls for service from 2016

## Look at distinct calls 
SELECT DISTINCT description
FROM calls AS c;

## Look at distinct call priorities 
SELECT DISTINCT priority
FROM calls AS c;

## Narrow it to armed people
SELECT DISTINCT description 
FROM calls AS c 
WHERE description LIKE '%armed%';

## Count armed person calls, high priority
SELECT COUNT(*)
FROM calls AS c 
WHERE description LIKE '?????'
AND (priority LIKE '???' OR priority LIKE '????');

## Now, let's join them with schools using ST_DISTANCE_SPHERE()
## More at: https://tighten.co/blog/a-mysql-distance-function-you-should-know-about 

SELECT *
	FROM schools AS s
	JOIN calls AS c
	ON ST_DISTANCE_SPHERE(c.coord,s.coord) <= 200 ## In meters
	WHERE c.description LIKE 'ARMED%'
	AND (c.priority LIKE 'High' OR c.priority LIKE 'Emergency'); 

## Let's winnow down our columns. Which ones do we want? (Hint: We might want to add the distance, too.)

SELECT ??????????
	FROM schools AS s
	JOIN calls AS c
	ON ST_DISTANCE_SPHERE(c.coord,s.coord) <= 200 ## In meters
	WHERE c.description LIKE 'ARMED%'
	AND (c.priority LIKE 'High' OR c.priority LIKE 'Emergency'); 
	
## How about we find schools that have the most armed person calls near them?
	
SELECT schoolname, COUNT(*) AS count
FROM (
	SELECT s.*
	FROM schools AS s
	JOIN calls AS c
	ON ST_DISTANCE_SPHERE(c.coord,s.coord) <= 200
	WHERE c.priority LIKE 'High'
	AND c.description LIKE 'ARMED%'
	) AS x
GROUP BY schoolname	
ORDER BY count DESC

## What about just elementary schools?

SELECT schoolname, COUNT(*) AS count
FROM (
	SELECT s.*
	FROM schools AS s
	JOIN calls AS c
	ON ST_DISTANCE_SPHERE(c.coord,s.coord) <= 200
	WHERE c.priority LIKE 'High'
	AND c.description LIKE 'ARMED%'
	AND ?????????
	) AS x
GROUP BY schoolname	
ORDER BY count DESC;

## Add some more data. Might even want to narrow the time frame to school hours.

SELECT c.call_number, c.date_time, c.incident_location, c.priority, c.xlon, c.ylat
FROM schools AS s
JOIN calls AS c
ON ST_DISTANCE_SPHERE(c.coord,s.coord) <= 200
WHERE c.priority LIKE 'High'
AND c.description LIKE 'ARMED%'
AND s.schoolname like 'Gilmor Elementary'
	AND HOUR(c.date_time) >= 8
	AND HOUR(c.date_time) <= 15
	AND WEEKDAY(c.date_time) NOT IN (5,6);

## Let's plot some quick lat/longs: http://www.copypastemap.com/

SELECT DISTINCT c.xlon, c.ylat
FROM schools AS s
JOIN calls AS c
ON ST_DISTANCE_SPHERE(c.coord,s.coord) <= 200
WHERE c.priority LIKE 'High'
AND c.description LIKE 'ARMED%'
AND s.schoolname like 'Gilmor Elementary'
	AND HOUR(c.date_time) >= 8
	AND HOUR(c.date_time) <= 15
	AND WEEKDAY(c.date_time) NOT IN (5,6);
	
```
