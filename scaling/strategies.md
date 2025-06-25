# Scaling Strategies

## 1. Optimize Existing Architecture

- **Database queries**: Indexing, Caching, Denormalization, Materialized Views
- **Code efficiency**: 
- **Asset delivery**: CDNs
- **Vertical scaling**: adding more CPU, RAM, or storage 
- **Asynchronous processing**: Workers + Message Queue

## 2. Horizontal Scaling 

- **Service replication**
- **Load balancing**: distributes incoming traffic across multiple servers (HAProxy, NGINX)
- **Stateless architecture**: no session data stored on servers (JWT)
- **Microservices**: independent scaling of each service based on its specific needs
- **CQRS pattern** (Command Query Responsibility Segregation): enables the read models and write models to scale independently

### Database Scaling

- **Data partitioning** (Sharding)
- **Master-Slave** replication: one database server (the master) handles all write operations, while one or more other servers (the slaves) replicate the data from the master; read operations can be distributed among the master and slaves
- **Master-Master** replication (multi-master replication)

### PACELC Theorem

PALCELC and CAP were developed to provide a framework for comparing distributed systems. PACELC is an extension to the CAP theorem to address a key limitation of the CAP theorem: it makes no provision for performance or latency. 

> [!NOTE]
> A partition is a break in communications—a temporarily delayed or lost connection between nodes. In a distributed database, there is no way to avoid system partitions. 
>
> - Partition tolerance means that in spite of any number of breakdowns in communication between nodes in the system, the cluster will continue to work.
> - Although CAP theorem stating a CA distributed database is possible exists, there is currently no true CA distributed database system

## 3. References
 
- [ScyllaDB | PACELC Theorem](https://www.scylladb.com/glossary/pacelc-theorem/)
- [Why isn't RDBMS Partition Tolerant](https://stackoverflow.com/questions/36404765/why-isnt-rdbms-partition-tolerant-in-cap-theorem-and-why-is-it-available)