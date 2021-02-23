# mpp-protocol
Legacy protocol of Multiplayer Piano  
The types used here are [TypeScript types](https://www.typescriptlang.org/docs/handbook/basic-types.html).  

## Custom types:
### note
| Parameter |  Type   |       Description        |
|-----------|---------|--------------------------|
|     n     | string  |        Note name         |
|     v     | number  |         Volume           |
|     d     | number  |          Delay           |
|     s     | boolean | Whether to stop the note |

### chset
| Parameter |  Type   |         Description          |
|-----------|---------|------------------------------|
|  visible  | boolean |     Is the room visible?     |
|   chat    | boolean |   Does the room have chat?   |
| crownsolo | boolean |   Only the owner can play?   |
|   lobby   | boolean | Whether the room is a lobby. |
|   color   | string  |     Inner room color.        |
|  color2   | string  |     Outer room color.        |

Note: The `lobby` setting is disregarded by the server, meaning you can't create lobbies with custom names. Only using test/.

### userset
| Parameter |  Type  |     Description     |
|-----------|--------|---------------------|
|   name    | string |      The name.      |
|   color   | string | The color (#rrggbb) |

### uinfo
| Parameter |  Type  |      Description       |
|-----------|--------|------------------------|
|    \_id    | string |      User's \_id        |
|   name    | string |      User's name       |
|   color   | string | User's color (#rrggbb) |

### cuinfo
| Parameter |  Type  |      Description       |
|-----------|--------|------------------------|
|    \_id    | string |      User's \_id        |
|   name    | string |      User's name       |
|   color   | string | User's color (#rrggbb) |
|    id     | string |       User's id        |

### vec2
| Parameter |  Type  | Description |
|-----------|--------|-------------|
|     x     | number | X position. |
|     y     | number | Y position. |

### crown
|   Parameter   |  Type  |               Description                 |
|---------------|--------|-------------------------------------------|
|    userId     | string |    userId of the latest crown holder.     |
| participantId | string | participantId of the latest crown holder. |
|     time      | number |      Time of the last crown update.       |
|   startPos    |  [vec2](#vec2)  |        Crown drop start position.         |
|    endPos     |  [vec2](#vec2)  |         Crown drop end position.          |

Note: participantId *must* be undefined if there is no current crown holder.

### channel
| Parameter |  Type  |          Description          |
|-----------|--------|-------------------------------|
|   count   | number | Number of people in the room. |
|   crown   | [crown](#crown)  |     Channel crown object.     |
| settings  | [chset](#chset)  |       Channel Settings        |
|    \_id    | string |           Room name           |


### cinfo
| Parameter |   Type   |              Description              |
|-----------|----------|---------------------------------------|
|    ch     | [channel](#channel)  |             Channel info              |
|    ppl    | [cuinfo](#cuinfo)[] |         People in the channel         |
|     p     |  string  | Only used when a user joins a channel |

### nq
| Parameter  |  Type  |                           Description                           |
|------------|--------|-----------------------------------------------------------------|
| allowance  | number | How many points to give back to the user when tick() is called. |
|    max     | number |                Maximum amount of points allowed.                |
| maxHistLen | number |   Has something to do with aggressive limitation. Usually 3.    |

Notes: userId = \_id, participantId = id

## Serverbound
### hi
Initializes the client on the server.  
(no parameters)

### bye
Destroys your client on the server.
(no parameters)

### +ls
Tells the server you want to listen for rooms. The server first will send back a [ls](#ls) message containing the full room list. After that, every time a room updates, it will send ls with that room's info.
(no parameters)

### -ls
Tells the server you don't want to listen for rooms anymore.

### t
Used to sync time between the client and the server. Must be sent in under 40~ seconds after the last ping, or after connecting.
| Parameter |  Type  |         Description          |
|-----------|--------|------------------------------|
|     e     | number | Client time in milliseconds. |

### a
Sends a chat message to the server.
| Parameter |  Type  |   Description    |
|-----------|--------|------------------|
|  message  | string | Message content. |

### chown
Sends a **ch**ange **own**ership request to the server.
| Parameter |  Type  |                      Description                      |
|-----------|--------|-------------------------------------------------------|
|    id     | string | Used when transferring the crown. Drops it otherwise. |

### n
Sends the client's note buffer to the server.
| Parameter |  Type  |             Description              |
|-----------|--------|--------------------------------------|
|     t     | number | noteBufferTime plus serverTimeOffset |
|     n     | [note](#note)[] |             Note buffer              |

### m
Sends your mouse position. People can see your entire cursor **between** 0 and 101.
| Parameter |  Type  | Description |
|-----------|--------|-------------|
|     x     | number |   Mouse X   |
|     y     | number |   Mouse Y   |

### chset
Changes the channel settings if you're the room owner.
| Parameter | Type  |    Description    |
|-----------|-------|-------------------|
|    set    | [chset](#chset) | Channel settings. |

Note: `set` can include all channel settings, or only the one(s) you want to set.  
Example: `{ visible: false }` is valid, and will make the room invisible without changing the other channel settings.

### userset
Changes your user info. Changes your color on some servers.
| Parameter |  Type   | Description |
|-----------|---------|-------------|
|    set    | [userset](#userset) | User info.  |

### kickban
Kicks a user from the room. Not supported on nagalun's server.
| Parameter |  Type  |               Description                 |
|-----------|--------|-------------------------------------------|
|    \_id    | string |   userId of the user you want to kick.    |
|    ms     | number |  How long to kick them in milliseconds.   |

### ch
Changes the channel you're in.
| Parameter |  Type  |  Description  |
|-----------|--------|---------------|
|    \_id    | string |   Room name   |
|    set    | [chset](#chset)  | Room settings |


## Clientbound
### hi
Sent to a newly connected client once. Initializes the client and gives them their user info.
| Parameter |  Type  |      Description      |
|-----------|--------|-----------------------|
|     u     | [uinfo](#uinfo)  |       User info       |
|     t     | number |      Server time      |
|     v     | string | Server version string |
|   motd    | string |      Server MOTD      |

Note: The last two are optional and are not used by the vanilla client in any way.

### bye
Sent to an entire room, except the person who left. Tells clients that a participant left.
| Parameter |  Type  |             Description             |
|-----------|--------|-------------------------------------|
|     p     | string | participantId of the user who left. |

### ls
Sent to all room update listeners. Returns all existing data when sent initially, after that only gives an updated room's info.
| Parameter |   Type    |                              Description                              |
|-----------|-----------|-----------------------------------------------------------------------|
|     c     |  boolean  | `true` when sending all room data, `false` when sending a room update |
|     u     | [channel](#channel)[] |                             Channel infos                             |

### t
Sent to a client when they send "t"
| Parameter |  Type  |  Description  |
|-----------|--------|---------------|
|     t     | number |  Server time  |
|     e     | number | Client's time |

### n
Sent to an entire room, except the sender. Contains the note buffer and noteBufferTime of the person who sent the message.
| Parameter |  Type  |                     Description                     |
|-----------|--------|-----------------------------------------------------|
|     n     | [note](#note)[] |                     Note buffer                     |
|     t     | number |                   noteBufferTime                    |
|     p     | string | participantId of the user who sent the note buffer. |

### a
Sent to an entire room. A chat message.
| Parameter |  Type  | Description |
|-----------|--------|-------------|
|     a     | string | The message |
|     p     | [cuinfo](#cuinfo) | Sender info |
|     t     | number |  Timestamp  |

### m
Sent to an enire room, except the sender. Mouse movement.
| Parameter |  Type  | Description |
|-----------|--------|-------------|
|     x     |  [vec2](#vec2)  |   Mouse X   |
|     y     |  [vec2](#vec2)  |   Mouse Y   |

### p
Sent to an entire room. User info updates.
| Parameter |  Type  |  Description  |
|-----------|--------|---------------|
|   color   | string |  User color   |
|   name    | string |   User name   |
|    id     | string | participantId |
|    \_id    | string |    userId     |
|     x     | number |    Mouse X    |
|     y     | number |    Mouse Y    |

### ch
Sent to a user, or back to the sender after they've sent "ch". Changes or updates a user's channel.
| Parameter |   Type   |                           Description                            |
|-----------|----------|------------------------------------------------------------------|
|    ch     | [channel](#channel)  |                          Channel info                            |
|    ppl    | [cuinfo](#cuinfo)[] |                      People in the channel                       |
|     p     |  string  | Only present when switching a channel. User's new participantId. |

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
| duration  | number | How long the notification will last, in ms |
|   class   | string |           Type of window to show           |

Note: The "classic" class shows the title, while the "small" class does not.
Note: Set the duration to -1 to make the notification stay until the user closes it.
