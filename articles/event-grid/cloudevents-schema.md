---
title: Use Azure Event Grid with events in CloudEvents schema
description: Describes how to set the CloudEvents schema for events in Azure Event Grid.
services: event-grid
author: banisadr
manager: timlt

ms.service: event-grid
ms.topic: conceptual
ms.date: 05/09/2018
ms.author: babanisa
---

# Use CloudEvents schema with Event Grid

In addition to its [default event schema](event-schema.md), Azure Event Grid natively supports events in the [CloudEvents JSON schema](https://github.com/cloudevents/spec/blob/master/json-format.md). [CloudEvents](http://cloudevents.io/) is an [open standard specification](https://github.com/cloudevents/spec/blob/master/spec.md) for describing event data in a common way.

CloudEvents simplifies interoperability by providing a common event schema for publishing, and consuming cloud based events. This schema allows for uniform tooling, standard ways of routing & handling events, and universal ways of deserializing the outer event schema. With a common schema, you can more easily integrate work across platforms.

CloudEvents is being build by several [collaborators](https://github.com/cloudevents/spec/blob/master/community/contributors.md), including Microsoft, through the [Cloud Native Compute Foundation](https://www.cncf.io/). It's currently available as version 0.1.

This article describes how to use the CloudEvents schema with Event Grid.

[!INCLUDE [event-grid-preview-feature-note.md](../../includes/event-grid-preview-feature-note.md)]

## CloudEvent schema

Here is an example of an Azure Blob Storage event in CloudEvents format:

``` JSON
{
    "cloudEventsVersion" : "0.1",
    "eventType" : "Microsoft.Storage.BlobCreated",
    "eventTypeVersion" : "",
    "source" : "/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.Storage/storageAccounts/{storage-account}#blobServices/default/containers/{storage-container}/blobs/{new-file}",
    "eventID" : "173d9985-401e-0075-2497-de268c06ff25",
    "eventTime" : "2018-04-28T02:18:47.1281675Z",
    "data" : {
      "api": "PutBlockList",
      "clientRequestId": "6d79dbfb-0e37-4fc4-981f-442c9ca65760",
      "requestId": "831e1650-001e-001b-66ab-eeb76e000000",
      "eTag": "0x8D4BCC2E4835CD0",
      "contentType": "application/octet-stream",
      "contentLength": 524288,
      "blobType": "BlockBlob",
      "url": "https://oc2d2817345i60006.blob.core.windows.net/oc2d2817345i200097container/oc2d2817345i20002296blob",
      "sequencer": "00000000000004420000000000028963",
      "storageDiagnostics": {
        "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
      }
    }
}
```

CloudEvents v0.1 has the following properties available:

| CloudEvents        | Type     | Example JSON Value             | Description                                                        | Event Grid Mapping
|--------------------|----------|--------------------------------|--------------------------------------------------------------------|-------------------------
| eventType          | String   | "com.example.someevent"          | Type of occurrence that happened                                   | eventType
| eventTypeVersion   | String   | "1.0"                            | The version of the eventType (Optional)                            | dataVersion
| cloudEventsVersion | String   | "0.1"                            | The version of the CloudEvents specification the event uses        | *passed through*
| source             | URI      | "/mycontext"                     | Describes the event producer                                       | topic#subject
| eventID            | String   | "1234-1234-1234"                 | ID of the event                                                    | id
| eventTime          | Timestamp| "2018-04-05T17:31:00Z"           | Timestamp of when the event happened (Optional)                    | eventTime
| schemaURL          | URI      | "https://myschema.com"           | A link to the schema that the data attribute adheres to (Optional) | *not used*
| contentType        | String   | "application/json"               | Describe the data encoding format (Optional)                       | *not used*
| extensions         | Map      | { "extA": "vA", "extB", "vB" }  | Any additional metadata (Optional)                                 | *not used*
| data               | Object   | { "objA": "vA", "objB", "vB" }  | The event payload (Optional)                                       | data

For more information, see the [CloudEvents spec](https://github.com/cloudevents/spec/blob/master/spec.md#context-attributes).

## Configure Event Grid for CloudEvents

Currently, Azure Event Grid has preview support for CloudEvents JSON format input and output in **West Central US**, **Central US**, and **North Europe**.

You can use Event Grid for both input and output of events in CloudEvents schema. You can use CloudEvents for system events, like Blob Storage events and IoT Hub events, and custom events. It can also transform those events on the wire back and forth.


| Input schema       | Output schema
|--------------------|---------------------
| CloudEvents format | CloudEvents format
| Event Grid format  | CloudEvents format
| CloudEvents format | Event Grid format
| Event Grid format  | Event Grid format

For all event schemas, Event Grid requires validation when publishing to an event grid topic and when creating an event subscription. For more information, see [Event Grid security and authentication](security-authentication.md).

### Input schema

To set the input schema on a custom topic to CloudEvents, use the following parameter in Azure CLI when you create your topic `--input-schema cloudeventv01schema`. The custom topic now expects incoming events in CloudEvents v0.1 format.

To create an event grid topic, use:

```azurecli
# if you have not already installed the extension, do it now.
# This extension is required for preview features.
az extension add --name eventgrid

az eventgrid topic create \
  --name <topic_name> \
  -l westcentralus \
  -g gridResourceGroup \
  --input-schema cloudeventv01schema
```

The current version of CloudEvents doesn't support batching of events. To publish events with CloudEvent schema to a topic, publish each event individually.

### Output schema

To set the output schema on an event subscription to CloudEvents, use the following parameter in Azure CLI when you create your Event Subscription `--event-delivery-schema cloudeventv01schema`. Events for this event subscription are now be delivered in CloudEvents v0.1 format.

To create an event subscription, use:

```azurecli
az eventgrid event-subscription create \
  --name <event_subscription_name> \
  --topic-name <topic_name> \
  -g gridResourceGroup \
  --endpoint <endpoint_URL> \
  --event-delivery-schema cloudeventv01schema
```

The current version of the CloudEvents doesn't support batching of events. An event subscription that's configured for CloudEvent schema receives each event individually.

## Next steps

* For information about monitoring event deliveries, see [Monitor Event Grid message delivery](monitor-event-delivery.md).
* We encourage you to test, comment on, and [contribute](https://github.com/cloudevents/spec/blob/master/CONTRIBUTING.md) to CloudEvents.
* For more information about creating an Azure Event Grid subscription, see [Event Grid subscription schema](subscription-creation-schema.md).
