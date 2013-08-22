Neo4Forum
=========

###Selezione delle ultime dieci domande postate con punteggio positivo e numero di visite superiore a 50;
Query:

    MATCH (q:Question)
    WHERE q.score > 0 AND q.views > 50
    RETURN q.title, q.data, q.score, q.views
    ORDER BY q.data DESC
    LIMIT 10;
