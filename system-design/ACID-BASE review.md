### ACID

**A**tomic - Each operation within a transaction succeeds or the process halts and the database rolls back to the state before the transaction started. This ensures that all data in the database is valid.

**C**onsistent - processed transaction never endangers structural integrity

**I**solated - transactions cannot compromise integrity of other transactions in progress, moderated by the DB

**D**urable - results of completed transaction persists permanently even after network outages

-   ensures that a performed transaction is always consistent    
-   good fit for transaction processing (financial institutions, data warehousing)    
    -   e.g. money debited from acc but not credited to corresponding acc        
-   handle many small, simultaneous transactions    
-   zero tolerance for invalid states    
-   MySQL, PgSQL, Oracle SQL etc.
    

### BASE

For many domains and use cases, ACID transactions are far more pessimistic (more worried about data safety than actually required). Some databases have loosened the requirements for immediate consistency, data freshness and accuracy in order to gain other benefits, like scale and resilience.

**B**asically Available - more focus on availability through spreading and replicating across nodes of DB cluster, rather than immediate consistency of data

**S**oft state - data values change over time, lack of immediate write consistency, this responsibility is delegated to developers

**E**ventually Consistent - data reads are still possible, but they are not the real source of truth until eventual consistency is achieved lazily at read time