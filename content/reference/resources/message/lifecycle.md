---
title: "Message Lifecycle"
---

# Message Lifecycle

* TOC
{:toc}

## Create a Message

Create a [Message](/reference/resources/message/) in the specified Channel.

Send a JSON document that matches the [Message schema](/reference/resources/message/) with an HTTP header of `Content-Type: application/json`. Currently, the only keys we use from your JSON will be `text`, `reply_to`, `annotations`, `entities`, and `machine_only`.

If you would like to specify your own entities, please refer to the [user specified entites](/reference/meta/entities/#user-specified-entities) documentation, otherwise we will parse out links, hashtags, and mentions from the `text` field.

If you want to test how your text will be processed you can use the [text processor](/reference/resources/text-processor).

#### Creating Private Messages for use with net.app.core.pm Channels

To create private group messages for use in `net.app.core.pm` channels, you can specify the special `channel_id` of `pm`. With this parameter, the server will look for an extra field in the provided message object called `destinations` which is a list of user ids to send this message to. If a private message channel already exists between this group of users, its `channel_id` will be reused. Otherwise, a new channel will be created and the users specified in the `destinations` list will be auto-subscribed (according to their [subscription preferences](https://account.app.net/settings/privacy/#messaging)) and able to write to that channel. Note that the `destinations` value may include user ids in the form of "@username" or integer id.

<%= general_params_note_for "message" %>

<%= endpoint "POST", "channels/[channel_id]/messages", "User" %>

<%= url_params [
    ["channel_id", 'The id of the <a href="/reference/resources/channel/">Channel</a> in which to create the Message. Alternatively, you can specify <code>pm</code> to auto-create/reuse a <code>net.app.core.pm</code> <a href="/reference/resources/channel/#private-message">private message channel</a>.']
]%>

#### Generic Channel Example

<% data = {
    "text" => "Hello channel!",
} %>
<%= curl_example(:post, "channels/1/messages", :message, {:data => data}) %>

#### PM Channel Example

<% data = {
    "text" => "Hello channel!",
    "destinations" => ["@berg", 1]
} %>
<%= curl_example(:post, "channels/pm/messages", :message, {:data => data}) do |h|
    h["data"]["channel_id"] = "2"
    h["data"]["id"] = "105"
    h["data"]["thread_id"] = "105"
end %>

## Delete a Message

Delete a message. The current user must be the same user who created the Message. It returns the deleted Message on success.

*You can always delete a message you created even if you are no longer able to view the rest of the Channel anymore.*

*Remember, access tokens can not be passed in a HTTP body for `DELETE` requests. Please refer to the [authentication documentation](/reference/authentication/#making-authenticated-api-requests).*

<%= general_params_note_for "message" %>

<%= endpoint "DELETE", "channels/[channel_id]/messages/[message_id]", "User", "public_messages</code> or <code>messages" %>

<%= url_params [
    ["channel_id", "The id of the Channel this Message belongs to."],
    ["message_id", "The id of the Message to delete."]
]%>

#### Example

<%= curl_example(:delete, "channels/1/messages/1", :message) do |h|
    h["data"]["is_deleted"] = true
    h["data"].delete("text")
    h["data"].delete("html")
    h["data"]["entities"].each { |k, v| h["data"]["entities"][k] = []}
end %>


## Retrieve the Messages in a Channel

Retrieve a stream of the Messages in a channel.

<%= general_params_note_for "message" %>

<%= pagination_note %>

<%= endpoint "GET", "channels/[channel_id]/messages", "User", "public_messages</code> or <code>messages" %>

<%= url_params [
    ["channel_id", "The id of the Channel to retrieve Messages from."]
]%>

#### Example

<%= curl_example(:get, "channels/1/messages", :message, {:response => :paginated}) do |h|
    h["meta"]["marker"] = {"name" => "channel:1"}
end %>
