# mpp-protocol
Legacy protocol of the Multiplayer Piano.
<details>
  <summary>Table of Contents</summary>

  - [Basic Types](#basic-types)
  - [Types of IDs](#types-of-ids)
  - [Message Structure](#message-structure)
    - [example](#message-structure-example)
  - [Nested Objects](#nested-objects)
    - [note](#nested-objects-note)
    - [chset](#nested-objects-chset)
    - [userset](#nested-objects-userset)
    - [uinfo](#nested-objects-uinfo)
    - [point](#nested-objects-point)
    - [crown](#nested-objects-crown)
    - [channel](#nested-objects-channel)
    - [nq](#nested-objects-nq)
  - [Client -> Server](#cs)
    - [hi](#cs-hi)
    - [bye](#cs-bye)
    - [+ls](#cs-pls)
    - [-ls](#cs-mls)
    - [t](#cs-t)
    - [a](#cs-a)
    - [n](#cs-n)
    - [m](#cs-m)
    - [userset](#cs-userset)
    - [kickban](#cs-kickban)
    - [unban](#cs-unban)
    - [ch](#cs-ch)
    - [chset](#cs-chset)
    - [chown](#cs-chown)
  - [Server -> Client](#sc)
    - [hi](#sc-hi)
    - [bye](#sc-bye)
    - [ls](#sc-ls)
    - [t](#sc-t)
    - [n](#sc-n)
    - [a](#sc-a)
    - [m](#sc-m)
    - [p](#sc-p)
    - [ch](#sc-ch)
    - [c](#sc-c)
    - [nq](#sc-nq)
    - [notification](#sc-notification)
</details>

## Basic Types
| Type | Description |
|---|---|
| string | A good old string, as seen in many many _many_ different programming and scripting languages. |
| boolean | A value with two states, namely `true` and `false`. This is also seen in many languages. |
| int | A whole number, e.g. `103`. |
| float | A fractional number, e.g. `103.82`. |
| type[] | Any type suffixed with '[]' is an array. |

## Types of IDs
A `userId` (or `_id`) is the hexadecimal representation of a cryptographic sum, generated using the client's IP address combined with a salt of some sort. It identifies that client across the protocol.  
A `participantId` (or `id`) is similar to a _userId_. It identifies the client in a single room, and changes if the client were to leave and rejoin the room. Any subsequent connections from clients with the same _userId_ will receive the _participantId_ of the client(s) already in the room.  
Both IDs are shortened to 24 characters.

## Message Structure
A message's name is in the header. It is sent as the value of a property named `m`, along with the other properties in the message.
### <span id="message-structure-example">example</span>
This is an example message.
| Property | Type | Description | Valid data |
|---|---|---|---|
| awesome | boolean | Example property | |
| u | [uinfo](#nested-objects-uinfo) | Example nested properties | |

> The resulting message structure would be as follows:
> ```json
> {
> 	"m": "example",
> 	"awesome": true,
> 	"u": {
> 		"_id": "f114e78aeff86657ce65a66c",
> 		"name": "Anonymous",
> 		"color": "#80e480",
> 		"id": "96027cb27933773a4f078342"
> 	}
> }
> ```

## Nested Objects
### <span id="nested-objects-note">note</span>
Info about a single piano note.
| Property | Type | Description | Valid data |
|---|---|---|---|
| n | string | Note to press, e.g. `fs4`. | Presumably any string, with length <= 5 |
| v | float | Note velocity (volume). | 0.00 to 1.00 |
| d | int | Delay in milliseconds before the note will be played. | |
| s | boolean | When `true`, releases a key. | |

### <span id="nested-objects-chset">chset</span>
Room settings.
| Property | Type | Description | Valid data |
|---|---|---|---|
| visible | boolean | Is the room visible in [`ls`](#sc-ls)? | |
| chat | boolean | Does the room have chat? | |
| crownsolo | boolean | Only the owner can play? | |
| lobby | boolean | Whether or not the room is a lobby. | |
| color | string | Primary room accent color. | 6 hexadecimal characters prefixed with # |
| color2 | string | Secondary room accent color. | 6 hexadecimal characters prefixed with # |

> Clients used to be able to turn rooms into lobbies by sending [chset](#cs-chset) with the `lobby` property set to `true`. This was eventually fixed.

> The `lobby` property is not present if the room is not a lobby.

### <span id="nested-objects-userset">userset</span>
Properties for the [`userset`](#cs-userset) message.
| Property | Type | Description | Valid data |
|---|---|---|---|
| name | string | Client name | Any string with length <= 40 |
| color | string | Client color | 6 hexadecimal characters prefixed with # |

> The `color` property is ignored by the server so that a client cannot change its own color.

### <span id="nested-objects-uinfo">uinfo</span>
Client information.
| Property | Type | Description | Valid data |
|---|---|---|---|
| \_id | string | Client [_userId_](#types-of-ids) | |
| name | string | Client name | Any string with length <= 40 |
| color | string | Client color | 6 hexadecimal characters prefixed with # |
| id | string | Client [_participantId_](#types-of-ids) | |

> `id` is only present when the client has joined a room.

### <span id="nested-objects-point">point</span>
A pair of two-dimensional coordinates.
| Property | Type | Description | Valid data |
|---|---|---|---|
| x | float | X position | |
| y | float | Y position | |

### <span id="nested-objects-crown">crown</span>
Room crown information.
| Property | Type | Description | Valid data |
|---|---|---|---|
| userId | string | [_userId_](#types-of-ids) of the last known crown holder. | |
| participantId | string | [_participantId_](#types-of-ids) of the crown holder. | |
| time | int | Timestamp of the last crown update. | Unix epoch with milliseconds |
| startPos | [point](#nested-objects-point) | Start point for the crown drop animation. | |
| endPos | [point](#nested-objects-point) | End point for the crown drop animation. | |

> When the crown is dropped, a timeout of 15 seconds takes place before anyone can pick it up.  
> The `userId` property is used by the server to allow the client with that [_userId_](#types-of-ids) to bypass this timeout.

> The `participantId` property identifies the current holder of the crown.

> The crown drop animation takes 2 seconds on the traditional client.

### <span id="nested-objects-channel">channel</span>
Room information.
| Property | Type | Description | Valid data |
|---|---|---|---|
| count | int | Number of clients in the room. | |
| crown | [crown](#nested-objects-crown) | Room crown information | |
| settings | [chset](#nested-objects-chset) | Room settings | |
| \_id | string | Room name | Any string with length <= 512 |

> If the `lobby` setting is `true`, the `crown` property is not present.

> Rooms whose names are prefixed with "test/" count as lobbies.  
> "test/" by itself does not count, however.

> On the traditional client, the piano spins clockwise in rooms whose names are suffixed with "/spin".

### <span id="nested-objects-nq">nq</span>
Note quota information.
| Property | Type | Description | Valid data |
|---|---|---|---|
| allowance | int | Points to return to the client per tick. | |
| max | int | Maximum amount of points. | |
| maxHistLen | int | Maximum history length. | |

> Note quota is returned at a tick rate of 2000ms.

> Note quota keeps track of point history according to the `maxHistLen` property. Current points are added to an array each tick. When spending quota, array entries are added to a sum. If the sum is <= 0, the amount the client is trying to spend is multiplied by the `allowance`.

> The `maxHistLen` property is always `3`.

## <span id="cs">Client -> Server</span>
### <span id="cs-hi">hi</span>
Requests client info from the server.
| Property | Type | Description | Valid data |
|---|---|---|---|

### <span id="cs-bye">bye</span>
Causes the server to disconnect the client.
| Property | Type | Description | Valid data |
|---|---|---|---|

### <span id="cs-pls">+ls</span>
Requests list of rooms and subsequent room updates from the server.
| Property | Type | Description | Valid data |
|---|---|---|---|

### <span id="cs-mls">-ls</span>
Requests that the server stop sending room updates.
| Property | Type | Description | Valid data |
|---|---|---|---|

### <span id="cs-t">t</span>
Client heartbeat.
| Property | Type | Description | Valid data |
|---|---|---|---|
| e | int | Client time. | Unix epoch with milliseconds |

> Sent every 20 seconds by the traditional client. If 40 seconds pass with no heartbeat, the server kicks the client.

### <span id="cs-a">a</span>
Chat message.
| Property | Type | Description | Valid data |
|---|---|---|---|
| message | string | Message content. | Any string with length <= 256 |

### <span id="cs-n">n</span>
Note batch.
| Property | Type | Description | Valid data |
|---|---|---|---|
| t | int | noteBufferTime plus serverTimeOffset | |
| n | [note](#nested-objects-note)[] | Note buffer | |

### <span id="cs-m">m</span>
Update mouse position.
| Property | Type | Description | Valid data |
|---|---|---|---|
| x | float | Mouse X position. | |
| y | float | Mouse Y position. | |

### <span id="cs-userset">userset</span>
Update client info.
| Property | Type | Description | Valid data |
|---|---|---|---|
| set | [userset](#nested-objects-userset) | Client info | |

### <span id="cs-kickban">kickban</span>
Ban a client from the room.
| Property | Type | Description | Valid data |
|---|---|---|---|
| \_id | string | [_userId_](#types-of-ids) of the client to ban. | |
| ms | number | How long the ban will last, in milliseconds. | |

> Banned clients are sent to the room "test/awkward" with a [short](#sc-notification) notification telling them they've been banned from the room, and for how long.  
> The server sends another [short](#sc-notification) notification to the original room, notifying other clients of who was banned, and for how long.

> Banning oneself works just the same, except that the original room gets a [classic](#sc-notification) notification, titled "Certificate of Award", with "Let it be known that [name] kickbanned him/her self." as the body.

### <span id="cs-unban">unban</span>
Unban a client from the room.
| Property | Type | Description | Valid data |
|---|---|---|---|
| \_id | string | [_userId_](#types-of-ids) of the client to unban. | |

### <span id="cs-ch">ch</span>
Join or create a room.
| Property | Type | Description | Valid data |
|---|---|---|---|
| \_id | string | Room name | Any string with length <= 512 |
| set | [chset](#nested-objects-chset) | Room settings | |

> If the room already exists and the client holds the crown, or if creating a new room, the `set` property will change that room's settings.

### <span id="cs-chset">chset</span>
Update room settings.
| Property | Type | Description | Valid data |
|---|---|---|---|
| set | [chset](#nested-objects-chset) | Room settings | |

> The `set` property does not have to include every setting. Including only one or two settings will update those settings without affecting the others.

### <span id="cs-chown">chown</span>
"**ch**ange **own**ership". Interacts with the crown.
| Property | Type | Description | Valid data |
|---|---|---|---|
| id | string | [_participantId_](#types-of-ids) of the client to transfer the crown to. | |

> The `id` property is optional. If it does not refer to a valid [_participantId_](#types-of-ids) or is not present, the crown is dropped. If the crown is already dropped, the client retrieves the crown regardless of the `id` property.

## <span id="sc">Server -> Client</span>
### <span id="sc-hi">hi</span>
Sent in response to [`hi`](#cs-hi). Gives a client its info, accompanied by info about the server.
| Property | Type | Description | Valid data |
|---|---|---|---|
| u | [uinfo](#nested-objects-uinfo) | Client info | |
| t | int | Server time | Unix epoch with milliseconds |
| v | string | Server version string | |
| motd | string | Server MOTD | |

> The `v` and `motd` properties are not used by the traditional client in any way.

> Original MOTD was "You agree to read this message."

### <span id="sc-bye">bye</span>
Tells clients that a client left the room.
| Property | Type | Description | Valid data |
|---|---|---|---|
| p | string | [_participantId_](#types-of-ids) of the client that left. | |

> Sent to all clients in the room, except the client that left.

### <span id="sc-ls">ls</span>
Tells interested clients the list of rooms and subsequent room updates.
| Property | Type | Description | Valid data |
|---|---|---|---|
| c | boolean | Whether or not this is a bulk update. | |
| u | [channel](#nested-objects-channel)[] | Array of room info. | |

> The `c` property is `true` when sending the initial list of rooms, after which this message is sent for each update to any room, including creation of new rooms, and the `c` property is `false`.
> Subsequent updates to a room that is made invisible (`visible` set to `false` in the [room settings](#nested-objects-chset)) are ignored until it is made visible again.

### <span id="sc-t">t</span>
Server heartbeat. Sent in response to [`t`](#cs-t).
| Property | Type | Description | Valid data |
|---|---|---|---|
| t | int | Server time | Unix epoch with milliseconds |
| e | int | Client time as specified in the `e` property on the original [`t`](#cs-t) message. | |

### <span id="sc-n">n</span>
Note batch. Sent to an entire room, except the sending client.
| Property | Type | Description | Valid data |
|---|---|---|---|
| n | [note](#nested-objects-note)[] | Note buffer | |
| t | int | noteBufferTime as specified in the `t` property on the original [`n`](#cs-n) message. | |
| p | string | [_participantId_](#types-of-ids) of the sending client. | |

### <span id="sc-a">a</span>
Chat message. Sent to an entire room.
| Property | Type | Description | Valid data |
|---|---|---|---|
| a | string | The message | |
| p | [uinfo](#nested-objects-uinfo) | Sender info | |
| t | int | Timestamp | Unix epoch with milliseconds |

> The `t` property is not used by the traditional client.

### <span id="sc-m">m</span>
Mouse position update. Sent to an enire room, except the sending client.
| Property | Type | Description | Valid data |
|---|---|---|---|
| x | float | Mouse X position. | |
| y | float | Mouse Y position. | |
| id | string | [_participantId_](#types-of-ids) of the sending client. | |

### <span id="sc-p">p</span>
Client info update. Sent to an entire room.
| Property | Type | Description | Valid data |
|---|---|---|---|
| color | string | Client color | 6 hexadecimal characters prefixed with # |
| name | string | Client name | Any string with length <= 40 |
| id | string | Client [_participantId_](#types-of-ids) | |
| \_id | string | Client [_userId_](#types-of-ids) | |
| x | float | Mouse X position. | |
| y | float | Mouse Y position. | |

### <span id="sc-ch">ch</span>
Room update. Changes the room information on a client.
| Property | Type | Description | Valid data |
|---|---|---|---|
| ch | [channel](#nested-objects-channel) | Room info. | |
| ppl | [`p`](#sc-p)[] | Clients in the room. | |
| p | string | Client's new [_participantId_](#types-of-ids). | |

> The `p` property is only present when transferring a client to a different room.

### <span id="sc-c">c</span>
Room chat history. Should be sent after [`ch`](#sc-ch).
| Property | Type | Description | Valid data |
|---|---|---|---|
| c | [`a`](#sc-a)[] | Array containing the last 32 chat messages. | |

### <span id="sc-nq">nq</span>
Provides a client with new note quota parameters.
| Property | Type | Description | Valid data |
|---|---|---|---|
| params | [nq](#nested-objects-nq) | New note quota parameters | |

### <span id="sc-notification">notification</span>
Send a notification.
| Property | Type | Description | Valid data |
|---|---|---|---|
| id | string | Unique identifier for the notification. | |
| title | string | Notification title | |
| text | string | Plain text body | |
| html | string | HTML body | HTML |
| target | string | Where the notification should appear. | HTML query selector |
| duration | int | How long the notification will be shown, in milliseconds. | |
| class | string | HTML classes to add. | |

> The `id` property is prefixed with "Notification-" and used as the ID for the notification HTML element.

> On the traditional client, the "classic" `class` shows the `title`.  
> The "short" `class` does not show the `title`.

> Setting the `duration` below `1` prevents the notification from disappearing.

> All properties are optional. The client is expected to have default values.
