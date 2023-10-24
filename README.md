# Bloop Protocol Specification

Version: 2 (2023-10-24)

Bloop Boxes use a binary protocol to communicate, layered over a TLS stream. The protocol is synchronous and split into
individual request-response packages.

## Changelog

### Version 2:

- Authentication message format adjusted and local IP address added
- Achievement IDs changed to UUIDs
- Quit message added

## Types

The following types are used in this protocol:

| Type        | Description                                                              |
|-------------|--------------------------------------------------------------------------|
| `uint8`     | Single byte unsigned integer                                             |
| `uint32_le` | Four byte unsigned integer                                               |
| `ip_addr`   | Type `uint8` (`4` or `6`) followed by respectively 4 or 16 bytes         |
| `string`    | Type `uint8` followed by the respective number of bytes as UTF-8 string. |
| `bytes`     | Type `uint32_le` followed by the respective number of bytes.             |
| `nfc_uid`   | 7 byte NFC UID                                                           |
| `uuid`      | 16 byte UUID                                                             |

## Message format

All messages except the authentication in this protocol have the following format:

```
uint8   Message type
uint8[] Optional payload. The length must be inferred from the message type and payload contents.
```

A client must not send a new request until it received a response from the server.

## Authentication

After the connection is established, the client must send its credentials and local IP address in order to authenticate
with the server. The authentication message has the following format:

```
string  Client ID
string  Secret
ip_addr Local IP address
```

The server must respond with one of the following messages:

```
\0x00   Authentication failed
```

```
\0x01   Authentication succeeded
```

## Messages

### Blooping

This message is send when the client detects an NFC tag.

#### Request

```
\x00    NFC tag detected
nfc_uid UID of the scanned tag
```

#### Responses

```
\x00    UID is not registered
```

```
\x01    UID is registered
uint8   Number of achievements
uuid[]  Array of achievement IDs
```

```
\x02    UID is registered but server enforced throttling
```

### Retrieving audio

When the client receives a new achievement ID for which it doesn't have the audio file stored, it should issue the
following message:

#### Request

```
\x01    Achievement audio request
uuid    Achievement ID
```

#### Responses

```
\x00    Achievement ID unknown or audio file missing
```

```
\x01    Audio file found
bytes   Audio file payload (MP3 format)
```

### Ping

Periodically the client should send a ping to the server to verify connectivity.

#### Request

```
\x02    Ping
```

#### Response

```
\x00    Pong
```

### Closing connection

To facilitate a clean shutdown of the connection, the client should inform the server when it is shutting down.

#### Request

```
\x03    Quit
```

#### Response

No response will be sent by the server. Instead, it will close the socket immediately. The client should do the same
after sending the request.
