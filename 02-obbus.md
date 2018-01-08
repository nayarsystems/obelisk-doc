# Obbus protocol

The obbus protocol allows to interact with the obelisk pubsub system. 

It uses [msgpack](https://msgpack.org/) to subscribe and publish in the pubsub.

Currently only the TCP transport is supported by obelisk (by default listening on port 2324).

## Traffic format

All the traffic in both directions in the connection is serialized using _msgpack_ arrays.

> As *mspack* is a binary protocol, the arrays in the following document are represented in JSON format for readability purposes. But it should be noticed that the final data sent through the connection must be encoded in _msgpack_.
>
> For example, given the following JSON array corresponding to a **`ping`** *command*:
>
> `[ 0, 1 ]`
>
> It should be sent through the connection as the following bytes:
>
> `0x92 0x00 0x01`

Each array sent is a **command**.

Each array received can be:

* The **response** from a sent *command*.
* The **error** from a sent *command*.
* A **message** originiated in the pubsub in a topic previously subscribed.

> Notice that obbus is an asynchronous protocol in the sense that multiple commands can be sent before receiving its responses or errors and messages can be received at any time. 

### Commands

Commands have the following format:

    [ command_code, arguments... ]

`command_code` and the number of arguments will depend on the type of command (later in this document the codes for each command and its arguments are explained).

When a command is sent, a response or an error will be received.

### Responses

Responses have the following format:

    [ 17, command_code, result ]

`command_code` refers to the type of command that originated the response.

Later in this document the `result` values for each type of command are explained.

### Errors

Errors have the following format:

    [ 18, command_code, error_code ]

`command_code` refers to the type of command that originated the response or error.

The possible values of `error_code` are:

error_code | Description
-- | --
1 | Command does not exist (provided `command_code` not valid)
2 | Invalid number of arguments for command
3 | Invalid type of argument
4 | Invalid frame (sent data is not an array)
5 | Invalid topic index, not in topic table (more on this later)

### Messages

Messages are received when a publish in a subscribed topic is done. Depending on the type of publish, they have one of the following formats:

    [ 16, flags, topic, value ]

    [ 16, flags, topic, value, rtopic ]

## Commands explained

### set_keepalive_timeout

The connection is closed by the remote side when no commands are received after the keepalive timeout expires. When any command is received in the remote side its keepalive timeout is restarted. A dummy **`ping`** command can be sent periodically in order for the keepalive to be resetted.

The keepalive timeout is configured to `600` seconds by default, but the remote side could be configured with another value.
The client can configure the remote keepalive timeout with the **`set_keepalive_timeout`** command.

**command**: `[ 6, n ]` 

* `n` is an integer representing the number of seconds to set as the remote keepalive timeout.

**response**: `[ 17, 6, n ]` 

* `n` is the integer sent in the command.

### ping

Gets a response to test connectivity (and resets the keepalive timeout).

**command**: `[ 0, n ]` 

* `n` is an integer that will come back in the response.

**response**: `[ 17, 0, n ]` 

* `n` is the integer sent in the command.

### close

Forces the remote side to close the connection.

**command**: `[ 7 ]`

**response**: As the server closes the connection, no response is received. The client should close the connection too.

### topic_table_create

To save bandwidth an index-to-topic translation table is implemented. This table translates an integer index (e.g. `0`) to a string topic (e.g. `can.evt.mod.stat`) where the client wants to publish or from where he receives messages because he subscribed. This results in less bytes sent over the wire but its use isn't mandatory, as later is explaned, either an integer index or a string can be passed as a topic.

`topic_table_create` creates a table for the connection with the desired size (indexes start by 0). Successive calls will delete the old topic table and create it again.

**command**: `[ 8, n ]` 

* `n` is a positive integer representing the desired size for the table (the maximum size is `256`).

**response**: `[ 17, 8, 0 ]` means success.

**response**: `[ 17, 8, -1 ]` means the size passed exceeded the maximum, the table hasn't been created and the old one hasn't been deleted.

### topic_table_set

Sets (or clears) an entry of the topic table.

**command**: `[ 9, i, t ]` 

* `i` is a positive integer (or zero) representing the index to be set.
* `t` is a string with the topic (or an empty string to clear the index).

**response**: `[ 17, 9, 0 ]` means success.

**response**: `[ 17, 9, -1 ]` means the index is not in the valid range (0 to size-1).

### subscribe

Subscribe to a pubsub topic in order to receive all the messages published to it.

The topic can be full like `some_instance.interesting.topic_A` or partial like `some_instance.*`.

> #### examples:
> The partial topic `some_instance.interesting.*` will receive messages to `some_instance.interesting.topic_A`, `some_instance.interesting.subtopic.topic_Z`...
>
> The partial topic `some_instance.*` will receive messages to `some_instance.not_so_interesting.topic_A`, `some_instance.interesting.topic_B`...
>
> ¡The partial topic `*` will receive ALL messages on the pubsub!

**command**: `[ 1, topic ]`

* `topic` is a string representing the topic to subscribe to.

**response**: `[ 17, 1, 0 ]` means success.

**response**: `[ 17, 1, -1 ]` means the subscription has not been done because there was a subscription to this topic already.

### unsubscribe

Unsubscribe from a pusbub topic in order to stop receiving the messages published to it.

**command**: `[ 2, topic ]`

**response**: `[ 17, 2, 0 ]` means success.

**response**: `[ 17, 2, -1 ]` means no subscription to this topic existed.

### unsubscribe_all

Unsubscribe from all previous subscriptions. Stop receiving messages from the pubsub.

**command**: `[ 3 ]`

**response**: `[ 17, 3, n ]`

* `n` is an integer representing the number of topics that were unsubscribed.

### publish

Sends a message to a pubsub topic. Optionally giving a response topic or message flags for customizing how the message is treated.

The allowed message flags are:

Flag | Description
-- | --
Instant | When it's set, the subscriber callback functions are called in the precise moment where the pubsub processes the message, so when the publish operation is completed all the callbacks code has been executed. Otherwise the message is queued, the publish operation is completed and then the callbacks are called when the main event loop is free (usually in a few milliseconds).
Non-recursive | When it's set only the subscribers on the full topic will receive the message (when publishing to `some_instance.interesting.topic` only the subscribers to `some_instance.interesting.topic` will receive it). Otherwise the subscribers to partial topics will receive the message too (the subscribers to `some_instance.interesting.*`, `some_instance.*` and `*`).
Error | It signals the message as an error (useful for response messages to signal something went bad), the subscriber can check for this flag.

**command**: `[ 4, topic_or_index, value ]`

**command**: `[ 4, topic_or_index, value, flags ]` 

**command**: `[ 4, topic_or_index, value, flags, rtopic_or_index ]`

* `topic_or_index` is a string with the topic or a positive integer (or zero) valid index of the topic table where the publish will be done. Publish is always done with full topics, not partial ones.
* `value` is a valid value to publish on the pubsub (integer, float, string or byte array).
* `flags` *(optional)* is an integer where the message flags are set, for most cases this is not needed and a `0` value can be passed. The flags are set through this bits:
    * bit 0: Instant
    * bit 1: Non-recursive
    * bit 3: Error
* `rtopic_or_index` *(optional)* is a string with a topic or a positive integer (or zero) valid index of the topic table where the receptor/s of the publish can respond with another publish (depending on the topic the other side may respond or not). Response topics are always full topics too, not partial ones.

**response**: As the `publish` command can be intensively used, as an exception it does not receive a response to save bandwith. Another command (`publish_ack`) is provided to force a response.

**error**: `[ 18, 4, 5 ]` a provided topic index (`topic_or_index` or `rtopic_or_index`) was not found in the topic table.

### publish_ack

Does the same as `publish` but receives a response acknowledging that the publish has been completed.

> The use of `publish_ack` in combination with the *Instant* flag is useful as before the response for the command (the ack) has arrived, it's assured that all the subscribers have received the message and treated it.

**command**: `[ 5, topic_or_index, value ]`

**command**: `[ 5, topic_or_index, value, flags ]` 

**command**: `[ 5, topic_or_index, value, flags, rtopic_or_index ]`

**response**: `[ 17, 5, n ]`

* `n` is `0` if the flag *Instant* was not set on the message, otherwise it's the number of subscribers that received the message.

**error**: `[ 18, 5, 5 ]` a provided topic index (`topic_or_index` or `rtopic_or_index`) was not found in the topic table.

## Messages explained

When subscriptions to topics are done and messages are published to hhose topics, messages are received in the following format:

`[ 16, flags, topic_or_index, value ]`

`[ 16, flags, topic_or_index, value, rtopic_or_index ]`

* `flags` is an integer describing the flags that were used on the publish. The flags are read through this bits:
    * bit 0: Instant
    * bit 1: Non-recursive
    * bit 3: Error
* `topic_or_index` is a string with the topic or a positive integer (or zero) valid index of the topic table where the publish was done.
* `value` is the value published on the topic (integer, float, string or byte array).
* `rtopic_or_index` *(optional)* is a string with a topic or a positive integer (or zero) valid index of the topic table where the publisher expects responses to this message.

## Implementing RPC

Some modules offer an external API through the pubsub, they are subscribed to a topic for each method they want to offer (`some_instance.method_A`, `some_instance.method_B`). When someone publishes to those topics and they receive the message, they execute the request and respond with the results to the response topic. This allows an RPC-like pattern.

> As it depends on how the subscriber behaves, this pattern only happens in some topics. Some topics may not respond to messages while others could respond multiple times. Always check the documentation on the used topic.

Here is a suggestion on how to implement RPC calls on the client side:

* Subcribe to a partial topic dedicated only to RPC for the connection, for example `rpc123456789.*` (the number should be randomly generated to avoid collisions with others).

* For each RPC call do a publish to the desired topic providing an auto-incrementing response topic (`rpc123456789.1`, `rpc123456789.2`, `rpc123456789.3`...).

* For each RPC call wait for the response message to arrive through the subscription and match to the call with the auto-incrementing number.

* Optionally timeouts may be implemented because a response message could never arrive or/and take special care of extra response messages that may arrive...

## Client in Go (gobbus)

A client in Go is implemented: [gobbus](https://github.com/nayarsystems/gobbus).

## Simple Lua example

Example of a publish command of the value `1234` to the topic `test.lua`

```
local mp = require("MessagePack")
local socket = require("socket")
local host, port = "127.0.0.1", 2324
local tcp = assert(socket.tcp())

tcp:connect(host, port);
tcp:send(mp.pack { 4, "test.lua", 1234 } ) 
tcp:close()
```
