## Postgresql sample codes as example

### Prepared statements
´´´sql
/* create it */
PREPARE getActorName(INTEGER) AS
SELECT * FROM actor WHERE actor_id = $1;
/* execute it */
EXECUTE getActorName(20);
-- delete it
DEALLOCATE getActorName;

```

### Functions and procedures



#### IN, OUT and INOUT

```sql

CREATE OR REPLACE FUNCTION getStat(
	OUT min_length INTEGER,
	OUT avg_length INTEGER,
	out max_length INTEGER
) RETURNS RECORD AS $$ -- RETURNS is optional here
BEGIN
 SELECT min(length), avg(length), max(length) INTO min_length, avg_length, max_length
 FROM film;
END; $$
LANGUAGE plpgsql;

SELECT getStat();

```