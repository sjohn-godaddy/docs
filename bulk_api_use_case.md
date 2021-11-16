
## Reasons for a bulk api for migrating conversations from Conversations to Re:amaze

- rate limiting
    - there are 2.7 million sms messages to migrate. if there is no bulk api, and we used the regular api, we will be calling the api 10000s of times
    - concerns about rate limiting
    - concerns about api processing speed versus backlog of incoming api calls
- data loss resiliency
    - if there is no bulk api, Conversations would need to manage a queue to hold the import messages that are being migrated over. 
    - SkipMorse prefers that such a queue is better if its implemented on the receiving target (reamaze platform), rather than the source (Conversations platform)
- default behavior of regular apis
    - concerns that the regular api assumes that a message is being created 'right now', as oposed to a message that was created 6 months ago
    - similar related concerns about other pieces of data on a message (eg: attchments)
- deduplication concerns
    - if Conversations were to send the same migration twice for the same message (hypothetically because they didnt get back acknowledgement that a message made it into the queue), can the api figure that out?. this is a concern for both bulk api and regular api
- volume/size
    - there are 2.7M sms messages (1kb each). plus attachments
    - if we use regular api that means 1 api call for the message, and n api calls for each attachment. that doubles the number api calls to 2.7 x 2 = 5 million (possibly)
    - in a bulk api we could expect messages+attachment to be imported at the same time

