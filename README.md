# Trigger-Functions-from-Notes-Create-or-Edit-Zoho-CRM
A workaround that allows Notes (Create/Edit) to be a function trigger in Zoho CRM.

## Core Idea
Workflow Rules in Zoho CRM doesn't allow *Notes* to be set as a function trigger. While waiting for Zoho to support this, we have found a workaround that enables function triggering from *Notes* (create/edit). Here's the idea:

[Enable Notifications](https://www.zoho.com/crm/developer/docs/api/v2/notifications/overview.html) -> [Zoho Flow](https://www.zoho.com/flow/) -> [Serverless Function](https://github.com/TheWorkflowAcademy/Zoho-CRM-Serverless-Functions)

1. Enable Notifications in CRM that sends a webhook to Zoho Flow on trigger of *Notes* (edit/create).
2. Zoho Flow receives and interprets the data package, then sends the ID to a serverless function in CRM.
3. The Serverless Function gets the ID and executes the custom function.

## Configuration
The following Zoho CRM Connection scope is needed:
* ZohoCRM.notifications.ALL

## Tutorial

### Create a Zoho Flow Webhook
Login to Zoho Flow and create a new Flow based on this settings: 
* Create Flow -> Webhook -> Payload Format (JSON)

Once this is done, *COPY* the webhook that you have created.

### Enable Notifications

Notification APIs allow you to get instant notifications whenever an action is performed (create/update/delete) on the records of a module. The system notifies you of the event on the URL provided. The [Zoho Flow webhook that you have created](#create-a-zoho-flow-webhook) will be set as the URL here.
* channel_id
  * The given value is sent back in notification URL body to make sure that the notification is for a particular channel. Can be any arbitrary number.
* events
  * The module action(s) for triggger (Module.action). The possible actions are create/edit/delete.
* channel_expiry
  * Notifications expiry time. Maximum of only one day from the time they were enabled. If it is not specified or set for more than a day, the default expiry time is for one hour. Accepts only Zoho date-time format.
  * Here, we would want to set the maximum expiry (1 day after enabling notifications) and have a [scheduled function](https://help.zoho.com/portal/en/kb/crm/automate-business-processes/schedules/articles/custom-schedules) to run daily so that the notification stays active at all times.
  * [Check out this post on how to get current Zoho date-time and format it.](https://github.com/TheWorkflowAcademy/Date-Time-Format-Conversion-Zoho-Deluge)
* notify_url
  * URL to be notified (POST request). Whenever any action gets triggered, the notification will be sent through this notify url.
  * Insert the Zoho Flow Webhook URL here.

```javascript
param = 
{
  "watch":
   {
     {
        "channel_id":"1", //Arbitrary number
        "events":{"Notes.create","Notes.edit"}, //The module actions for trigger
        "channel_expiry":dateTime_FORMATTED, //If not set, default expiry is 1 hour
        "notify_url": "INSERT_ZOHO_FLOW_WEBHOOK_HERE"
     } 
   }
};


```
