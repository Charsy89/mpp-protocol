# mpp-protocol
Legacy protocol of Multiplayer Piano

## Custom types
### note
| Parameter |  Type   |       Description        |
|-----------|---------|--------------------------|
|     n     | string  |        Note name         |
|     v     |  float  |       Note volume        |
|     d     |  int64  |     Note delay in ms     |
|     s     | boolean | Whether to stop the note |

### chset
| Parameter |  Type   |         Description          |
|-----------|---------|------------------------------|
|  visible  | boolean |     Is the room public?      |
|   chat    | boolean |   Does the room have chat?   |
| crownsolo | boolean |   Only the owner can play?   |
|   lobby   | boolean | Whether the room is a lobby. |
|   color   | string  |  Ellipse inner color (hex).  |
|  color2   | string  |  Ellipse outer color (hex).  |

Note: The `lobby` setting is ignored by the server, meaning you can't create lobbies with custom names. Only using test/.

### userset
| Parameter |  Type  |     Description      |
|-----------|--------|----------------------|
|   name    | string |      User name       |
|   color   | string | User color (#rrggbb) |

Note: The `color` setting is usually ignored by the server, meaning you can't change your color.

### uinfo
| Parameter |  Type  |      Description       |
|-----------|--------|------------------------|
|    \_id    | string |      User's \_id        |
|   name    | string |      User's name       |
|   color   | string | User's color (#rrggbb) |
|    id     | string | User's participant id  |

Note: `id` will only exist when the user is assigned to a room. It changes every time they move to a different room.

### vec2
| Parameter | Type  | Description |
|-----------|-------|-------------|
|     x     | float | X position. |
|     y     | float | Y position. |

### crown
|   Parameter   |  Type  |             Description             |
|---------------|--------|-------------------------------------|
|    userId     | string |  userId of the last crown holder.   |
| participantId | string | participantId of the crown holder.  |
|     time      | int64  | Timestamp of the last crown update. |
|   startPos    |  [vec2](#vec2)  |     Crown drop start position.      |
|    endPos     |  [vec2](#vec2)  |      Crown drop end position.       |

Note: participantId *must* be undefined if there is no current crown holder.

### channel
| Parameter |  Type  |          Description          |
|-----------|--------|-------------------------------|
|   count   | int32  | Number of people in the room. |
|   crown   | [crown](#crown)  |         Channel Crown         |
| settings  | [chset](#chset)  |       Channel Settings        |
|    \_id    | string |           Room name           |


### cinfo
| Parameter |   Type   |                Description                 |
|-----------|----------|--------------------------------------------|
|    ch     | channel  |                Channel info                |
|    ppl    | uinfo[]  |           People in the channel            |
|     p     |  string  | Only present when a user joins the channel |

### nq
| Parameter  | Type  |                             Description                             |
|------------|-------|---------------------------------------------------------------------|
| allowance  | int32 |                Points to return to the user per tick                |
|    max     | int32 |                      Maximum amount of points                       |
| maxHistLen | int32 | Has something to do with aggressive limitation. Value is usually 3. |

Notes: userId = \_id, participantId = id

## Serverbound
### hi
Create a client for you on the server.  
###### (no parameters)

### bye
Destroy your client on the server. This will disconnect you.  
###### (no parameters)

### +ls
Tell the server you want to listen for room updates. The server first will send back a [`ls`](#ls) message containing the full room list and each one's info. After that, every time a room updates, it will send `ls` with that room's info.  
###### (no parameters)

### -ls
Tell the server to stop sending you room updates.  
###### (no parameters)

### t
Used to sync time between the client and the server. Must be sent every ~40 seconds.
| Parameter | Type  |         Description          |
|-----------|-------|------------------------------|
|     e     | int64 | Client time in milliseconds. |

### a
Send a chat message to the server.
| Parameter |  Type  |   Description    |
|-----------|--------|------------------|
|  message  | string | Message content. |

### chown
Send a "**ch**ange **own**ership" request to the server.
| Parameter |  Type  |                      Description                      |
|-----------|--------|-------------------------------------------------------|
|    id     | string | Used when transferring the crown. Drops it otherwise. |

### unban
Unban a person from the room on some servers.
| Parameter |  Type  |   Description   |
|-----------|--------|-----------------|
|    \_id    | string | userId to unban |

### n
Send a note buffer to the server.
| Parameter |  Type  |             Description              |
|-----------|--------|--------------------------------------|
|     t     | int64  | noteBufferTime plus serverTimeOffset |
|     n     | [note](#note)[] |             Note buffer              |

### m
Send your mouse position. People can see your entire cursor **between** -1 and 101.
| Parameter | Type  | Description |
|-----------|-------|-------------|
|     x     | float |   Mouse X   |
|     y     | float |   Mouse Y   |

### chset
Change the channel settings if you're the room owner.
| Parameter | Type  |    Description    |
|-----------|-------|-------------------|
|    set    | [chset](#chset) | Channel settings. |

Note: `set` can include all channel settings, or only the one(s) you want to set.  
Example: `{ visible: false }` is valid, and will make the room invisible without changing the other channel settings.

### userset
Change your user info.
| Parameter |  Type   | Description |
|-----------|---------|-------------|
|    set    | [userset](#userset) | User info.  |

### kickban
Kick a user from the room. Not supported on nagalun's server as of writing.
| Parameter |  Type  |               Description                 |
|-----------|--------|-------------------------------------------|
|    \_id    | string |   userId of the user you want to kick.    |
|    ms     | number |  How long to kick them in milliseconds.   |

### ch
Go to a different room.
| Parameter |  Type  |  Description  |
|-----------|--------|---------------|
|    \_id    | string |   Room name   |
|    set    | [chset](#chset)  | Room settings |

Note: If the room already exists and you're the owner, or if it's a new room, `set` will change the settings of that room.

## Clientbound
### hi
Sent to a newly connected client once. Notifies the client we're finished server-side and gives them their user info.
| Parameter |  Type  |      Description      |
|-----------|--------|-----------------------|
|     u     | [uinfo](#uinfo)  |       User info       |
|     t     | int64  |      Server time      |
|     v     | string | Server version string |
|   motd    | string |      Server MOTD      |

Note: The last two are optional and are not used by the vanilla client in any way.

### bye
Sent to an entire room, except the person who left. Tells clients that a participant left.
| Parameter |  Type  |             Description             |
|-----------|--------|-------------------------------------|
|     p     | string | participantId of the user who left. |

### ls
Sent to clients listening for room updates. Includes all rooms' data (except private ones) when sent initially. After, it only includes an updated room's data.
| Parameter |   Type    |                              Description                              |
|-----------|-----------|-----------------------------------------------------------------------|
|     c     |  boolean  | `true` when sending all room data, `false` when sending a room update |
|     u     | [channel](#channel)[] |                             Channel infos                             |

### t
Sent to a client when they send [`t`](#t)
| Parameter | Type  |  Description  |
|-----------|-------|---------------|
|     t     | int64 |  Server time  |
|     e     | int64 | Client's time |

### n
Sent to an entire room, except the sender. Contains note data and noteBufferTime of the person who sent the message.
| Parameter |  Type  |                     Description                     |
|-----------|--------|-----------------------------------------------------|
|     n     | [note](#note)[] |                     Note buffer                     |
|     t     | int64  |                   noteBufferTime                    |
|     p     | string | participantId of the user who sent the note buffer. |

### a
Sent to an entire room. It's a chat message.
| Parameter |  Type  | Description |
|-----------|--------|-------------|
|     a     | string | The message |
|     p     | [cuinfo](#cuinfo) | Sender info |
|     t     | int64  |  Timestamp  |

Note: `t` is optional, as it is not used by the vanilla client.

### m
Sent to an enire room, except the sender. Mouse movement.
| Parameter | Type  | Description |
|-----------|-------|-------------|
|     x     | float |   Mouse X   |
|     y     | float |   Mouse Y   |

### p
Sent to an entire room. User info updates.
| Parameter |  Type  |  Description  |
|-----------|--------|---------------|
|   color   | string |  User color   |
|   name    | string |   User name   |
|    id     | string | participantId |
|    \_id    | string |    userId     |
|     x     | float  |    Mouse X    |
|     y     | float  |    Mouse Y    |

### ch
Changes a user's room. Used for room settings updates (sent to entire room), or changing a single user's room (sent to single user).
| Parameter |  Type   |                        Description                        |
|-----------|---------|-----------------------------------------------------------|
|    ch     | [channel](#channel) |                       Channel info                        |
|    ppl    | [uinfo](#uinfo)[] |                   People in the channel                   |
|     p     | string  | Only present when moving rooms. User's new participantId. |

### c
Sent to a user right after they join a channel. Contains chat history for that room.
| Parameter |   Type   | Description  |
|-----------|----------|--------------|
|     c     | string[] | Chat history |

### nq
Sent to a user when their NoteQuota params change. Contains new NoteQuota params.
| Parameter | Type | Description |
|-----------|------|-------------|
|  params   |  [nq](#nq)  | New params  |

### notification
Send a custom notification to a user (or everyone if you'd like, just send the message to all connected users)
| Parameter |  Type  |                Description                 |
|-----------|--------|--------------------------------------------|
|    id     | string |       ID of the notification in HTML       |
|   title   | string |         Title of the notification          |
|   text    | string |      Text to show in the notification      |
|   html    | string |     HTML to stick in the notification      |
|  target   | string |    Where the notification should appear    |
| duration  | int64  | How long the notification will last, in ms |
|   class   | string |           Type of window to show           |

Note: The "classic" class shows the title, while the "small" class does not.
Note: Set the duration to -1 to make the notification stay until the user closes it.
Note: Any of these are optional, the client has default values.
