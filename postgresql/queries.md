## Postgresql sample codes as example

### Prepared statements

Prepared statements are not persisted. i.e only available for a single session. the cannot be shared among Users

```
-- create it 
PREPARE getActorName(INTEGER) AS
SELECT * FROM actor WHERE actor_id = $1;
-- execute it 
EXECUTE getActorName(20);

-- delete it 
DEALLOCATE getActorName;
```

### Functions and procedures
Functions and Procedures are stored and are not dependent on a session.
#### Procedures
Procedures dont have a return value (Void) so we dont add RETURNS, nor can we expect a table or resul.
they are basically functions without a return type

```
-- create it
CREATE OR REPLACE PROCEDURE CreateActor (
 firstname varchar (255) ,
 lastname varchar (255)
 )
 AS $$
    DECLARE highestid integer ;
    BEGIN
        SELECT max ( actor_id ) + 1 -- get the highest number, and add 1 to make a new id
            INTO highestid -- store it in to declared variable
            FROM Actor;
        INSERT
            INTO ACTOR ( actor_id , first_name , last_name , last_update )
            -- use generated id as new ID for the new actor
            VALUES ( highestid , firstname , lastname , CURRENT_TIMESTAMP ); 
    END; $$ 
LANGUAGE plpgsql;

-- execute is
CALL CreateActor('Muster', 'Musteractor');

-- drop it
DROP PROCEDURE IF EXISTS createActor(); -- just like a function
```

#### Functions
Functions have return value unlike Procedures
```
-- create it
CREATE OR REPLACE FUNCTION getFilmName(f_id INTEGER) 
RETURNS TEXT AS $$
DECLARE f_name TEXT;
BEGIN
 SELECT title INTO f_name FROM film WHERE film_id = $1;
 return f_name;
END; $$
LANGUAGE plpgsql;

-- call it
SELECT getFilmName(20);

-- drop it

DROP getFilmName(INTEGER) -- full function signature
```
#### Declare and Return Types
```
CREATE OR REPLACE FUNCTION getFilmDescription(film_title varchar)
-- if we dont know the type of content in the column description in table film is
-- we do a tablename.column%TYPE. 
-- for a ROW we do tablename%ROWTYPE -> gets us the whole row
RETURNS film.description%TYPE AS $$ 
DECLARE dsc film.description%TYPE;
BEGIN
	SELECT description INTO dsc 
	FROM film WHERE title LIKE film_title;
RETURN dsc;
END; $$
LANGUAGE plpgsql;
```

#### IN, OUT and INOUT

```sql
-- create it
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

-- execute it
SELECT getStat();

-- drop it

DROP FUNCTION IF EXISTS getStat()

```