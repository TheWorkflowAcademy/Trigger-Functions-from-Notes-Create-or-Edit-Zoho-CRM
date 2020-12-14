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
In this set up, you will be frequently toggling between CRM and Flow, so it's a good idea to have 2 tabs/windows open for each app.

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

### Test the Webhook on Zoho Flow
Now that you have enabled notifications, you can test the signal by clicking *Test the Webhook* in Zoho Flow, then create/edit a CRM note. On success, you will be able to see a payload that looks like this.

```javascript
{
   query_params: {
      isdebug: "false"
   },
   module: "Notes",
   resource_uri: "https://www.zohoapis.com/crm/v2/Notes",
   ids: [
      "4371574000001619029"
   ],
   operation: "insert",
   channel_id: "1000000068001",
   token: null
}
```



### Create a Serverless Function in Zoho CRM
If you're not familiar with the idea of a serverless function, [please refer to this post](https://github.com/TheWorkflowAcademy/Zoho-CRM-Serverless-Functions). 
* In the serverless function, set `crmAPIRequest` as the argument with a `string` data type. Assuming that all you need here is the information of the Note record created/edited, you can just get the ID, then use a `getRecordsbyID` function.
 * Note: When you do `crmAPIRequest.get("body")`, it will return you the ID in a form of a list. To get the ID, use `.get(0)` to get the first index.
```javascript
id = crmAPIRequest.get("body").get(0);
note = zoho.crm.getRecordById("Notes",id);
```
* Once you have the ID, you can program any actions you want in this serverless function. However, the set up is not yet complete. For this to work, we need to send the ID from Flow to the serverless function that we've made. 
 * Go out of the function, click on the ellipsis and select REST API. Switch on the API KEY and `COPY` the URL.

### Write a Custom Function in Zoho Flow
Go back to Zoho Flow, and in the Flow Builder, create a custom function by going to Logic > + Custom Function. 
* Name the function
* Leave return type as *void*
* Set the Input Parameters as *id*, and choose *string* as the data type and hit create.
* Insert the script below into the function, save it, and link it to your Flow.
```javascript
void ServerlessReformator(string id)
{
  callServerlessFunction = invokeurl
  [
    url :"INSERT_REST_API_KEY_URL_HERE"
    type :POST
    parameters:id
  ];
  info callServerlessFunction;
}
```
Once this is done, you're now ALL SET! Whenever a Note is created/edited, CRM sends the Note ID to Zoho Flow which then sends an executes the serverless function.

TIP: To test that the setup is working properly, you can add a `sendmail` function to send the response to your email in your [serverless function script.](#create-a-serverless-function-in-zoho-crm)
