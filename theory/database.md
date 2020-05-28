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

### Index
Clustered index: the rows are stored physically the same order as the index. So that there can be only one clustered index.  
Non-clustered index: there's one more mapping from the index to the physical row. So that when you find the right index, you need to go to the table by the corresponding mapping(index -> table row).  

What is clustered index? Leaf node is data node. This is why it's called "clustered."  
"Clustered" means that index and data are "clustered" and not able to split.  
And about non-clustered index. Leaf node is index + rowid node. It requires find data by rowid. It requires another round of **cluster index searching** to find the right data.

Index can only be prefix matching or all matching: like 'name%';  
It's because index's essence is sorting.  
What about concatenated index(multi-column index)?  
Concatenated index can only be used when there are leading columns in the where condition.  
Because the concatednated index is firstly index with first column, and secondly indexed with second column.  
Another thing need to be noted is that in the where condition, the column order doesn't matter.

  
