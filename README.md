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
