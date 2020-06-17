### CAP theorem
consistency, availability, partition tolerance  
1. consistency: any time any access should read the most recent write.  
2. availability: there's no downtime on the service  
3. partition tolerance: cluster continue to function if there's a communication break(partition) between 2 nodes. note that in fact nodes are up, it's just they are not able to communicate.  
consistency + partition tolerance: when the partition happens, bring down the service to keep consistency, and then the availability breaks.
consistency + availability: partition(communication breaks) is not tolerant.  
partition tolerance + availability: there must be something outdated in some nodes. so consistency is not able to provide.  
  
In fact it should be state in this way:  
For distribution system, the most important part with the "non-distribution system" is the network partition.  
so the CAP is: when the network partition failure fails, what we want to decide to:
1. proceed with the operation so that availability is provided but there's there may be inconsistency
2. cancel the operation and so the availability is decreased but the consistency is ensured.  

**it's about read and write**

### Why we come to Nosql
Nosql vs relational database  
Relational data has acid constraint, it means that the write and read will be slow.  
And Nosql doesn't have this constraint, one aspect is that it's deployed in distributed system which support its huge data volume.  
I think the biggest item here is that the data volume become bigger and bigger, which the relational database is not able to maintain in one machine.  
**The key point is scale out**
The good side is the load is distributed.  
The side effect is the non-acid protect.

### read-write split
master-slave mode. master for write and slave for read.  
pros:  
read can be load balance
cons:  
write can not be load balance  
important read may have outdated data in slave  
  
master-master mode  
write and read can all be load balance
while inconsistency may be more serious than master-slave mode  
there's no single-point faiure.  

### table split vs database split
table split:  
business split  
horizontal split:  range route, hash route(not easy to scale out when there's new table), config route
vertical split： split out some less-use column   
  
database split:  
transaction can only be provided by ourselves  



