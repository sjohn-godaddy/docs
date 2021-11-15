---

## completed work

#### dev-private
- dms initial load completed, and running 24/7 to collect incremental changes
- etl jobs automatically scheduled to run once a day
- queries written to produce reamaze reports

#### production
- dms configured and tested on 5%, 10%, 20% of initial load
- etl jobs configured and tested on 5%, 10%, 20% of initial load

---

## remaining work

#### dms work
  - wait for aurora restart to enable cdc support
  - re-configure dms to use cdc
  - wait for cp_solutions to fix 4096 limit
  - re-configure dms table filter
  - run entire database through dms

#### etl work
  - wait for etl team to fix hstore
  - re-configure etl to use hstore
  - switch all etl jobs to 1.22.2 to enable regex_replace
  - test etl with 2 parallel workers and confirm cpu/ram estimates
  - run and finish etl work

#### data lake work
  - add the data lake role/policy
  - merge data lake pull request
  - test running queries on the lake

---
