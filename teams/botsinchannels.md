# Interact in a team channel with a Microsoft Teams bot

Microsoft Teams allows users to bring bots into their channel conversations.  By adding a bot as a team member, all users of the team can take advantage of the bot functionality right in the channel conversation.  You can also access Teams-specific functionality within your bot like querying team information and @mentioning users.

To add a bot to a team, you'll need to follow the [packaging](createpackage.md) and [sideloading](sideload.md) instructions.

## Designing a great bot for channels

Bots added to a team become another team member, who can be @mentioned as part of the conversation. In fact, bots only receive messages when they are @mentioned, so other conversations on the channel are not sent to the bot. 

>**Note:** For convenience when replying to bot messages in a channel, the bot name is prepended automatically.

A bot in a channel should provide information relevant and appropriate for all members of the team.  While your bot can certainly provide any information relevant to the experience, keep in mind conversations with it are visible to all members of the channel.  Therefore, a great bot in a channel should add value to all users on the channel, and certainly not inadvertantly share information that would otherwise be more relevant in a personal context. 

Note that depending on your experience, the bot might be entirely relevant in both scopes (personal and team) as is, and in fact, no significant extra work is required to allow your bot to work in both. In Microsoft Teams, there is no expectation that your bot function in all contexts, but you should ensure your bot makes sense, and provides user value, in whichever scope you choose to support. For more information on scopes, see [Apps in Microsoft Teams](teamsapps.md).

## Building your bot

Developing a bot that works in channels uses much of the same functionality from 1:1 conversation. Additional events and data in the payload provide Teams channel information. Those differences, as well as key differences in common functionality are enumerated in the following sections.

### Receiving messages

