
# webmessage (inbound)
``` json
{
  "messageThread": {
    "messageThreadId": "fb10e8ee-cf29-455f-9719-ba5879a83653",              <---- external_reference_id (indexable)
    "lastMessageReadId": "ed1a5f5b-f09d-4f6c-82f7-1367fd831ee9",            <---- ?
    "lastMessageId": "ed1a5f5b-f09d-4f6c-82f7-1367fd831ee9",                <---- messages.last (derived)
    "isStarred": true                                                       <---- custom_attributes
  },
  "message": {
    "messageId": "ed1a5f5b-f09d-4f6c-82f7-1367fd831ee9",                    <---- dont need to store this
    "messageThreadId": "fb10e8ee-cf29-455f-9719-ba5879a83653",              <---- conversation.external_ref_id (derived)
    "createdDate": "2020-08-27T19:19:02.8406989Z",                          <---- create date
    "channel": "webmessage",                                                <---- conversations.category=internal (not a message attribute)
    "direction": "INBOUND",                                                 <---- not stored
    "channelFrom": "+16159429139",                                          <---- C2 (inbound)
    "channelTo": "84e75891-9b29-4c01-82b9-f447421ee1b1",                    <---- C1
    "deliveryState": "RECEIVED",                                            <---- ?
    "body": "Hi. Your sweaters look really nice. Can we collaborate?"       <---- body
  }
}
```

# webmessage (outbound)
``` json
{
  "messageThread": {
    "messageThreadId": "fb10e8ee-cf29-455f-9719-ba5879a83653",
    "lastMessageReadId": "ed1a5f5b-f09d-4f6c-82f7-1367fd831ee9",
    "lastMessageId": "ed1a5f5b-f09d-4f6c-82f7-1367fd831ee9"
  },
  "message": {
    "messageThreadId": "7347ff28-1efc-455f-969c-8fd19103f591",
    "messageId": "810423ea-555a-4053-aa39-32136edb3292",
    "createdDate": "2021-02-23T21:13:37.147Z",
    "channel": "webmessage",
    "channelFrom": "ef90d2ea-f131-4c7a-9dad-b28f82403bdb",                  <---- C1 (outbound)
    "channelTo": "+18432275878",                                            <---- C2
    "direction": "OUTBOUND",
    "deliveryState": "DELIVERED",
    "body": "Oy thats a nice pair of headphones"
  }
}
```

# SMS message (inbound)
``` json
{
  "messageThread": {
    "messageThreadId": "3d09fda1-e85f-4441-89ae-d22a020ec975",
    "lastMessageReadId": "e1430cf9-87ec-4f94-8677-050f93087307",
    "lastMessageId": "e1430cf9-87ec-4f94-8677-050f93087307"
  },
  "message": {
    "messageThreadId": "27b1033c-e3cf-4f1d-bcaa-ec4e91a25982",
    "messageId": "63c123c0-4a83-4c52-9362-eb8d53f11867",
    "createdDate": "2021-07-23T22:07:59.853Z",
    "channel": "sms",                                                       <---- conversations.category=email ?
    "channelFrom": "+13136105873",
    "channelTo": "+18325019979",
    "direction": "INBOUND",
    "deliveryState": "RECEIVED",
    "media": [
      {
        "mimetype": "image/jpeg",                                           <---- http header ContentType
        "url": "https://media.cloud.vnm.godaddy.com/media/v1/CdUypQTqK"     <---- attachment_id
      }
    ]
  }
}
```

# SMS message (outbound)
``` json
{
  "messageThread": {
    "messageThreadId": "3d09fda1-e85f-4441-89ae-d22a020ec975",
    "lastMessageReadId": "e1430cf9-87ec-4f94-8677-050f93087307",
    "lastMessageId": "e1430cf9-87ec-4f94-8677-050f93087307",
    "isArchived": false                                                     <---- status=archived (add logic to handle archiving)
  },
  "message": {
    "messageId": "e1430cf9-87ec-4f94-8677-050f93087307",
    "messageThreadId": "3d09fda1-e85f-4441-89ae-d22a020ec975",
    "createdDate": "2021-03-29T21:53:03.409Z",
    "channel": "sms",
    "channelFrom": "+14047773643",
    "channelTo": "+16787587921",
    "direction": "OUTBOUND",
    "deliveryState": "DELIVERED",
    "body": "Negative books are real books"
  }
}
```

