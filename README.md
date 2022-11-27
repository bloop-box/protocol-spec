# Protocol Specification

Bloop Boxes use a binary protocol to communicate, layered over a TLS stream. The protocol is synchronous and split into individual request-response packages. Each package begins with a single byte denoting the type of the request or response.

## Authentication

Right after the connection is established, the client tries to authenticate against the server. This is done by sending a client ID and a secret, separated by a colon. The authentication string is prefixed by a UINT8 denoting the length of the string.

> Example: `\x07foo:bar`

The server should respond either with a `\x00` byte on authentication failure or `\x01` on success.

## Blooping

When the Bloop Box detects an NFC tag, it sends a `\x00` byte followed by the serial number of the NFC chip (7 bytes) to the server.

> Example: `\x00\xff\xff\xff\xff\xff\xff\xff`

The server then processes the bloop on its side and responds with one of the following:

| Code   | Description                                                                                                                                                                                                                     |
|--------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `\x00` | Serial number is not registered on the server.                                                                                                                                                                                  |
| `\x01` | Serial number is registered and bloop was acknowledged. This response is followed by a single UINT8 specifying the number of new achievements. For each achievement, its ID should be returned, which is a 20 byte binary hash. |
| `\x02` | Serial number is registered but the server enforced throttling.                                                                                                                                                                 |

## Retrieving audio

When the client receives a new achievement ID for which it doesn't have the audio file stored yet, it will issue an `\x01` command to the server followed by the achievement ID (20 bytes).

> Example: `\x01\xff\xff\xff\xff\xff\xff\xff\xff\xff\xff\…`

The server then responds with one of the following:

| Code   | Description                                                                                                                    |
|--------|--------------------------------------------------------------------------------------------------------------------------------|
| `\x00` | Achievement ID is not known or no audio file exists.                                                                           |
| `\x01` | Audio file is found. This code is followed by a UINT32LE specifying the length of the audio data, followed by the actual data. |

## Ping

In order to check connectivity, the client will periodically send a ping to the server. This is done by sending an `\x02` byte. The server should respond with an `\x00` byte.

