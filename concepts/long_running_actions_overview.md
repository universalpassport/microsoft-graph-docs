---
author: rgregg
ms.author: rgregg
ms.date: 09/10/2017
ms.topic: conceptual
---
# Working with long running actions (beta)

> **Important:** APIs under the /beta version in Microsoft Graph are in preview and are subject to change. Use of these APIs in production applications is not supported.

Some API responses require indeterminate time to complete.
Instead of waiting until the action is complete before returning a response, Microsoft Graph may use a long running actions pattern.
This pattern provides your app a wait to poll for status updates on a long running action, without any request waiting for the action to complete.

The general pattern follows these steps:

1. Your app requests a long running action via the API. The API accepts the action and returns a `202 Accepted` response along with a Location header for the API URL to retrieve action status reports.
2. Your app requests the action status report URL and receives an [AsyncJobStatus](../api-reference/beta/resources/asyncjobstatus.md) response with the progress of the long running action.
3. The long running action completes. 
4. Your app requests the action status report URL again and receives an [AsyncJobStatus](../api-reference/beta/resources/asyncjobstatus.md) response showing the completion of the action.

## Initial action request

Let's walk through the steps for an example [DriveItem Copy](../api-reference/beta/api/driveitem_copy.md) scenario.
In this scenario, your app requests to copy a folder that contains a large amount of data.
This request will likely take several seconds to complete since the amount of data is large.

<!-- { "blockType": "request", "name": "lro-copy-item-example", "scopes": "files.readwrite" } -->

```http
POST https://graph.microsoft.com/beta/me/drive/items/{folder-item-id}/copy
Content-Type: application/json

{
  "parentReference": {
    "path": "/drive/root:/Documents"
  },
  "name": "Copy of LargeFolder1"
}
```

The API responds that the action was accepted and the URL for retrieving the status of the long running action.

<!-- { "blockType": "response" } -->

```http
HTTP/1.1 202 Accepted
Location: https://api.onedrive.com/monitor/4A3407B5-88FC-4504-8B21-0AABD3412717
```

**Note:** The location URL returned may not be on the Microsoft Graph API endpoint.

In many cases this may be the end of the request, since the copy action will complete without the app doing any additional work.
However, if your app needs to show the status of the copy action or ensure that it completes without error, it can do so using the monitor URL.

## Retrieve a status report from the monitor URL

To check on the status of the copy action, the app makes a request to the URL provided in the previous response.
*Note:* This request does not require authentication, since the URL is short-lived and unique to the original caller. 

<!-- { "blockType": "request", "name": "lro-check-status", "scopes": "files.readwrite" } -->

```http
GET https://api.onedrive.com/monitor/4A3407B5-88FC-4504-8B21-0AABD3412717
```

The service responses with information that the long running action is still in progress:

<!-- { "blockType": "response", "@odata.type": "microsoft.graph.asyncJobStatus" } -->

```http
HTTP/1.1 202 Accepted
Content-type: application/json

{
  "operation": "ItemCopy",
  "percentageComplete": 27.8,
  "status": "inProgress"
}
```

This information can be used to provide an update to the user about the progress of the copy action.
The app can continue to poll the monitor URL to request status updates and keep track of the progress of the action.

## Retrieve a completed status report from the monitor URL

After a few seconds the copy operation has completed.
This time when the app makes a request to the monitor URL the response is a redirection to the finished result of the action.

<!-- { "blockType": "request", "name": "lro-check-status-complete", "scopes": "files.readwrite" } -->

```http
GET https://api.onedrive.com/monitor/4A3407B5-88FC-4504-8B21-0AABD3412717
```

When the action has completed, the response from the monitor service will return the resourceId for the results.

<!-- { "blockType": "response", "@odata.type": "microsoft.graph.asyncJobStatus" } -->

```http
HTTP/1.1 202 Accepted
Content-type: application/json

{
    "percentageComplete": 100.0,
    "resourceId": "01MOWKYVJML57KN2ANMBA3JZJS2MBGC7KM",
    "status": "completed"
}
```

## Retrieve the results of the completed operation

Once the job has completed, the monitor URL returns the resourceId of the result, in this case the new copy of the original item.
You can address this new item using the resourceId, for example:

<!-- { "blockType": "request", "name": "lro-copy-item-example-complete", "scopes": "files.readwrite" } -->

```http
GET https://graph.microsoft.com/beta/me/drive/items/01MOWKYVJML57KN2ANMBA3JZJS2MBGC7KM
```

<!-- { "blockType": "response", "@odata.type": "microsoft.graph.driveItem" } -->

```http
HTTP/1.1 200 OK
Content-type: application/json

{
    "id": "",
    "name": "Copy of LargeFolder1",
    "folder": { },
    "size": 12019
}
```

## Supported resources

Long running actions are supported on the following API methods

| **Resource** | **API** |
|:------ | :------ |
| DriveItem | [Copy](../api-reference/beta/api/driveitem_copy.md) |

## Prerequisites

The same [permissions](./permissions_reference.md) that are required to perform a long running action are also required to query the status of a long running action.




<!-- {
  "type": "#page.annotation",
  "description": "Monitor the progress of long-running actions in the API.",
  "keywords": "monitor,long,running,operation,action",
  "section": "documentation",
  "tocPath": "Concepts/Long running actions"
} -->
