# Trigger-Functions-from-Notes-Create-or-Edit-Zoho-CRM
A workaround that allows Notes (Create/Edit) to be a function trigger in Zoho CRM.

## Core Idea
Unlike other modules in CRM, Workflow Rules doesn't allow you to set *Notes* as a function trigger. Till that day arrives, we have found a workaround that enables function triggering from Notes (create/edit). Here's the idea:

[Enable Notifications](https://www.zoho.com/crm/developer/docs/api/v2/notifications/overview.html) -> [Serverless Function](https://github.com/TheWorkflowAcademy/Zoho-CRM-Serverless-Functions)

1. Enable Notifications API for the Notes module that sends data on trigger of Notes (create/edit) to a serverless function in CRM.
2. The serverless function receives the Note ID along with other data and executes the custom function.

## Configuration
The following Zoho CRM Connection scope is needed:
* ZohoCRM.notifications.ALL

## Tutorial

### Create a Serverless Function in Zoho CRM
If you're not familiar with the idea of a serverless function, [please refer to this post](https://github.com/TheWorkflowAcademy/Zoho-CRM-Serverless-Functions). 
* In the serverless function, set `crmAPIRequest` as a *string* argument. Assuming that all you need here is the information of the Note record, you can just get the ID, then use a `getRecordsbyID` function to fetch the record details.
 * Note: When you do `crmAPIRequest.get("body")`, it will return you the ID in a form of a list. To get the ID, `.get(0)` is used to get the first index.
```javascript
id = crmAPIRequest.get("body").get(0);
note = zoho.crm.getRecordById("Notes",id);
```
* Tip: To test that the setup is working initially, you can configure a sendmail function in the serverless to send the response to your email.
* Save the serverless function and create an API key for it (you need this key to enable notifications in the next step).


### Enable Notifications

Notification APIs allow you to get instant notifications whenever an action is performed (create/update/delete) on the records of a module. The system notifies you of the event to the URL provided. The CRM serverless function API key will be set as the URL here.
* channel_id
  * The given value is sent back in notification URL body to make sure that the notification is for a particular channel. Can be any arbitrary number.
* events
  * The module action(s) for triggger (Module.action). The possible actions are create/edit/delete.
* channel_expiry
  * Notifications expiry time. Maximum of only one day from the time they were enabled. If it is not specified or set for more than a day, the default expiry time is for one hour. Accepts only Zoho date-time format.
  * Here, we would want to set the maximum expiry (1 day after enabling notifications) and have a [scheduled function](https://help.zoho.com/portal/en/kb/crm/automate-business-processes/schedules/articles/custom-schedules) to run daily so that the notification stays active at all times.
  * [Check out this post on how to get the current Zoho date-time and convert it into a parseable format for update.](https://github.com/TheWorkflowAcademy/Date-Time-Format-Conversion-Zoho-Deluge)
* notify_url
  * URL to be notified (POST request). Whenever any action gets triggered, the notification will be sent through this notify url.
  * Insert the Zoho CRM Serverless Function API Key here.

```javascript
param = 
{
  "watch":
   {
     {
        "channel_id":"1",
        "events":{"Notes.create","Notes.edit"}, 
        "channel_expiry":"INSERT_DATE_TIME_HERE",
        "notify_url": "INSERT_ZOHO_FLOW_WEBHOOK_HERE"
     } 
   }
};

response = invokeurl
[
	url :"https://www.zohoapis.com/crm/v2/actions/watch"
	type :POST
	parameters:param + ""
	connection:"INSERT_ZOHO_CRM_CONNECTION_HERE"
];
info response;
```
*Note: To check if your notification is active, you can use the [notification details API.](https://www.zoho.com/crm/developer/docs/api/v2/notifications/get-details.html)*

Now that you have enabled notifications, you can test the signal by creating/editing a CRM note. On success, you will be able to see the payload data.

If you have sendmail configured in the serverless function, you can verify that the setup is working by receiving the data in your email.
Once this is done, you’re ALL SET! You've now successfully set up a Note action (create/edit) as a workflow trigger where you can configure custom actions in the serverless function.
