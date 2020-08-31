﻿# Issue Notification Bot

This bot tracks GitHub issues in the Bot Framework repos and sends notifications via Teams to the assignees if:

* Issue is tagged with `customer-reported`, and:
  * Issue has not been tagged with `customer-replied` in last 3 business days, or
  * Issue has not been closed in the last 30 days, or
  * Issue has not been closed in the last 90 days

*Note: These are based on SLA timelines*

It also notifies users daily of stale PRs.

It is broken up into two main folders:

1. AzureFunction - This lives in an Azure Function and queries GitHub every half hour
2. Bot - This is the bot. It received data from the Azure Function and sends notifications via Teams

## Azure Function

Every half hour, the Azure Function queries GitHub for all issues within the Bot Framework. It categorizes them based on tags and whether or not they've been replied to, then sends them to the bot at the `/api/issues` endpoint.

At 8am every morning, the Azure Function queries GitHub for all open PRs. It categorizes them by assignees and sends them to be processed by the bot on `/api/prs`.

### Setup

Rename `.env.sample` to `.env` and fill out the following environment variables:

```cmd
GitHubToken=<Your GitHub Access Token Created in GitHub>

MicrosoftAppId=<AppId of the Bot>
MicrosoftAppPassword=<AppPassword of the Bot>
BotBaseUrl=<https://<myBot>.azurewebsites.net>

UseTestRepo=<true|false - this allows you to create a test repo and fake issues. The bot then changes the expiration time so the issue appears expired>
```

### Run

Run locally via `npm run start:local`

### Deploy

This is easiest to deploy by using the [Azure Functions VS Code Extension](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions)

Be sure to set the `WEBSITE_TIME_ZONE` to the appropriate time zone (Pacific Standard Time), as the PR notification trigger is based on UTC, otherwise.

## Bot

This is a pretty simple, primarily-notifications bot. It listens for the data sent from the Azure Function on `/api/issues` and `/api/prs`, authenticates the request, processes the data, then sends out notifications to the appropriate users based on issue or PR expiration.

Once installed to a Teams Team/Channel, it queries all members on the Team. For any team members who have not provided their GitHub info to the bot, it sends them a notification message, requesting they log in so that we can notify them.

### Setup

Rename `appsettings.json.sample` to `appsettings.json` and fill out the following environment variables:

```json
{
  "MicrosoftAppId": "",
  "MicrosoftAppPassword": "",
  "ConnectionName": "<OAuth Connection Name (for GitHub OAuth)>",
  "CosmosDbEndpoint": "",
  "CosmosDbAuthKey": "",
  "CosmosDbDatabaseId": "",
  "CosmosDbContainerId": "",
  "ApplicationInsights": {
    "InstrumentationKey": ""
  },
  "EnableTestMode": "<true|false - this allows you to create a test repo and fake issues. The bot then changes the expiration time so the issue appears expired>"
}

```

### Deploy

Deploy like a standard bot. Be sure to update the Teams App Manifest.

### Maintainer

The Bot has some additional commands that it only responds to if they come from the maintainer listed in `Bot/Constants.cs > MaintainerGitHubId`. Currently, the commands are:

* `command:enableNotifications`: Send bot error notifications to the Maintainer via Teams.
* `command:disableNotifications`: Turn off notifications of bot errors.
* `command:sendCards`: Send test adaptive cards to the maintainer (for testing purposes).
* `command:resendGreetings`: Query the channel the message comes from for all team members. For any team members who have not provided their GitHub info to the bot, it sends them a notification message, requesting they log in so that we can notify them.

## Links

* [Bot Web App](https://ms.portal.azure.com/#@microsoft.onmicrosoft.com/resource/subscriptions/0389857f-2464-451b-ac83-5f54d565b1a7/resourceGroups/v-micricMAIN/providers/Microsoft.BotService/botServices/IssueNotificationBot/overview)
* [Bot App Service](https://ms.portal.azure.com/#@microsoft.onmicrosoft.com/resource/subscriptions/0389857f-2464-451b-ac83-5f54d565b1a7/resourceGroups/v-micricMAIN/providers/Microsoft.Web/sites/issuenotificationbot/appServices)
* [Azure Function](https://ms.portal.azure.com/#@microsoft.onmicrosoft.com/resource/subscriptions/0389857f-2464-451b-ac83-5f54d565b1a7/resourceGroups/v-micricMAIN/providers/Microsoft.Web/sites/IssueNotificationIssueRetriever/appServices)
* [Cosmos Storage](https://ms.portal.azure.com/#@microsoft.onmicrosoft.com/resource/subscriptions/0389857f-2464-451b-ac83-5f54d565b1a7/resourceGroups/v-micricMAIN/providers/Microsoft.DocumentDb/databaseAccounts/vmicriccosmos/dataExplorer)