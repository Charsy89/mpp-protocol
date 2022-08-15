# mpp-protocol
Legacy protocol of Multiplayer Piano

## Notes on IDs
A `userId` is generated using your IP Address, and sticks with you across the site as long as your IP doesn't change. It is also referred to by `_id`.  
A `participantId` changes every time a user joins a new room, unless they are already in that room, in which case they receive the `participantId` belonging to the connection already in that room. It is also referred to by `id`.

## Custom types
These are type definitions for nested objects.
### note
| Parameter |  Type   |            Description            |
|-----------|---------|-----------------------------------|
| n         | string  | Which note to press, e.g. `fs4`.  |
| v         | float   | Note velocity (volume)            |
| d         | int     | MS before the note will be played |
| s         | boolean | If true, releases a held key      |

### chset
| Parameter |  Type   |         Description         |
|-----------|---------|-----------------------------|
| visible   | boolean | Is the room public?         |
| chat      | boolean | Does the room have chat?    |
| crownsolo | boolean | Only the owner can play?    |
| lobby     | boolean | Whether the room is a lobby |
| color     | string  | Ellipse inner color (hex)   |
| color2    | string  | Ellipse outer color (hex)   |

Note: Clients used to be able to turn rooms into lobbies by sending [chset](#chset-1) with `lobby` as `true`. It was fixed as it became more widespread.  
Note: `lobby` is not present if the room is not a lobby.

### userset
| Parameter |  Type  |   Description    |
|-----------|--------|------------------|
| name      | string | User name        |
| color     | string | User color (hex) |

Note: The `color` setting typically is ignored by the server, meaning a user cannot change his color.

### uinfo
| Parameter |  Type  |    Description     |
|-----------|--------|--------------------|
| \_id      | string | User userId        |
| name      | string | User name          |
| color     | string | User color (hex)   |
| id        | string | User participantId |

Note: `id` should NOT be present until the user joins a channel.  
Note: For types "uinfo+", these will also include participant's mouse x and y positions.

### vec2
| Parameter | Type  | Description |
|-----------|-------|-------------|
| x         | float | X position  |
| y         | float | Y position  |

### crown
|   Parameter   |     Type      |                Description                |
|---------------|---------------|-------------------------------------------|
| userId        | string        | userId of the last crown holder           |
| participantId | string        | participantId of the current crown holder |
| time          | int           | Timestamp of the last crown update        |
| startPos      | [vec2](#vec2) | Crown drop start position                 |
| endPos        | [vec2](#vec2) | Crown drop end position                   |

Note: If no-one holds the crown (it's dropped), `participantId` is not present.

### channel
| Parameter |      Type       |         Description          |
|-----------|-----------------|------------------------------|
| count     | int             | Number of people in the room |
| crown     | [crown](#crown) | Channel Crown                |
| settings  | [chset](#chset) | Channel Settings             |
| \_id      | string          | Room name                    |

Note: If the room is a lobby, `crown` is not present.

### nq
| Parameter  | Type  |                             Description                             |
|------------|-------|---------------------------------------------------------------------|
| allowance  | int   | Points to return to the user per tick                               |
| max        | int   | Maximum amount of points                                            |
| maxHistLen | int   | Has something to do with aggressive limitation. Value is usually 3. |

## Client -> Server
### hi
Asks the server for your user info.
###### (no parameters)

### bye
Tells the server to disconnect you.
###### (no parameters)

### +ls
Tells the server you want to listen for room updates. The server will send [`ls`](#ls) first, containing all visible rooms and their info. Subsequent `ls` messages will contain room updates.
###### (no parameters)

### -ls
Tell the server to stop sending you room updates.
###### (no parameters)

### t
Used to sync time between the client and the server. Must be sent every ~40 seconds.
| Parameter | Type  |         Description         |
|-----------|-------|-----------------------------|
| e         | int   | Client time in milliseconds |

### a
Send a chat message to the server.
| Parameter |  Type  |   Description   |
|-----------|--------|-----------------|
| message   | string | Message content |

### n
Send notes to the server.
| Parameter |      Type       |             Description              |
|-----------|-----------------|--------------------------------------|
| t         | int             | noteBufferTime plus serverTimeOffset |
| n         | [note](#note)[] | Note buffer                          |

### m
Send your mouse position.
| Parameter | Type  | Description |
|-----------|-------|-------------|
| x         | float | Mouse X     |
| y         | float | Mouse Y     |

### userset
Change your user info.
| Parameter |        Type         | Description |
|-----------|---------------------|-------------|
| set       | [userset](#userset) | User info   |

### kickban
Ban a user from the room for an amount of time.
| Parameter |  Type  |                 Description                 |
|-----------|--------|---------------------------------------------|
| \_id      | string | userId of the user you want to ban          |
| ms        | number | How long the ban will last, in milliseconds |

### unban
Unban a person from the room.
| Parameter |  Type  |   Description   |
|-----------|--------|-----------------|
| \_id      | string | userId to unban |

### ch
Go to a different room.
| Parameter |      Type       |  Description  |
|-----------|-----------------|---------------|
| \_id      | string          | Room name     |
| set       | [chset](#chset) | Room settings |

Note: If the room already exists and you're the owner, or if it's a new room, `set` will change the settings of that room.

### chset
Change the channel settings.
| Parameter |      Type       |   Description    |
|-----------|-----------------|------------------|
| set       | [chset](#chset) | Channel settings |

Note: `set` can include all channel settings, or only the one(s) you want to set. For example, `{ visible: false }` is valid, and will make the room invisible without changing the other channel settings.

### chown
"**ch**ange **own**ership". Interacts with the crown.
| Parameter |  Type  |                       Description                       |
|-----------|--------|---------------------------------------------------------|
| id        | string | participantId of the user you want to give the crown to |

Note: `id` is optional. If it's invalid or not present, the crown is dropped. If the crown is already dropped, you pick up the crown regardless of `id`.

## Server -> Client
### hi
Sent to a newly connected client once. Gives the client their user info, along with info about the server.
| Parameter |      Type       |      Description      |
|-----------|-----------------|-----------------------|
| u         | [uinfo](#uinfo) | User info             |
| t         | int             | Server time           |
| v         | string          | Server version string |
| motd      | string          | Server MOTD           |

Note: The last two are optional and are not used by the client in any way. NMPB does use them, however.

### bye
Sent to an entire room, except the person who left. Tells other participants that this person left.
| Parameter |  Type  |            Description             |
|-----------|--------|------------------------------------|
| p         | string | participantId of the user who left |

### ls
Sent to clients listening for room updates. See [`+ls`](#ls) for more info.
| Parameter |         Type          |                              Description                              |
|-----------|-----------------------|-----------------------------------------------------------------------|
| c         | boolean               | `true` when sending all room data, `false` when sending a room update |
| u         | [channel](#channel)[] |  Channel infos                                                        |

### t
Sent to a client when they send [`t`](#t)
| Parameter | Type  |  Description  |
|-----------|-------|---------------|
| t         | int   | Server time   |
| e         | int   | Client's time |

### n
Sent to an entire room, excluding the sender. Contains note data and noteBufferTime of the person who sent the message.
| Parameter |      Type       |                    Description                     |
|-----------|-----------------|----------------------------------------------------|
| n         | [note](#note)[] | Note buffer                                        |
| t         | int             | noteBufferTime                                     |
| p         | string          | participantId of the user who sent the note buffer |

### a
Sent to an entire room. It's a chat message.
| Parameter |       Type        | Description |
|-----------|-------------------|-------------|
| a         | string            | The message |
| p         | [uinfo](#uinfo)   | Sender info |
| t         | int               | Timestamp   |

Note: `t` is not used by the client and is therefore optional.

### m
Sent to an enire room, except the sender. Mouse movement.
| Parameter |  Type  |                   Description                   |
|-----------|--------|-------------------------------------------------|
| x         | float  | Mouse X                                         |
| y         | float  | Mouse Y                                         |
| id        | string | participantId of the user who moved their mouse |

### p
Sent to an entire room when a user updates their info.
| Parameter |  Type  |  Description  |
|-----------|--------|---------------|
| color     | string | User color    |
| name      | string | User name     |
| id        | string | participantId |
| \_id      | string | userId        |
| x         | float  | Mouse X       |
| y         | float  | Mouse Y       |

### ch
Changes a user's room. Used for room settings updates (sent to entire room), or moving a user to another room (sent to single user).
| Parameter |        Type         |                        Description                        |
|-----------|---------------------|-----------------------------------------------------------|
| ch        | [channel](#channel) | Channel info                                              |
| ppl       | [uinfo](#uinfo)+[]  | People in the channel                                     |
| p         | string              | Only present when moving rooms. User's new participantId. |

### c
Provides the client with the room's chat history. Should be sent after [ch](#ch-1).
| Parameter |   Type   | Description  |
|-----------|----------|--------------|
| c         | [a](#a-1)[] | Chat history |

### nq
Provides the client with new NoteQuota parameters.
| Parameter |   Type    | Description |
|-----------|-----------|-------------|
| params    | [nq](#nq) | New params  |

### notification
Send a notification.
| Parameter |  Type  |                      Description                       |
|-----------|--------|--------------------------------------------------------|
| id        | string | ID of the notification in HTML                         |
| title     | string | Title of the notification                              |
| text      | string | Text to show in the notification                       |
| html      | string | HTML to stick in the notification                      |
| target    | string | Where the notification should appear (jQuery selector) |
| duration  | int    | How long the notification will last, in milliseconds   |
| class     | string | Type of window to show                                 |

`class`: "classic" shows the title, while "short" does not.  
Note: Set the duration to -1 to make the notification persist until the user closes it.  
Note: Any of these are optional, the client has default values.
