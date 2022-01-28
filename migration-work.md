### Conversations Migration: Work Breakdown

- Add migration-status to Account, Brand, Category (not as columns, but as json data). Default is 'None'
- v2/accounts: should return the migration-status at the Account, Brand, Category level
- v2/accounts: if Account migration-status is None, reamaze should make a call to Conversations api to check if account exists there. if exists, v2/accounts should populate the migration-status on reamaze Account/Brand/Category
- v2/check-migration-status: Should return minimal information about the entire account's migration status, as well as the migration status of each website, channel. Keep the payload very small, this will be called tons of times
- v2/start-migration: Should drop a background job and return success
- migration background job: 
  - Should call Conversations api to get migration payload (for a specific C1)
  - At this point we can either process everything together, or split into different jobs
    Set migration-status for account/brand/channel appropriately at each step
    - For each account, update the account on reamaze. set status=migrating
    - For each account, create/update brands on reamaze. set status=migrating
    - For each account, for each brand, create/update channels on reamaze. set status=migrating
    - For each account, for each brand, for each channel, initiate phone number porting (async)
    - For each account, for each brand
      - Import recent threads/messages from dynamodb
      - Merge sms and void threads (by C1-to-C2)
      - Set brand status to recent-messages-migrated
      - Set account status to recent-messages-migrated if all brands completed
    - For each account, for each brand
      - Import remaining threads/messages from dynamodb
      - Merge sms and void threads (by C1-to-C2)
      - Set brand status to all-messages-migrated
      - Set account status to all-messages-migrated if all brands completed
- Phone number porting callback: when phone number is ported, flip channel status to 'ready' 
- Handle failure in channel creation/updation. 
    Set brand.migration_status=channel-failure
    account.migration_status=brand-failure
    v2/check-migration-status would return these values
    (the mobile app, is expected to retry calling v2/start-migration with exponential intervals)

- Handle failure in channel phone porting. 
    This happens on its own separate process, not directly related to the messages migration. When failure is detected, set the channel.migration-status=failure, and also notify its corresponding brand if necessary.

- Handle failure in loading recent messages. 
    Set the brand/account migration-status accordingly
    v2/check-migration-status would return these values
    (the mobile app, is expected to retry calling v2/start-migration with exponential intervals)

- Handle failure in loading all remaining messages. 
    This is different because we want to update the migration-statuses accordingly, but we want the mobile app to ignore this and continue normal usage. Write a aws script that periodically queries such records and report it somewhere

- Handle failure in migrating attachments. 
    This failure can be ignored. No migration-status updates here. but we need to record the error on the Attachment object, or elsewhere, so that we can query for such records later and fix them manually

- Add ability to pick up migration where it was left off, in case of failure of some kind. Reamaze should use the migration-status on account, brand, channel to understand where to restart the migration. This might mean removing partially create objects: Brands, Channels, PhonePorting, MessageThreads, Messages

- Add api to force a migration to clean everything out (brands, channels, messages) and restart from the beginning. This is for Conversations team development purposes. Only needed in dev-private and test.

- Work needed for frontend apps
  - Call the v2/accounts api, handle the different statuses
  - Call the v2/start-migration api
  - Call the v2/check-migration-status api periodically
  - If migration fails, retry with exponential backup
