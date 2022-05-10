# Sending Azure Service Health Advisories to Slack

## Creating a webhook to send messages to

1. Create a new app at https://api.slack.com/apps/
2. Click on **"Add features and functionality"**
3. Click on **incoming webhook**
4. If not already done, enable the feature by toggling the slider in the top right corner
5. Click the **"Add New Webhook to Workspace"** button, and follow through the guided steps to add the webhook to your chosen channel.

## Creating a service health alert
1. In Azure, search for and go to the **Service Health** product.
2. Click on **"Add service health alert"**
3. Select services, regions, and health criteria as desired (selecting all is okay)
4. Click on **"Select action groups"**
5. Create a new action group
6. Finish creating your alert rule by giving it a name 

## Creating a logic app
1. In Azure, search for and go to the **Logic App** product.
2. Create a new consumption logic app
3. Use the logic app flow to perform the following actions:
    1. Enter the schema of the json payload of the service alert (use example payload to generate it automatically)
    2. Parse the JSON of the body
        * Name the action `Parse Body JSON`
    4. Escape the body of the alert (**body('Parse_JSON')?['properties']?['communication']**)
        * Name action `Escape_Body`
    5. Post the title (**body('Parse_JSON')?['properties']?['title']**) to Slack using the Logic App's built-in Slack integration tool
    6. Parse the JSON of the resulting response (once again use an example response to generate automatically)
        * Name the action `JSON: Sent slack msg`
    7. Make a POST request to the webhook URL with the following properties:
        * URI should be your webhook URL
        * Header should set "Content-Type" to "application/json" (without ")
        * Body should be `concat('{"thread_ts": "', body('JSON:_Sent_slack_msg')?['ts'], '", "text": "', outputs('Escape_Body'), '"}')`
            * Replace `JSON:_Sent_slack_msg` with the name of the connector you made in step 3.iv

And you're all done! The title should now be posted to the channel of your choosing, and then the (sometimes somewhat spammy) body should end up in the thread under it.