# internal message (inbound)
``` json
{
  "messageThread": {
    "messageThreadId": "26643b3b-0367-4e7f-8902-7c44c86a4394",
    "lastMessageId": "4ed5f47e-071b-4906-9412-5454b3ba7ab6",
    "endpoints": [
      {
        "channel": "Internal",                                              <---- ignore. get from json.message.channel instead
        "firstName": "GoDaddy Guide Bot",                                   <---- ? (need a special system user with this name)
        "channelFrom": "Conversations"                                      <---- ? 
      }
    ],
    "isArchived": false,                                                    <---- status=arhived (plus archive logic)
    "isSpam": false                                                         <---- status=spam    (plus spam logic)
  },
  "message": {
    "messageThreadId": "26643b3b-0367-4e7f-8902-7c44c86a4394",
    "messageId": "4ed5f47e-071b-4906-9412-5454b3ba7ab6",
    "createdDate": "2021-10-12T16:32:46.232Z",
    "channel": "internal",                                                  <---- conversations.category=internal
    "channelFrom": "Conversations",
    "channelTo": "AjEWSYTZEXaByvZmRbtW2c",                                  <---- productInstanceId. should be mapped to C1
    "direction": "INBOUND",
    "deliveryState": "RECEIVED",
    "body": "Welcome to your unified conversations inbox! Reply to messages"
  }
}
```

# voip message (inbound)
``` json
{
  "messageThread": {
    "messageThreadId": "2d5d393c-7cfc-4102-8c66-c27373e9daee",
    "lastMessageId": "fe14c306-f4e0-41c0-bd0b-5ae729d91fb4",
    "endpoints": [
      {
        "channel": "Voip",                                                  <---- ignore. get from json.message.channel instead
        "channelFrom": "+16145713211"                                       <---- ignore. get from json.message.channelFrom instead
      }
    ],
    "isArchived": false,
    "isSpam": false
  },
  "message": {
    "messageThreadId": "2d5d393c-7cfc-4102-8c66-c27373e9daee",
    "messageId": "fe14c306-f4e0-41c0-bd0b-5ae729d91fb4",
    "createdDate": "2021-09-19T22:12:12.135Z",
    "traits": [
      {
        "name": "callSubType",                                              <---- custom_attributes trait_callSubType (single level)
        "value": "inboundMissed"
      },
      {
        "name": "callSid",                                                  <---- custom_attributes trait_callSid
        "value": "CA4d48dff20b4cd8e4480263abfcb47d35"
      }
    ],
    "channel": "voip",                                                      <---- conversations.category=voip ?
    "channelFrom": "+16145713211",
    "channelTo": "+14104985652",
    "direction": "INBOUND",
    "deliveryState": "RECEIVED",
    "media": [
      {
        "mimetype": "audio/x-wav",                                          <---- http header ContentType
        "audioLengthInSeconds": 25,                                         <---- ? (custom_attribute on attachment)
        "url": "https://media.cloud.vnm.godaddy.com/media/v1/RDsizqGc"      <---- attachment_id
      }
    ],
    "body": "so that we can discuss and take necessary action"              <---- ? (transcript. empty if missed call or unable transcribe) 
  }
}
```

# voip message (outbound)
``` json
{
  "messageThread": {
    "messageThreadId": "3d09fda1-e85f-4441-89ae-d22a020ec975",
    "lastMessageReadId": "e1430cf9-87ec-4f94-8677-050f93087307",
    "lastMessageId": "e1430cf9-87ec-4f94-8677-050f93087307"
  },
  "message": {
    "messageThreadId": "247ca993-fae6-4d21-820a-70cbd637ff51",
    "messageId": "837cb023-7f4a-42a8-96b7-9d76aefdc71e",
    "createdDate": "2021-08-21T09:16:43.978Z",
    "traits": [
      {
        "name": "callSubType",
        "value": "outbound"
      },
      {
        "name": "callSid",
        "value": "CAf071554ae414923ad34c1b743ef13388"
      },
      {
        "name": "callDuration",                                             <---- custom_attributes trait_callDuration
        "value": "7"
      }
    ],
    "channel": "voip",
    "channelFrom": "+17706705434",
    "channelTo": "+14704551021",
    "direction": "OUTBOUND",
    "deliveryState": "RECEIVED",
  }
}
```

