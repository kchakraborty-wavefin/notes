Goal of profiles project is to be the one true home of all “real world” info about people and businesses, aka the new identity

- A basis for product onboarding, compliance, and risk processes
- Want to find a way to ensure that profiles Persons and Businesses are available to use when identity Users and Businesses are created

From the example, we see bob’s user and business getting created
- Currently, is a profile for bob being created?
- What does the Person data model look like?

Considerations of current solutions:
- some need 3rd party to coordinate
	- what’s an example?
	- some use async communication, some use sync, and some hybrid
	- some are designed to never fail, some are designed to be recoverable/compensating

### Solutions
[[UUIDs|Aside on UUIDs]]

#### Solution 1: Update or Create in profiles
identity creates profile ids ahead of time using [[UUIDs|UUID v5]]→ so we supply the UUIDv1 to identity to autogenerate the profile ids

**[Steps (partial signup flow User/Person)](/)**

- Identity creates user → assigns `person_id` namespaced to identity (when does this happen?)
some very short time later? since when user first signs up, we create a User profile along with the business, and then following happens..?
- arbitrary client issues GET for Person info from **profiles** based on this `person_id`
- if profiles finds this, it returns, else empty entity returned without persistence
- client issues an UPDATE to profiles, which does a CREATE under covers if Person doesn’t exist

for failure cases, at worst data is not persisted, and client must rety (idempotent if persisted)

concurrent update requests can be protected against with [[Optimistic Locking]], client to handle error gracefully

Migration process:

- enable feature for net-new users
- backfill missing `person_ids` for Users in identity
- services migrate then they are ready, deal with conflict resolution b/w products during migration?

**[Steps (partial business Create/Update pre-migration for net-new users)](/)**

Requires two endpoints in profiles to fetch business profile OR detect client via UUIDv5 and return not found if `identity` is client

Algorithms required to allow out of order migrations - first app to migrate a given business would be diff (read from `identity` and others from `profiles`) due to pre-migration state? 

- identity creates a Business object and assigns `profiles_business_id`
> some time later... using businesses > create business or using the business switcher?
- client GET requests identity for business profile info based on `profiles_business_id`
- identity attempts to fetch data directly from profiles → this would raise not found exception → falls back on identity Business profile data to return to front end for temp 
- client issues UPDATE call to identity to update Business which proxies the request to profiles by transforming the identity` business_id` into a `profiles_business_id`
> how does this proxy work, wouldn't this be akin to relying on a DTM?
- profiles attempts to update or does create under the covers if Business doesn’t exist

for failure cases, at worst, 

![[partial_business_create.png]]

**[Steps (partial business create post-migration)](/)**

Same as user/person

Scenarios
- Director - someone who can invite a user account? is this the OG account holder/BO, the first account?
- Net new business + user created in new world, aka arbitrary users invited by director?

Migration Process:
- services operate as they do
- `profiles` endpoints implemented but sys no data
- `identity` generates `profiles_business_id` atrribs on `Business` create (uuid5, namespaced to identity)
- `identity` business endpoints are refactored
	- first to query profiles for `Business` info
	- falling back to `identity` if not found
- patterns updated to before algo
- backfill data
	- generate `profiles_business_id` for `Businesses` in `identity`
	- for each commit transaction for above change, stream `Business` profile info to `profiles` (how?)
- services migrate to after algo when ready, conflict resolution -> mark local profile?

**[Questions regarding this solution](/)**

>1.) When programmatic copying is mentioned for step 4 of the migration process, could that entail the following hypothetical scenario: something like an initial check in profiles which will have its own responsibility to generate the profile_business_id, if not found, would then query the identity DB using a [stored procedure](https://stackoverflow.com/questions/44417200/calling-a-stored-procedure-from-another-sql-server-database-engine), store the retrieved data within profiles DB, and pass the data to the client. Is the additional work on the DB worse than having the latency of identity route the GET call to profiles which then hits the profiles DB, returns a not found and profiles responds with an exception/empty entity which triggers identity to proxy the request with the profile id.

> 2.) In scenario 1 and 3, we have empty entities being returned by something like a `get_or_create` call? I know it's mentioned that these aren't persisted, but [this type of call](https://github.com/django/django/blob/main/django/db/models/query.py#L777) would net an `(object, created)` tuple, or is this some other form of logic involving the `except Model.DoesNotExist` case? Also, for scenario 2, why are there two endpoints for fetching business profile, and also lack of empty entities, but explicit need for not found exceptions?

> 3.) How does a concurrent update scenario look on the client or another service updating? What does our commit/rollback schema look like or more so our [[optimistic locking]] handling strategy? switch to sycchronous? if a transaction were to start over how does it look on the UI end? Since everything is synchronous there is no point of faillure?

> 4.) What is being referred to as the `client` in terms of handling concurrent update requests gracefully based on [[optimistic locking]]? Is it the initial client making the GET request, the client side UI, or the db client used to interface and query the db?

> 5.) With the current preferred solution, it would seem that the migration process would be the most work wrt backfilling (what do we use? airflow?) and what would be a conflict resolution scenario between two example services?


#### Saga
- `identity` creates transaction, creates and commits User/Business entity with pre-gen `profiles` id, writing to kafka
- `profiles` consumer consumes messag and commits corresponding `Person` or `Business`

- kinda like facebook? where you like gets sent to a kafka producer, and takes time for change in UI -> let owner know of imminent change in sometime a la github style
- Some services prevent frequently updating for a set number of days so users can think properly before setting (reduces actions on our end)
- What's the business value for keeping this instant. Chances of multiple updates are not as high, and we can use client side interval polling to not poll as often, given realistically we would know avg latency via stress test maybe?
- are there any use-cases where lazy update based on poll would be detrimental for the BO? e.g. trying to do something time-sensitive on the app with the updated business profile changes?

#### Hybrid Async Two Phase (by identity)
- `identity` creates User/Business entity in prepared state?
- leverages rpc call directly to profiles to prepare Person/Business in there
- `identity` creates transaction, commits init entry as it also streams created message in Kafka
- `profiles` consumer consumes message and commits to profiles changes
- crons for cleanup of prepared entities older than N mins

#### Two system Two phase commit (by identity)
- `identity` creates User/Business in prepared state
- `identity` issues RPC call to profiles to create Person/Business for it
- `identity` commits init entry
- `identity` cron removes prepared entities older than N minutes

### References
- [https://github.com/waveaccounting/profiles/blob/distributed-transaction-decision/decisions/001-identity-user-business-sync.md](https://github.com/waveaccounting/profiles/blob/distributed-transaction-decision/decisions/001-identity-user-business-sync.md "https://github.com/waveaccounting/profiles/blob/distributed-transaction-decision/decisions/001-identity-user-business-sync.md")
- [https://aws.amazon.com/rds/proxy/](https://aws.amazon.com/rds/proxy/ "https://aws.amazon.com/rds/proxy/") 
- https://www.startdataengineering.com/post/how-to-backfill-sql-query-using-apache-airflow/
 