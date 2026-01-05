### The Spec
Below is a spec that I generated with chatgpt. It is purely a teaching artifact, not a published
standard. It is minimal by design and meant to be the first toe-dip into writing protocols.
Bear in mind that this is a fake thing, so the point is to understanding every little bit and piece
of it so that when I move on I can have a slightly better shot at understanding a real thing, or 
begin implementing a published standard.

Its
- Small enough to hold in my head (paramount importance)
- real enough to hurt
- constrained enough to finish

#### 1. The Protocol Spec (Minimal but Real)
- Name: LDP/1.0 — Length-Delimited Protocol
- Overview: Defines a simple request/response protocol for interacting with a remote key-value store
over a persistent TCP connection 

This protocol specifies:
  - message framing
  - message structure
  - message typees
  - payload formats
  - error handling requirements

#### 2. Transport
- The protocol must be carried over TCP
- A single connection may carry multiple requests
- The connection MUST be closed on unrecoverable protocol errors (Blocking I/O (no epoll, no async)

#### 3. Message Framing and structure
Each protocol message consists of a fixed-size header followed by a variable-length payload.

##### 3.1 Message Layout
0                   1                   2                   | 
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-------------------------------+-------+-------+-------+-------+
|          Length (uint32)      | Type  | Ver   |  Rsv  |  Rsv  |
+-------------------------------+-------+-------+-------+-------+
|                                                               |
|                         Payload (N bytes)                     |
|                                                               |
+---------------------------------------------------------------+

##### 3.2 Header Fields

| Field | Size | Description |
| ----- | ---- | ----------- |
| Length | 4 bytes | Total Message length in bytes, including header and payload. MUST be encoded in network byte order |
| Type | 1 byte | Message type identifier |
| Ver | 1 byte | Protocol version. Must be 0x01 |
| Rsv | 2 bytes | Reserved for future use. MUST be set to zero and ignored on receipt |

##### 3.3 Payload
- Payload length is `Length - 8` bytes
- Payload format depends on the message types
- Payload data is UTF-8 encoded text unless otherwise specified

#### 4. Message types

| Type | Name | Direction |
|------|------|-----------|
| 0x01 | PUT | Client -> Server |
| 0x02 | GET | Client -> Server |
| 0x03 | DEL | Client -> Server |
| 0x10 | OK | Server -> Client |
| 0x11 | ERROR | Server -> Client |

#### 5. Payload Formats
All payloads use newline {`\n`) as a delimiter.

##### 5.1 PUT
``
key\n
value\n
``

##### 5.2 GET
``
key\n
``

##### 5.3 DEL
``
key\n
``

##### 5.4 OK
``
(optional UTF-8 text)
``

##### 5.5 ERROR
``
error_code\n
error_message\n
``

#### 6. Constraints
- Maximum key length: 128 bytes
- Maximum value length: 1024 bytes
- Maximum total message length: 2048 bytes

Messages exceeding these limits MUST be rejected

#### 7. Error Handling
The server MUST respond with an ERROR message when:
- an unknown message type is received
- the protocol version is unsupported
- the message format is invalid

The server MUST immediately close the connection if:
- the Length field is invalid
- message framing fails
- payload size exceeds maximum limits

#### 8. Semantics
- PUT stores a key-value pair
- GET retrieves the value for a key
- DEL removes a key-value pair
- Requests are processed sequentially
- The server maintains an in-memory key-value store

#### 9. Versioning
- This document defines version `0x01`
- Future versions may change message semantics but must preserve header framing

#### 10. Compliance
An implementation is compliant with LDP/1.0 if it:
- follows the message format exactly
- enforces size constraints
- handles errors as specified
- processes messages in order


