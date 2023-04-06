




# The Loki protocol: Protolok
Authors: Daniel Daniella H., Daniel Amanda H., Daniel Marcus H.


**Motivation**
Loki is meant to be a secure and fast chat app. To allow for this, the protocol has to be designed with special consideration given to these aspects. This RFC is meant to document the first draft of the protocol, and make sure extensive thought from multiple people is given to security.

**General architecture**
Protolok is designed to run on many servers, connecting them all, and relaying messages between the  users registered on some particular server and all others. This is called federation in the rest of this document. Protolok is also the protocol used by clients to communicate with their home servers (the ones they have a registered account on). All mentions of “message” in this section mean both encrypted and unencrypted, without a distinction.<br>
To achieve all this, we propose the following infrastructure:
    - Whenever a message is sent to a channel, the home server of the sender will look up which server the channel is hosted on (the channel’s home server) and notify this server of the message.
    - At the same time, it will notify all other home users (users it is a home server of) that are members of the channel of the message.
    - The home server of the channel will look up the list of members of the channel, and relay the message to their home servers to be handled, while also handling the message for its own users.
    - Those servers will then notify the users of that message, and serve the message to them in further requests for messages.
In short, 
`User -> Home server [-> Channel home server [-> Other users’ home servers]] -> Other users`

**Encryption architecture**
Because Loki is meant to be very secure, Protolok needs to be designed with security as a priority. We propose multiple types of channels: 
    1. Encrypted two-user channel (“DM channel”, “E-channel”)
    2. Encrypted multi-user channel (“Group channel”, “E-channel”)
    3. Encrypted guild channel (“P channel”, “Guild channels”, “E-channel”)
    4. Unencrypted guild channel (“Pub channel”, “Public Guild channel”, “Guild channels”)
DM channels are used for direct messages between two users.<br>
Group channels are used for groups, which are meant to be relatively small, but allow as many users as may be required. Servers should restrict the amount of allowed members so that this doesn’t stress the network. We recommend using a restriction of around 50 members. <br>
P channels are Guild channels in private guilds. They are meant for close-knit, non-public communities that won’t have a high member turnover rate. This is because keys will need to be regenerated whenever someone leaves.<br>
Pub channels are Guild channels in public guilds. They are meant for public communities and larger private communities with bigger turnover rates. They are the only unencrypted channels.<br>

Much of the encryption should happen client-side, usually using a password-derived key or similar. However, this has the drawback of making password resets impossible without destroying message logs.

Federation of messages does not depend on whether they are encrypted. They are handled identically.

**Messaging protocol**
Whenever a # property is specified, it can be left out but is useful for debugging purposes, so it is recommended to include.<br>
To send an encrypted message, the client will POST /channels/encrypted/[channel-id]. The content of this request should be a JSON document with the following content: 
```json
{
  “#type”: “message”,
  “in-reply-to”: “[message id of message this is in reply to]”,
  “mentions”: [
    [list of user tags this message mentions, in following format: 
     “TudbuT:lokichat.online”]
  ],
  “content”: {
    “#type”: “message-content-block”,
    “encrypted”: true,
    “hash”: “[sha-512 of the unencrypted text]”,
    “text”: “[RSA2048-encrypted text, using pubkey of the receiver]”
  }
}
```
The server (in this example, “lokichat.online”) will then look up the channel’s `home-server` property (in this example, “loki.tudbut.de”), and send the message over to that server in the following format to /fed/channels/[channel-id]: 
```json
{
  “#type”: “message-federation”,
  “id”: [generated id, in the format “T.UUUUUU:SSSSSS” where `T.` is the unix time as a hex string, UUUUUU is the first 6 hex digits of the sha256 hash of the `from` field, and SSSSSS is the first 6 hex digits of the sha256 hash of the sender’s home server domain]
  “from”: “TudbuT”,
  “deliver-back”: [if the server wants the message to be sent back to it as if it came from another server, true or false],
  “in-reply-to”: [unchanged],
  “mentions”: [unchanged],
  “content”: [unchanged],
}
```
The channel’s home server will then look up the channel’s `receivers` property and relay this exact JSON to them as well, also using the fed endpoint.<br>
All servers who now know of the message will then notify connected users who are members of the channel of it via WebSocket. They will also add the message to the channel log.

Federation can be turned off, in which case only the server the message was sent from will handle it.

**Requesting messages**
Any server which has a member within the channel can request its messages via GET /channels/[channel-id]/messages. A user can request messages from their home server via the same API path, but will have to provide their authorization header as with all requests sent by users.

**Account registration**
A user must have an account to be addressable, and to send messages. This account is on their home server, and created through a log-in form. For federation, no accounts are created on the other servers. User details are only ever requested from that account’s home server, but may be cached. Users can only log in at their home server.

**Encryption**
The home server of any member stores a keyring with keys to all encrypted channels the user is in. This keyring is accessible from the client, and will never be decrypted by the server. It is only ever written to or read by the client to add, change, or read keys. These keys are used to decrypt and encrypt messages in the channels the client is a member of.

When the user first joins an E-channel, it will be asked for a password to decrypt the channel’s keys client-side and store them into the keyring. The password may be included in the invite code.

A significant problem arises when a user leaves an E-channel, the messages need to be re-encrypted with a new key so that the user no longer has the ability to decrypt them or any future messages. For this, a new key will be generated by any of the users, chosen at random, which will be used to, on that user’s machine, decrypt all messages in the channel, and re-encrypt them with the new key. To avoid malicious actors, three other clients in the channel will be asked to check the hashes of the messages with the new contents, to make sure the message was not altered. If a message was said to have been altered by two of the three, the re-encryption will be reverted by the server (by simply restoring previous data) and the process repeated by another encryptor client. If there are not enough users available, it is assumed that the client(s) who did not do the encryption are more trustworthy, and a revert will happen if any of them flag the messages as altered.

Because this requires significant compute resources, the encryption process may be done by multiple clients in parallel, each being assigned some part of the chat history.

The server must keep the original copy of the messages until the messages are confirmed by (an)other client(s). After this, the server may delete them, but can also wait for more clients to confirm them and issue a re-encryption if some amount of clients end up disagreeing. It is only mandatory for the server to keep the messages until they have been agreed upon by the majority-of-three procedure described above. We recommend to choose slihtly higher numbers (i.e. a three-of-four agreement threshold) if the channel is big enough. However, this may cause delays when not many clients are logged in.

If the clients are behind a layer of federation, their home server must forward the requests for checking and re-encryption. It is recommended to not rely on a single server here, as it could otherwise introduce false results if a server is malicious. Spreading the checks and re-encryption as much as possible is therefore highly recommended. If only users from few servers are online, the original copy should be kept until other servers agree, if possible.
