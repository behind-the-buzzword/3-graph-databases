# BTBW3 - Graph Databases

## Instructions
The docker-compose file contained within this project will launch a Neo4j container running the latest available image.
To start it run: `docker-compose up -d; docker-compose logs -f`

Within the import folder are several CSV files to import. These represent different pieces of our model and the relationships between them.
To import these files run the following commands in order:

### Clear all data
```sql
MATCH (n)
   WITH n LIMIT 10000
   OPTIONAL MATCH (n)-[r]->()
   DELETE n,r;
```

### Load in person data & relationships
```sql
LOAD CSV WITH HEADERS FROM "file:///person_node.csv" AS r FIELDTERMINATOR ';'
WITH r LIMIT 10 WHERE r.`id:ID(Person)` IS NOT NULL
RETURN r.`id:ID(Person)`, r.name, r.`born:int`, r.poster_image
``` 

```sql
LOAD CSV WITH HEADERS FROM "file:///acted_in_rels.csv" AS r FIELDTERMINATOR ';'
WITH r LIMIT 10
RETURN r.`:START_ID(Person)`, r.`:END_ID(Movie)`, SPLIT(r.role, '/')
```   

```sql
CREATE CONSTRAINT ON (g:Genre) ASSERT g.id IS UNIQUE;
```   

```sql
LOAD CSV WITH HEADERS FROM "file:///genre_node.csv" AS r FIELDTERMINATOR ';'
CREATE (g:Genre {
 id: toInteger(r.`id:ID(Genre)`),
 name: r.name
});
```

### Keywords
```sql
CREATE CONSTRAINT ON (k:Keyword) ASSERT k.id IS UNIQUE;
```

```sql
LOAD CSV WITH HEADERS FROM "file:///keyword_node.csv" AS r FIELDTERMINATOR ';'
CREATE (k:Keyword {
  id: toInteger(r.`id:ID(Keyword)`),
  name: r.name
});
```

### Movies

```sql
CREATE CONSTRAINT ON (m:Movie) ASSERT m.id IS UNIQUE;
```

```sql
LOAD CSV WITH HEADERS FROM "file:///movie_node.csv" AS r FIELDTERMINATOR ';'
CREATE (m:Movie {
  id: toInteger(r.`id:ID(Movie)`),
  title: r.title,
  tagline: r.tagline,
  summary: r.summary,
  poster_image: r.poster_image,
  duration: toInteger(r.`duration:int`),
  rated: r.rated
});
```

```sql
CREATE CONSTRAINT ON (p:Person) ASSERT p.id IS UNIQUE;
```

```sql
LOAD CSV WITH HEADERS FROM "file:///person_node.csv" AS r FIELDTERMINATOR ';'
CREATE (p:Person {
  id: toInteger(r.`id:ID(Person)`),
  name: r.name,
  born: toInteger(r.`born:int`),
  poster_image: r.poster_image
});
```

### Relationships between Movie and Genre
```sql
LOAD CSV WITH HEADERS FROM "file:///has_genre_rels.csv" AS r FIELDTERMINATOR ';'
MATCH (m:Movie {id: toInteger(r.`:START_ID(Movie)`)}), (g:Genre {id: toInteger(r.`:END_ID(Genre)`)})
CREATE (m)-[:HAS_GENRE]->(g);
```

### Relationships between Movie and Keywords
```sql
LOAD CSV WITH HEADERS FROM "file:///has_keyword_rels.csv" AS r FIELDTERMINATOR ';'
MATCH (m:Movie {id: toInteger(r.`:START_ID(Movie)`)}), (k:Keyword {id: toInteger(r.`:END_ID(Keyword)`)})
CREATE (m)-[:HAS_KEYWORD]->(k);
```

### Relationships between Person and Movie (Writer of)
```sql
LOAD CSV WITH HEADERS FROM "file:///writer_of_rels.csv" AS r FIELDTERMINATOR ';'
MATCH (p:Person {id: toInteger(r.`:START_ID(Person)`)}), (m:Movie {id: toInteger(r.`:END_ID(Movie)`)})
CREATE (p)-[:WRITER_OF]->(m);
```

### Relationships between Person and Movie (Producer of)
```sql
LOAD CSV WITH HEADERS FROM "file:///produced_rels.csv" AS r FIELDTERMINATOR ';'
MATCH (p:Person {id: toInteger(r.`:START_ID(Person)`)}), (m:Movie {id: toInteger(r.`:END_ID(Movie)`)})
CREATE (p)-[:PRODUCED]->(m);
```

### Relationships between Person and Movie (Director of)
```sql
LOAD CSV WITH HEADERS FROM "file:///directed_rels.csv" AS r FIELDTERMINATOR ';'
MATCH (p:Person {id: toInteger(r.`:START_ID(Person)`)}), (m:Movie {id: toInteger(r.`:END_ID(Movie)`)})
CREATE (p)-[:DIRECTED]->(m);
```

### Relationships between Person and Movie (Acted In)
```sql
LOAD CSV WITH HEADERS FROM "file:///acted_in_rels.csv" AS r FIELDTERMINATOR ';'
MATCH (p:Person {id: toInteger(r.`:START_ID(Person)`)}), (m:Movie {id: toInteger(r.`:END_ID(Movie)`)})
CREATE (p)-[:ACTED_IN{role:SPLIT(r.role, '/')}]->(m);
```