# Neo4j in cypher

### create 
Creating a node
```sql
-- Creating a single Node
CREATE (p:Person {name: 'Alice', age: 30, city: 'New York'}) 

-- creating Multiple nodes in one go

CREATE (m:Movie {title: 'The Matrix', released: 1999, tagline: 'Welcome to the Real World'}),
       (p1:Person {name: 'Keanu Reeves'}),
       (p2:Person {name: 'Carrie-Anne Moss'})

-- create a Node with more than one label, eg. A person and Actor, like inheritance
CREATE (a:Person:Actor {name: 'Tom Hanks'})

-- Mergng a Node (creats if one doesnt exist, else matches it)
MERGE (g:Genre {name: 'Science Fiction'})
```
Creating Relationships
```sql
-- get Nodes using match with parameter, and create relationshp
MATCH (a:Person {name: 'Keanu Reeves'}), (m:Movie {title: 'The Matrix'})
CREATE (a)-[:ACTED_IN {roles: ['Neo']}]->(m)

-- Creating Nodes and Relationships in one go
-- this creates two movies and one person, and relates the person as director to both
CREATE (director:Person {name: 'Lana Wachowski'})-[:DIRECTED]->(matrix:Movie {title: 'The Matrix'}),
       (director)-[:DIRECTED]->(resurrections:Movie {title: 'The Matrix Resurrections'})

-- using merge to create relationships
-- if charlie already works with ABC corp (Relation exists), it is maped, 
-- if ABC corp exists but charlie doesnt, the relationship is created
-- if ABC corp doesnt exist, it and the relationship are created
MATCH (charlie:Person {name: 'Charlie'})
MERGE (charlie)-[:WORKS_FOR]->(company:Company {name: 'ABC Corp'})
```

Projecting. The most common
```sql
MATCH (n) RETURN n -- returns every thing

-- retrieve by label
-- gets all persons returns name and age
MATCH (p:Person) RETURN p.name, p.age 

-- retrieve by property (filtering by property)
-- returns from movies where title = the matrix
MATCH (m:Movie {title: 'The Matrix'})
RETURN m.released, m.tagline

-- retrieve by Node label and property
-- returns all Persons whos city propert = New York
MATCH (p:Person {city: 'New York'})
RETURN p.name
```

Retrieving Nodes and relationships
```sql
-- get all persons who acted in a movie
-- return name relationship movie title and role
-- we can also access this without direction
MATCH (p:Person)-[r:ACTED_IN]->(m:Movie)
RETURN p.name, type(r), m.title, r.roles

-- retrieve relationship of any type
MATCH (p:Person {name: 'Tom Hanks'})-[r]->(m:Movie)
RETURN type(r), m.title

-- multi hop. finding Friends of friends
MATCH (p1:Person {name: 'Alice'})-[:FRIENDS_WITH]->(p2:Person)-[:FRIENDS_WITH]->(p3:Person)
RETURN p3.name

-- finding paths of variable length
MATCH (p1:Person {name: 'Alice'})-[*1..3]->(p2:Person)
RETURN p1.name, p2.name, relationships(p1,p2) AS path_relationships

-- shortest path
MATCH (p1:Person {name: 'Alice'}), (p2:Person {name: 'Bob'})
MATCH p = shortestPath((p1)-[*]->(p2))
RETURN p
```

Filtering results using WHERE clause
```sql
-- simple
MATCH (m:Movie)
WHERE m.released > 2000
RETURN m.title, m.released

-- get relationship of a type form -> to
MATCH (from_node)-[l:Lectures]->(to_node)
RETURN from_node.name, l.property_name, to_node.title

-- --- Homework question ---
-- Create a query to get all lectures relationships. 
-- answer 
MATCH ()-[l:Lectures]->() RETURN l
-- --- question ---
-- Create a query that gets all students that have a relationship with the coursname Transfiguration. 
MATCH (student:Student)-[]->(course:Course {name: 'Transfiguration'})
RETURN DISTINCT student.name

-- --- last question ---
-- get fellow Students of Max musterman without specifying the course
MATCH (max:Student {firstname: 'Max', lastname: 'Mustermann'})
      -[max_c:LECTURES]->(max_l:Lecturer) -- get all courses max attends
MATCH (s:Student)
      -[l:LECTURES]->(fs_l:Lecturer) -- get all student courses
WHERE s <> max -- Exclude Max himself
AND l.coursename = max_c.coursename -- Compare course names dynamically

RETURN DISTINCT fellow_student.firstname, fellow_student.lastname, fellow_student.matriculationnumber


-- with properties
MATCH (p:Person)-[a:ACTED_IN]->(m:Movie)
WHERE 'Neo' IN a.roles  -- Check if 'Neo' is in the roles list
RETURN p.name, m.title

-- combining filters
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
WHERE p.age > 40 AND m.released < 2000
RETURN p.name, m.title

-- ordering results
MATCH (m:Movie)
RETURN m.title, m.released
ORDER BY m.released DESC

-- skipping and limiting results
MATCH (p:Person)
RETURN p.name
SKIP 5 LIMIT 5 -- skip the first 5 and return 5 nodes
```




