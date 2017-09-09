the disintermediated web

[watch the video](https://www.youtube.com/watch?v=6jcQoSraHcw&feature=youtu.be)

https://substack.neocities.org

---
# the web

an era of dumb terminals

---
# the web: new browser features

* webrtc
* IndexedDB
* Media{Recorder,Source}
* webgl

---
# the problem

too much technology depends on centrally controlled servers:

* requires an internet connection
* high latency
* copyright/political/economic censorship

---
# platform paternalism

* real-name policies
* requiring a mobile phone number
* platforms can turn off access whenever they want

---
# p2p

* datacenters are already distributed systems
* move the algorithms out to the clients
* then cut out the middlemen (servers)

---
# the best way to build a distributed p2p system

have a problem that centralized services cannot solve well:

* offline
* data replication
* live streaming
* very large files

(not an exaustive list)

---
# things that are hard with p2p

* global concensus (Don't have this problem in the first place)
* availability

---
# things that are easy with p2p

and hard/impossible with centralized services:

* infinitely scalable file transfer
* log-based data replication
* offline/marginal networking

---
# computing on the edges

EVERYTHING happens on the client.

The client is the only place where anything CAN happen.

Servers are clients, too.

---
# create an account

```
var sodium = require('chloride/browser')
var keypair = sodium.crypto_sign_keypair()
var keys = {
  publicKey: keypair.publicKey.toString('hex'),
  secretKey: keypair.secretKey.toString('hex')
}
console.log('PUBLIC KEY:', keys.publicKey)

var fs = require('fs')
fs.writeFile('keys.json', JSON.stringify(keys))
```

---
# sign

```
var sodium = require('chloride/browser')
var keypair = require('./keys.json')
var secretKey = Buffer(keypair.secretKey, 'hex')

var msg = new Buffer('hello london!')
var sig = sodium.crypto_sign_detached(msg, secretKey)
console.log(sig.toString('hex'))
```

---
# verify

```
var sodium = require('chloride/browser')
var publicKey = Buffer(require('./keys.json').publicKey, 'hex')
var sig = Buffer(process.argv[2], 'hex')

var msg = new Buffer('hello london!')
var ok = sodium.crypto_sign_verify_detached(sig, msg, publicKey)
console.log(ok)
```

---
# user id

public key or the hash of a public key

store the user id on a DHT or don't

---
# anarchitecture

architectures for services that nobody can own:

* DHTs (used by bittorrent, ipfs)
* kappa architecture (sometimes called "event sourcing")

---
# kappa architecture

append-only log is the source of truth

indexes (materialized views) are built on top of the log

---
# some node/browser primitives

* webrtc-swarm
* hyperlog
* hypercore
* hyperdrive
* webtorrent/torrent-stream
* ipfs

---
# randomized gossip protocol

* connect to random peers interested in the same topic
* replicate data

(easy to build with webrtc-swarm)

---
# p2p feed publisher

first, some modules...

```
var swarm = require('webrtc-swarm')
var signalhub = require('signalhub')
var memdb = require('memdb')
var sodium = require('chloride/browser')
var hsodium = require('hyperlog-sodium')
var split = require('split2')
var through = require('through2')
```
---
# p2p feed publisher

set up the swarm and hyperlog

```
var keypair = sodium.api.crypto_sign_keypair()
var log = hyperlog(memdb(), hsodium(sodium, keypair))

var hubs = [ 'https://mafintosh.signalhub.com' ]
var sw = swarm(signalhub(keypair.publicKey.toString('hex'), hubs))

```

---
# p2p feed publisher

populate the log

```
process.stdin.pipe(split()).pipe(through(function (buf, enc, next) {
  log.append(buf, function (err, node) {
    if (err) console.error(err)
    else console.log(node.key)
  })
}))
```

---
# p2p feed publisher

gossip!

```
sw.on('peer', function (peer, id) {
  peer.pipe(log.replicate()).pipe(peer)
})
```

---
# p2p feed subscriber

first, some modules...

```
var swarm = require('webrtc-swarm')
var signalhub = require('signalhub')
var memdb = require('memdb')
var sodium = require('chloride/browser')
var hsodium = require('hyperlog-sodium')
```

---
# p2p feed subscriber

set up the swarm and hyperlog

```
var publicKey = process.argv[2]
var log = hyperlog(memdb(), hsodium(sodium, { publicKey: publicKey }))

var hubs = [ 'https://mafintosh.signalhub.com' ]
var sw = swarm(signalhub(publicKey, hubs))
```

---
# p2p feed subscriber

subscribed!

```
log.createReadStream({ live: true })
  .on('data', console.log)

sw.on('peer', function (peer, id) {
  peer.pipe(log.replicate()).pipe(peer)
})
```

---
# p2p feed

* your followers will help host your data when you are offline
* aside from the signalhub, direct p2p connections

---
# demo: p2p irc

https://substack.neocities.org/chatwizard/#nodeconflondon

https://github.com/substack/chatwizard

---
# demo: osm-p2p

* https://github.com/digidem/dd-map-editor
* https://github.com/digidem/osm-p2p

---
# demo: p2p live streaming

https://github.com/substack/spellcast

---

