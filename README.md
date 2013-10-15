Neo4Forum
=========
0) Creazione di un indice sulle domande e i vincoli di unicità su {Post, User, Tag}

    CREATE INDEX ON :Question(pid);
    CREATE CONSTRAINT ON (t:Tag) ASSERT t.name IS UNIQUE;
    CREATE CONSTRAINT ON (p:Post) ASSERT p.pid IS UNIQUE;
    CREATE CONSTRAINT ON (u:User) ASSERT u.uid IS UNIQUE;

1) Selezione delle ultime dieci domande postate con punteggio positivo e numero di visite superiore a 50;

    MATCH (q:Question)
    WHERE q.score > 0 AND q.views > 50
    RETURN q.title, q.data, q.score, q.views
    ORDER BY q.data DESC
    LIMIT 10;

2) Selezione dei titoli delle domande con tag “Cypher”

    MATCH (q:Question)-[r:TaggedWith]-(t:Tag)
    WHERE t.name =~ '(?i)cypher'
    RETURN q.title;
    Returned 582 rows.Query took 1605ms
    MATCH (t:Tag)
    WHERE t.name =~ '(?i)cypher'
    WITH t
    MATCH (q:Question)-[r:TaggedWith]-(t)
    RETURN q.title;
    Returned 582 rows.Query took 152ms
    
3) Selezione degli utenti che sfruttano “C#” e “Neo4J” (hanno postato domande con tag “C#” o risposte e commenti all’interno di domande con tag “C#”)

    MATCH (u:User)-[r*1..2]-(q:Question)-[:TaggedWith]-(t:Tag)
    WHERE t.name =~ '(?i)c#' and NONE (x IN r WHERE type(x) = 'RelatedTo')
    RETURN DISTINCT u.username;
    returned 30 rows.Query took 17918ms
    
    MATCH (t:Tag)
    WHERE t.name =~ '(?i)c#'    
    WITH t
	MATCH (u:User)-[r*1..2]-(q:Question)-[:TaggedWith]-(t)
    WHERE NONE (x IN r WHERE type(x) = 'RelatedTo')
    RETURN DISTINCT u.username;
    Returned 30 rows.Query took 124ms
    
4) Data la domanda senza risposta con più visite, selezionare le domande inerenti (al più in due hop);

    MATCH nodes = (q:Question)
    WHERE not((q)<-[:Replies]-())
	WITH q
    ORDER BY q.views DESC
    LIMIT 1
    MATCH (q)-[:RelatedTo*1..2]->(related:Question)
    RETURN distinct related.title;
    Returned 28 rows.Query took 41ms

5) Calcolo della reputazione degli utenti (somma dei punteggi guadagnati / numero di interventi fatti);

    MATCH (p:Post)-[:WrittenBy]-(u:User)
    RETURN u.username, SUM(p.score)/(count(p)*1.0) as Media, SUM(p.score) as Punteggio, count(p) as Post
    ORDER BY Media DESC;
    
6) Numero di post pubblicati da Maggio 2013 (indicativamente data dell’ultima release di Neo4J) in funzione dei singoli tag;

    Epoch timestamp: 1367366400
    Timestamp in milliseconds: 1367366400000
    Human time (GMT): Wed, 01 May 2013 00:00:00 GMT
    Human time (your time zone): 1/5/2013 02:00:00

    MATCH (p:Post)-[:Replies|Improves]->(q:Question)-[:TaggedWith]->(t:Tag)
    WHERE q.data >= '1367366400'
    RETURN t.name, count(q)+count(p) AS Totale
    ORDER BY Totale DESC;
    Returned 240 rows.Query took 257ms
    
    MATCH (q:Question)
    WHERE q.data >= '1367366400'
    WITH q
    MATCH (p:Post)-[:Replies|Improves]->(q)-[:TaggedWith]->(t:Tag)
    RETURN t.name, count(q)+count(p) AS Totale
    ORDER BY Totale DESC;
    Returned 240 rows.Query took 223ms
