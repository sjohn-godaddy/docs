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
