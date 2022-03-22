Want to move $100 from A to B

A - 100, B + 100, therefore guarantee must be provided for the entire transfer and it should atomically succeed or fail as a whole

If system is monolithic, transfer can be accomplished in one service, one DB, and in one go using an ACID transaction
    
Things change in a distributed transaction env where A now needs to transfer to B _across banks_...

- transfer cannot be guaranteed by local transaction in one DB
- these follow [[ACID-BASE review|BASE]] principles to meet availability, performance with a tradeoff for consistency and isolation
    
## Some popular solutions to this problem:

#### Two Phase commit/XA:
- XA is a spec for distributed transactions proposed by X/Open org
- essentially, there’s an interface between a global Transaction Manager (DTM) and a local resource manager (MySQL, PgSQL etc.)
- Phases:
	- prepare: all RMs prepare to execute their transactions and lock the required resources. When ready, they report to TM stating ready state
	- commit/rollback: TM confirms all RMs ready, sends commit command to all participants 

#### Saga
- long lived transaction as a multiple short local transactions, done by SAGA coordinator (DTM)
- Phases:
	- commit saga transaction including transOUTs/INs provided by all participating RMs
	- the DTM coordinates the transOUT and then transINs
	- operation succeeds

#### Try, Confirm, Cancel
- Phases:
	- Try - requestor requests RM to perform tentative operation
	- Confirm: if provider completes Try, request executes a confirmation operation if provider to move forward. No more validations will be performed, 


### References
https://medium.com/@dongfuye/the-seven-most-classic-solutions-for-distributed-transaction-management-3f915f331e15