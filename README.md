# peersockets
[![Build Status](https://travis-ci.com/andrewosh/peersockets.svg?token=WgJmQm3Kc6qzq1pzYrkx&branch=master)](https://travis-ci.com/andrewosh/peersockets)

__Warning__: Extension messages are currently not MAC'd, so we can't detect tampering. This is still experimental!

Peersockets is a lightweight wrapper around [`@basestore/networker`](https://github.com/andrewosh/basestore-networker) that lets you exchange topic-tagged messages with peers over [`@ddatabase/protocol`](https://github.com/mafintosh/@ddatabase/protocol) streams.

To use a Peersocket, you first join a topic -- under the hood, this will register a topic-specific extension on every current peer connection in your networker. By defining topics at the extension level, Peersockets does not need to do any extra encoding/decoding. Once you've created a topic, you can send messages to peers by their NOISE keys. If the remote peer has also joined the topic, they will receive the message.

### Upgrading
If you've already been using peersockets v0, take a look at [`UPGRADE.md`](https://github.com/andrewosh/peersockets/blob/master/upgrade.md) to see what's changed in v1.

### Installation
```
npm i peersockets --save
```
### Example
This example assumes you already have Networker ([`@basestore/networker`](https://github.com/andrewosh/basestore-swarm-networking)) instance.
```
const networker = (my swarm networker)
const sockets = new Peersockets(networker)

// Let's say you have a friend's Peer object (which can be extracted from the Network's `peers` property or `peer-add` event).
const friendPeer = networker.peers[0]

const handle = sockets.join('my-topic', {
  onmessage: (msg, peer) => {
   console.log(`I got message ${msg} from ${peer.remotePublicKey.toString('hex')}`)
  }
})

// If your friend is also subscribed to topic 'my-topic', they will receive this message.
handle.send('hello!', friendPeer)
```
### API
#### `const socket = new Peersockets(networker)`
Creates a new Peersockets instance. 

* `networker` is an instance of [`@basestore/networker`](https://github.com/andrewosh/basestore-networker)).

#### `const handler = socket.join(topicName, opts = { onmessage, onclose })`
Joins a new topic. If this is the first time the topic has been joined, a topic extension will be registered on every connected peer. Otherwise the previous topic/extension will be reused.

* `topicName` can be any string
* `opts` can contain the following callbacks:
  * `onmessage(msg, peer)` - Called whenever a message with this topic is received from `peer`.
  * `onclose()` - Called if the topic is closed via `leave`.
  
The topic handler's API is described below.
  
#### `socket.leave([topicName])`
Close all topic handlers for `topicName` and unregister the topic's extension from all connected peers.

If a `topicName` is not specified, all topics will be left.

#### `topicHandle.send(msg, peer)`
Send a message to `peer`.

`msg` is a string or a Buffer.
`peer` is a `@basestore/networker` Peer instance that can be obtained from `networker.peers`.

#### `topicHandle.close()`
Close the handle when you're finished with it. This will not unregister the topic extension.

### License
MIT
