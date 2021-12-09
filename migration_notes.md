#  Migration Discussion

## Let's migrate on the fly
Migrate on demand. Its simpler and we dont have to worry about deltas or syncing. Migrate the accounts, brands, channels via existing api. the volume is small, and you get immediate feedback about success/failure

1. Before the migration every C1 in GoDaddy will get WebChat channel, and that will create a reamaze account with `status=active, migration_status=not_started`
2. Client downloads new version of the mobile app
   - When app is launched by user, the first thing it does is call reamaze accounts index api. 
   - Reamaze in turn will then call Conversation api to check if this account needs to be migrated. Reamaze will return migration_status to the mobile app in the response. 
     - This step is needed because reamaze cannot differentiate between an account that needs migration vs a new account that was never in Conversations db (and hence doesnt need migration). If migration is not needed reamaze should mark `migration_status=not_needed`
   - If migration is needed, Mobile app will call a new reamaze endpoint /start-migration. 
     - Client needs to poll for something to check if migration is finished.
     - Reamaze needs to expose the api for polling (could be accounts api itself)
   - Reamaze /start-migration will:
     - Mark the account as migrating: `status=active, migration_status=migrating`
     - Drop a background job to do the migration
   - Reamaze migration background job will:
     - Call Conversations /get_channels_to_migrate endpoint, get the data and update itself
     - We cannot use direct dynamodb access here because of application-level encryption in Conversations
     - Update `migration_status=channels-only`, so that client can move to 'Inbox view' when it polls for migration status
     - Read recent threads/messages from dynamodb and adds into reamaze
     - Update `migration_status=recent-messages-only`, so that client can pull and show recent messages
     - Read all threads/messages from dynamodb and adds into reamaze
     - Update `migration_status=completed`, so that client can pull and show all messages



   - Conversations /start-migration will:
       1. Call the brands/channels create/update apis (some brands to create, some to update, no deletion though)
       2. -- mobile client goes into inbox view at this point --
       3. Call /migrate to migrate the most recent threads/messages
       4. Poll /check-migration to check status
       5. Call accounts api to update `migration_status=recent-only`
       6. Call /migrate to migrate the remaining recent threads/messages
       7. Poll /check-migration to check status
       8. Call accounts api to update `migration_status=completed`



Questions
- Will reamaze remember where it left off between steps 3 and 6, or maybe step 6 overwrites all of the threads/messages
- How to handle attachment reading failure during migration. We can skip failed attachments and move on
- Can we redo the migration? can reamaze re-read all the data from Conversations and re-populate reamaze db again (for that account). Add a force option whenever Conversations wants to force re-migration of an account. This is useful for development/testing purposes.
- When account create api is called (after migration), how will reamaze know whether this account needs to be migrated or not? Reamaze should call Conversation api to ask about this account. If account is not found in Conversations then its doesnt need migration.

>>>>>>>> TODO TODO
>>>>>>>> TODO TODO

How to handle failures during migration
1. Call the brands/channels create/update apis (some brands to create, some to update, no deletion)
     
3. Call /migrate to migrate the most recent threads/messages
4. Poll /check-migration to check status
5. Call accounts api to update `status=active-recent-only`
6. Call /migrate to migrate the remaining recent threads/messages
7. Poll /check-migration to check status
8. Call accounts api to update `status=active`

  - Network failure reading a specific record from dynamo. We'll want to retry x times. After that what do we do? is there a dead letter queue equivalent. we can move on with the migration, skip this message. Handle this dead letter queue items later next weekend.
  - Can we skip threads/messages that fail to migrate?
  - Do we need a threshold 'if x failures happen, then mark this migration as errored'
  - At some point mark the migration as failure?



## Migrating thread/messages on the fly

### Method #1: Reamaze reads directly from Conversations dynamodb (preferred)
This removes the need for apis (rate limiting issues go away), and also reduces the data transfer time.
- Conversations will call a reamaze /migrate api and provide an account_id
- Reamaze will schedule a background job that will read data from Conversations dynamodb and copy the data over
- Migration status can be polled by calling /accounts api. It will return status `migrating/active-recent-only/active`

Todos/Questions
  - Explore the cross account dynamodb access (Skip/Dennis can open the access policy)
  - Will the raw dynamodb data for a thread/message need to be enhanced? eg: attachment urls, or telephony data
    - Attachments are media urls within dynamodb. requires shopper jwt which reamaze doesnt have
    - Conversations can open media service to use cert jwt, so that reamaze can access these media urls by providing cert jwt
    - No other data in dynamodb needs to be enhanced
  - Do a POC and show how long it typically takes to do this

### Method #2: Via S3 dumps
  - Conversations will dump a single account's threads/messages/attachments into S3
  - Conversations will call a reamaze /migrate api and provide the S3 location
  - Reamaze /migrate api will schedule a job to do the migration
  - Status cannot be returned since the job will happen asynchronously. Reamaze might need to provide a /check-migration api that can be polled
  - Conversations will make two /migrate calls per account. One call to migrate most recent threads/messages. Another call to migrate the remaining
  - Do a POC and show how long it typically takes to do this


## Handling redundant requests
When Conversations calls /migrate api multiple times
- If account is already active, then we ignore the subsequesnt /migrate request
- If account not-active and not-migrating, then we process the request
- Also add a force option whenever Conversations wants to force re-migration of an account

## Concerns about the 'ongoing migration' approach
There are a few reasons why we dont want to go this approach unless absolutely necessary.
- Having to deal with deltas
- Do we have to worry about deletions account/website/channel/thread/message during? In reamaze deletion is cascading. if parent gets deleted, children gets deleted
- What if the account failed to create. Or the conversation didnt create. How do we tell them that somethign went wrong? Logging and reporting to slack can be monitored. What kind of error handling does Conversations want (if we go this approach)
