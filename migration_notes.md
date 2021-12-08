#  Migration Discussion

## Let's migrate on the fly
Migrate on demand. Its simpler and we dont have to worry about deltas or syncing. Migrate the accounts, brands, channels via existing api. the volume is small, and you get immediate feedback about success/failure
1. Call the accounts create api with `status=migrating`
2. Call the brands/channels create apis
3. Migrate the most recent threads/messages
4. Call accounts api to update `status=active-recent-only`
5. Migrate the remaining recent threads/messages
6. Call accounts api to update `status=active`

## Migrating thread/messages on the fly

### Method #1: Reamaze reads directly from Conversations dynamodb (preferred)
This removes the need for apis (rate limiting issues go away), and also reduces the data transfer time.
- Conversations will call a reamaze /migrate api and provide an account_id
- Reamaze will schedule a background job that will read data from Conversations dynamodb and copy the data over
- Migration status can be polled by calling /accounts api. It will return status `migrating/active-recent-only/active`

Todos/Questions
  - Explore the cross account dynamodb access
  - Will the raw dynamob data for a thread/message need to be enhanced?
    eg: attachment urls, or telephony data
  - Do a POC and show how long it typically takes to do this

### Method #2: Via s3 dumps
  - Conversations will dump a single account's threads/messages/attachments into S3
  - Conversations will call a reamaze /migrate api and provide the S3 location
  - Reamaze /migrate api will schedule a job to do the migration
  - Status cannot be returned since the job will happen asynchronously. Reamaze might need to provide a /check-migration api that can be polled
  - Conversations will make two /migrate calls per account. One call to migrate most recent threads/messages. Another call to migrate the remaining
  - Do a POC and show how long it typically takes to do this


## Handling redundant requests
If Conversations calls /migrate api multiple times
- if account is already active, then we ignore the subsequesnt /mgrate request
- if account not-active and not-migrating, then we process the request


## Concerns about the 'ongoing migration' approach
There are a few reasons why we dont want to go this approach unless absolutely necessary.
- having to deal with deltas
- do we hae to worry about deletions account/website/channel/thread/message during?
  in reamaze deletion is cascading. if parent gets deleted, children gets deleted
- what if the account failed to create
  or the conversation didnt create
  how do we tell them that somethign went wrong?
    logging and reporting to slack can be monitored.
    what kind of error handling does Conversations want (if we go this approach)
