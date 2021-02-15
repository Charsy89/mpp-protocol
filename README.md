# mpp-protocol
Standard protocol of Multiplayer Piano
The types used here are [Java types](https://www.w3schools.com/java/java_data_types.asp) (not JavaScript, they're two different things).
Other types
### note
| Parameter |  Type   |       Description        |
|-----------|---------|--------------------------|
|     n     | String  |        Note name         |
|     v     | double  |         Volume           |
|     d     |  long   |          Delay           |
|     s     | boolean | Whether to stop the note |

### chset
| Parameter |  Type   |         Description          |
|-----------|---------|------------------------------|
|  visible  | boolean |     Is the room visible?     |
|   chat    | boolean |   Does the room have chat?   |
| crownsolo | boolean |   Only the owner can play?   |
|   lobby   | boolean | Whether the room is a lobby. |
|   color   | String  |     Inner room color.        |
|  color2   | String  |     Outer room color.        |

Note: The `lobby` value is disregarded by the server, meaning you can't create lobbies with custom names. Only using test/.

## Serverbound
### hi
Initializes the client on the server.  
(no parameters)

### t
Used to sync time between the client and the server. Must be sent in under 43~ seconds after the last ping, or after connecting.
| Parameter | Type |         Description          |
|-----------|------|------------------------------|
|     e     | long | Client time in milliseconds. |

### m
Sends your mouse position. People can fully see your cursor **between** 0 and 101.
| Parameter | Type  | Description |
|-----------|-------|-------------|
|     x     | float |   Mouse X   |
|     y     | float |   Mouse Y   |

### +ls
Tells the server you want to listen for rooms. The server first will send back a [ls](#ls) message containing the full room list. After that, every time a room updates, it will send ls with that room's info.
(no parameters)

### -ls
Tells the server you don't want to listen for rooms anymore.

### n
Sends the client's note buffer to the server.
| Parameter |  Type  |             Description              |
|-----------|--------|--------------------------------------|
|     t     |  long  | noteBufferTime plus serverTimeOffset |
|     n     | note[] |             Note buffer              |

### ch
Changes the channel you're in.
| Parameter |  Type  |  Description  |
|-----------|--------|---------------|
|    \_id    | String |   Room name   |
|    set    | chset  | Room settings |