For a bot in a channel, in addition to the [regular message schema](https://docs.botframework.com/en-us/core-concepts/reference/#activity), your bot also receives the following properties:

* `channelData`&emsp;See [Teams channel data](botsconversation.md#teams-channel-data)
* `conversationData.id`&emsp;The reply chain ID, consisting of channel ID plus the ID of the first message in the reply chain
* `conversationData.isGroup`&emsp;Set to `true` for bot messages in channels
* `entities`&emsp;Can contain one or more mentions (see [Mentions](#mentions))

### Replying to messages

To reply to an existing message, call [`ReplyToActivity`](https://docs.microsoft.com/en-us/bot-framework/dotnet/bot-builder-dotnet-connector#send-a-reply) in .NET or [`session.send`](https://docs.microsoft.com/en-us/bot-framework/nodejs/bot-builder-nodejs-use-default-message-handler) in Node.js. The Bot Builder SDK handles all the details.

If you choose to use the REST API, you can also call the [`/conversations/{conversationId}/activities/{activityId}`](https://docs.microsoft.com/en-us/bot-framework/rest-api/bot-framework-rest-connector-send-and-receive-messages#send-the-reply) endpoint.

Note that replying to a message in a channel shows as a reply to the initiating reply chain. The `conversationId` contains the channel and the top level message ID. Although the Bot Framework takes care of the details, you can cache that `conversationId` for future replies to that conversation thread as needed.

### Creating a channel conversation

Your team-added bot can post into a channel to create a reply chain. With the Bot Builder SDK, call [`CreateConversation`](https://docs.microsoft.com/en-us/bot-framework/dotnet/bot-builder-dotnet-connector#start-a-conversation) for .NET or use [proactive messages](https://docs.microsoft.com/en-us/bot-framework/nodejs/bot-builder-nodejs-proactive-messages) (`bot.send` and `bot.beginDialog`) in Node.js.

Alternatively, you can issue a POST request to the [`/conversations/{conversationId}/activities/`](https://docs.microsoft.com/en-us/bot-framework/rest-api/bot-framework-rest-connector-send-and-receive-messages#send-message) resource.

**Note:** At this point, bots in Microsoft Teams cannot initiate one-to-many or group conversations.

#### .NET example

```csharp
var channelData = new TeamsChannelData { Channel = new ChannelInfo(yourChannelId) };
IMessageActivity newMessage = Activity.CreateMessageActivity();
newMessage.Type = ActivityTypes.Message;
newMessage.Text = "Hello, on a new thread";
ConversationParameters conversationParams = new ConversationParameters(
    isGroup: true,
    bot: null,
    members: null,
    topicName: "Test Conversation",
    activity: (Activity)newMessage,
    channelData: channelData);
var result = await connector.Conversations.CreateConversationAsync(conversationParams);
```

### Best practice: Welcome messages in teams

When your bot is first added to the team, it is a best practice to send a welcome message to the team to introduce the bot to all users of the team and tell a bit about its functionality. To do this, ensure that your bot responds to the `conversationUpdate` message, with the `teamsAddMembers` eventType in the `channelData` object. Be sure that the `memberAdded` ID is the bot ID itself, because the same event is sent when a user is added to a team. See [Team member or bot addition](botevents.md#team-member-or-bot-addition) for more details.

You might also want to send a 1:1 message to each member of the team when the bot is added. To do this, you could [fetch the team roster](botapis.md#fetching-the-team-roster) and send each user a [direct message](bots1on1.md#starting-a-11-conversation).

We recommend that your bot *not* send a welcome message in the following situations:

* Your bot is added to a channel (as compared to being added to a team)
* A channel is renamed
* A team member is added to a channel

>For more best practices, see our [design guidelines](design.md).

## Mentions

Bots in a channel respond only when they are mentioned ("@_botname_") in a message. That means every message received by a bot in a channel contains its own name, and you must ensure that your message parsing handles that case. In addition, bots can parse out other users mentioned and mention users as part of their messages.

### Retrieving mentions

Mentions are returned in the `entities` object in payload and contain both the unique ID of the user and, in most cases, the name of user mentioned. You can retrieve all mentions in the message by calling the `GetMentions` function in the Bot Builder SDK for .NET, which returns an array of `Mentioned` objects.

All messages that your bot receives have the name of the bot in the channel and should accomodate that in its message parsing.

#### .NET example code: Check for and strip @bot mention

```csharp
Mention[] m = sourceMessage.GetMentions();
var messageText = sourceMessage.Text;

for (int i = 0;i < m.Length;i++)
{
    if (m[i].Mentioned.Id == sourceMessage.Recipient.Id)
    {
        //Bot is in the @mention list.  
        //The below example will strip the bot name out of the message, so you can parse it as if it wasn't included.  Note that the Text object will contain the full bot name, if applicable.
        if (m[i].Text != null)
            messageText = messageText.Replace(m[i].Text, "");
    }
}
```

>**Note:** You can also use the Teams extension function `GetTextWithoutMentions`, which strips out all mentions, including the bot.

#### Node.js example code: Check for and strip @bot mention

```javascript
var text = message.text;
if (message.entities) {
    message.entities
        .filter(entity => (entity.type === "mention" && entity.id === yourbotid))
        .forEach(entity => {
            text = text.replace(entity.text, "");
        });
    text = text.trim();
}
```
>**Note:** You can also use the Teams extension function `getTextWithoutMentions`, which strips out all mentions, including the bot.

### Constructing mentions

Your bot can mention other users in messages posted into channels. To do this, your message must do the following:

* Include `<at>@username</at>` in the message text
* Include the `mention` object inside the entities collection

The [Teams extensions for the Bot Builder SDK](https://msdn.microsoft.com/en-us/microsoft-teams/code#microsoft-teams-extensions-for-the-bot-builder-sdk) provide functionality to easily accomodate this.

>**Note**: At this time, team and channel mentions are not supported.

#### .NET example

This example uses the [Microsoft.Bot.Connector.Teams](https://www.nuget.org/packages/Microsoft.Bot.Connector.Teams) NuGet package.

```csharp
// Create reply activity
Activity replyActivity = activity.CreateReply();

// Construct text of the form @sender Hello
replyActivity.Text = "Hello ";
replyActivity.AddMentionToText(activity.From, MentionTextLocation.AppendText);

// Send the reply activity
await client.Conversations.ReplyToActivityAsync(replyActivity);
```

#### Node.js example

This sample uses the [botbuilder-teams](https://www.npmjs.com/package/botbuilder-teams) npm package.

```javascript
// User to mention
var toMention: builder.IIdentity = {
    name: 'John Doe',
    id: userId
};

// Create a message and add mention to it
var msg = new teams.TeamsMessage(session).text(teams.TeamsMessage.getTenantId(session.message));
var mentionedMsg = msg.addMentionToText(toMention);

// Post the message
var generalMessage = mentionedMsg.routeReplyToGeneralChannel();
session.send(generalMessage);
```

#### Schema example: Outgoing message with user mentioned

```json
{ 
    "type": "message", 
    "text": "Hey <at>Larry Jin</at> check out this message", 
    "timestamp": "2016-10-29T00:51:05.9908157Z", 
    "localTimestamp": "2016-10-28T17:51:05.9908157-07:00", 
    "serviceUrl": "https://skype.botframework.com", 
    "channelId": "msteams", 
    "from": { 
        "id": "28:9e52142b-5e5e-4d7b-bb3e- e82dcf620000", 
        "name": "SchemaTestBot" 
    }, 
    "conversation": { 
        "id": "19:aebd0ad4d6ab42c8b9ed19c251c2fc37@thread.skype;messageid=1481567603816" 
    }, 
    "recipient": { 
        "id": "8:orgid:6aebbad0-e5a5-424a-834a-20fb051f3c1a", 
        "name": "stlrgload100" 
    }, 
    "attachments": [ 
        { 
            "contentType": "image/png", 
            "contentUrl": "https://upload.wikimedia.org/wikipedia/en/a/a6/Bender_Rodriguez.png", 
            "name": "Bender_Rodriguez.png" 
        } 
    ], 
    "entities": [ 
        { 
            "type":"mention", 
            "mentioned":{ 
                "id":"29:08q2j2o3jc09au90eucae",
                "name":"Larry Jin" 
            }, 
            "text": "<at>@Larry Jin</at>"
        } 
    ], 
    "replyToId": "3UP4UTkzUk1zzeyW" 
}
```

## Accessing team context

Your bot can do more than send and receive messages in teams. For instance, it can also fetch the list of team members, including their profile information, as well as the list of channels. See [Get context for your Microsoft Teams bot](botapis.md) to learn more.
