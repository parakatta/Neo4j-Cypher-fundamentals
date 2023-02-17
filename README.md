# Neo4j Cypher fundamentals
Getting started with Neo4j Graph database fundamentals using Movie database.

Cypher is Neo4j’s graph query language that allows users to store and retrieve data from the graph database. It is a declarative, SQL-inspired language for describing visual patterns in graphs. The syntax provides a visual and logical way to match patterns of nodes and relationships in the graph.

Common used keywords in Cypher include : 
`MATCH , MERGE WHERE` etc. 

The basics of a query include nodes called **labels** and **relationship** between two nodes. \
In the Movie database, we have :Person , :Movie as labels and relationships include :ACTED_IN or :DIRECTED_IN

eg: `MATCH (m:Movie)<-[:ACTED_IN]-(p:Person)`
this will retrieve all Person aliased as 'p' who acted in all Movie aliased 'm'. Note that MATCH is used to retrieve all rows. 
Notice the arrow head direction in the above query. Here, Person acted in Movie are retrieved. This is because the graph is a directed graph. We may think to rewrite the query as 

`MATCH (p:Person)-[:ACTED_IN]->(m:Movie)`  (*notice the arrow direction*)

We use CamelCase for labels with title characters, UPPERCASE to denote relationships.

Now lets dive into creating, reading, and making clauses for our query. 

![Basic graph base](https://neo4j.com/docs/cypher-manual/current/_images/graph_match_clause.svg)

###CREATE 

To create the Person node you can either use the CREATE or MERGE keywords. The MERGE statement in the query below will only create the node if it does not already exist in the graph.

`MERGE (p:Person {name: 'Daniel Kaluuya'})`

To add a new node and relationship to the graph:
- Find the Person node for Daniel Kaluuya.
- Create the Movie node, Get Out.
- Add the ACTED_IN relationship between Daniel Kaluuya and the movie, Get Out.

```
MERGE (p:Person {name: 'Daniel Kaluuya'})
MERGE (m:Movie {title: 'Get Out'})
MERGE (p)-[:ACTED_IN]->(m)
RETURN p, m
```


Write the Cypher code to find this Movie node and add the tagline and released properties for the node with the values below.
tagline: Gripping, scary, witty and timely!
released: 2017

```
MATCH (m:Movie {title: 'Get Out'})
SET  m.tagline = 'Gripping, scary, witty and timely!',
     m.released = 2017
```

If you were to attempt DELETE p, it would fail because it has relationships and Cypher prevents you from deleting nodes without first deleting its relationships.
There is no REMOVE p clause in Cypher. We use DETACH DELETE to first detach the edge then delete it. 

```
MATCH (p:Person {name: "Emil Eifrem"})
DETACH DELETE p
```
The following query will find any :Person node with the name Emil Eifrem and then use the DETACH DELETE clause to remove each node and any relationships into our out from the node.


We can now add labels to the Person labels and subcategorize it to Actors and Directors. 
```
MATCH (p:Person)
WHERE exists ((p)-[:ACTED_IN]-())
SET p:Actor
```

This will set the query results labelled as Actor . So now we can query this label than retrieving entire Person results using PROFILE . 

```
PROFILE MATCH (p:Actor)-[:ACTED_IN]-()
WHERE p.born < '1950'
RETURN p.name
```

Lets create the database and add labels and relationships with properties
```
MERGE (:Movie {title: 'Apollo 13', tmdbId: 568, released: '1995-06-30', imdbRating: 7.6, genres: ['Drama', 'Adventure', 'IMAX']})
MERGE (:Person {name: 'Tom Hanks', tmdbId: 31, born: '1956-07-09'})
MERGE (:Person {name: 'Meg Ryan', tmdbId: 5344, born: '1961-11-19'})
MERGE (:Person {name: 'Danny DeVito', tmdbId: 518, born: '1944-11-17'})
MERGE (:Movie {title: 'Sleepless in Seattle', tmdbId: 858, released: '1993-06-25', imdbRating: 6.8, genres: ['Comedy', 'Drama', 'Romance']})
MERGE (:Movie {title: 'Hoffa', tmdbId: 10410, released: '1992-12-25', imdbRating: 6.6, genres: ['Crime', 'Drama']})
MERGE (:Person {name: 'Jack Nicholson', tmdbId: 514, born: '1937-04-22'})
MERGE (:User {name: 'Sandy Jones', userId: 534})
MERGE (:User {name: 'Clinton Spencer', userId: 105})
```

```
MATCH (apollo:Movie {title: 'Apollo 13'})
MATCH (tom:Person {name: 'Tom Hanks'})
MATCH (meg:Person {name: 'Meg Ryan'})
MATCH (danny:Person {name: 'Danny DeVito'})
MATCH (sleep:Movie {title: 'Sleepless in Seattle'})
MATCH (hoffa:Movie {title: 'Hoffa'})
MATCH (jack:Person {name: 'Jack Nicholson'})

// create the relationships between nodes
MERGE (tom)-[:ACTED_IN {role: 'Jim Lovell'}]->(apollo)
MERGE (tom)-[:ACTED_IN {role: 'Sam Baldwin'}]->(sleep)
MERGE (meg)-[:ACTED_IN {role: 'Annie Reed'}]->(sleep)
MERGE (danny)-[:ACTED_IN {role: 'Bobby Ciaro'}]->(hoffa)
MERGE (danny)-[:DIRECTED]->(hoffa)
MERGE (jack)-[:ACTED_IN {role: 'Jimmy Hoffa'}]->(hoffa)
```
What is the highest rated movies in year 1995 according to the imdbrating?
```
MATCH (m:Movie)
WHERE m.released STARTS WITH '1995'
RETURN m.title, m.imbdrating ORDER BY
m.imdbrating DESC LIMIT 1
```

Use a WITH clause to limit and order nodes.

eg : What is the highest revenue movie for Tom Hanks?

```
WITH  'Tom Hanks' AS theActor
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
WHERE p.name = theActor
AND m.revenue IS NOT NULL
WITH m ORDER BY m.revenue DESC LIMIT 1
RETURN m.revenue AS revenue, m.title AS title
```
What is the highest revenue for a movie for Tom Hanks?
```
WITH  'Tom Hanks' AS theActor
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
WHERE p.name = theActor
AND m.revenue IS NOT NULL
WITH m ORDER BY m.revenue DESC LIMIT 1
RETURN m.revenue AS revenue, m.title AS title
```
What actor acted in more than one of these top four movies
```
MATCH (n:Movie)
WHERE n.imdbRating IS NOT NULL AND n.poster IS NOT NULL
WITH n {
  .title,
  .imdbRating,
  actors: [ (n)<-[:ACTED_IN]-(p) | p { tmdbId:p.imdbId, .name } ],
  genres: [ (n)-[:IN_GENRE]->(g) | g {.name}]
}
ORDER BY n.imdbRating DESC
LIMIT 4
RETURN collect(n)
```
