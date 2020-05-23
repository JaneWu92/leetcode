### sql vs nosql
It's equal to relational database vs non-relational database.  
The biggest difference may be relational database the data should be in the format of tabular and has clear schemas. And non-relational may like key-value or others.  
And note that some non-relational database also provide some sql-like query engine.

### Why relational database is hard to scale 
Relational database requires ACID: Atomicity, consistency, isolation, durability.  
And these will be very difficult to maintain in distrubuted system.  
And for nosql, the ACID is not required, so it's easy to horizontally scale.  
**why for nosql ACID is not required?**  
I think it's not because that the previous relational scenario is changed.  
It may because that there are new scenario happening: data is becoming more and more huge. And it becomes a new case we need to take care.
