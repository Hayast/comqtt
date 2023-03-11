
<p align="center">
  
[![Build Status](https://travis-ci.com/wind-c/comqtt.svg?token=59nqixhtefy2iQRwsPcu&branch=master)](https://travis-ci.com/wind-c/comqtt)
[![contributions welcome](https://img.shields.io/badge/contributions-welcome-brightgreen.svg?style=flat)](https://github.com/wind-c/comqtt/issues)
[![codecov](https://codecov.io/gh/wind-c/comqtt/branch/master/graph/badge.svg?token=6vBUgYVaVB)](https://codecov.io/gh/wind-c/comqtt)
[![GoDoc](https://godoc.org/github.com/wind-c/comqtt?status.svg)](https://pkg.go.dev/github.com/wind-c/comqtt)

</p>

# Comqtt 
### A lightweight, high-performance MQTT server in Go (v3.0｜v3.1.1｜v5.0)

Comqtt is an embeddable high-performance MQTT broker server written in Go, and supporting distributed cluster, and compliant with the MQTT v3.0 and v3.1.1 and v5.0 specification for the development of IoT and smarthome projects. The server can be used either as a standalone binary or embedded as a library in your own projects. Comqtt message throughput is comparable with everyone's favourites such as Mosquitto, Mosca, and VerneMQ.

:+1: Comqtt code is cleaner, easier to read, customize, and extend than other Mqtt Broker code! :heart_eyes:

:+1: If you like or useful to you, please give a STAR, let more know and participate in maintenance together! :muscle:

> #### 📦 💬 See Github Discussions for discussions about releases
> Ongoing discussion about current and future releases can be found at https://github.com/wind-c/comqtt/discussions

#### What is MQTT?
MQTT stands for MQ Telemetry Transport. It is a publish/subscribe, extremely simple and lightweight messaging protocol, designed for constrained devices and low-bandwidth, high-latency or unreliable networks. [Learn more](https://mqtt.org/faq)

#### Comqtt Features
- Full MQTTv5 Feature Compliance, compatibility for MQTT v3.1.1 and v3.0.0.
- TCP, Websocket, (including SSL/TLS) and Dashboard listeners.
- File-based server, auth, storage and bridge configuration, [Click to see config examples](cmd/config).
- Auth and ACL Plugin is supported Redis, HTTP, Mysql and PostgreSql.
- Packets are bridged to kafka according to the configured rule.
- Single-machine mode supports local storage BBolt, Badger and Redis.
- Hook design pattern makes it easy to develop plugins for Auth, Bridge, and Storage.
- Cluster support is based on Gossip and Raft, [Click to Cluster README](cmd/cluster/README.md).

#### Roadmap
- Dashboard.
- Rule engine.
- Bridge(Other Mqtt Broker、RocketMQ、RabbitMQ).
- Enhanced Metrics support.
- CoAP.

## Quick Start
### Running the Broker with Go
Comqtt can be used as a standalone broker. Simply checkout this repository and run the [cmd/single/main.go](cmd/single/main.go) entrypoint in the [cmd](cmd) folder which will expose tcp (:1883), websocket (:1882), and dashboard (:8080) listeners.

### Build
```
cd cmd/single
go build -o comqtt cmd/single/main.go

```
### Start
```
./comqtt
or
./comqtt --conf=./config/single.yml
```
*If you want to obtain the bridge and multiple authentication capabilities, you need to use the configuration file to start.[Click to config example](cmd/config/single.yml).*

### Using Docker
A simple Dockerfile is provided for running the [cmd/single/main.go](cmd/single/main.go) Websocket, TCP, and Stats server:

```sh
docker build -t comatt:latest .
docker run -p 1883:1883 -p 1882:1882 -p 8080:8080 comqtt:latest
```

## Developing with Comqtt
### Importing as a package
Importing Comqtt as a package requires just a few lines of code to get started.
``` go
import (
  "log"

  "github.com/wind-c/comqtt/mqtt"
  "github.com/wind-c/comqtt/mqtt/hooks/auth"
  "github.com/wind-c/comqtt/mqtt/listeners"
)

func main() {
  // Create the new MQTT Server.
  server := mqtt.New(nil)
  
  // Allow all connections.
  _ = server.AddHook(new(auth.AllowHook), nil)
  
  // Create a TCP listener on a standard port.
  tcp := listeners.NewTCP("t1", ":1883", nil)
  err := server.AddListener(tcp)
  if err != nil {
    log.Fatal(err)
  }
  
  err = server.Serve()
  if err != nil {
    log.Fatal(err)
  }
}
```

Examples of running the broker with various configurations can be found in the [mqtt/examples](mqtt/examples) folder.

#### Network Listeners
The server comes with a variety of pre-packaged network listeners which allow the broker to accept connections on different protocols. The current listeners are:

| Listener | Usage |
| --- | --- |
| listeners.NewTCP | A TCP listener |
| listeners.NewUnixSock | A Unix Socket listener |
| listeners.NewNet | A net.Listener listener |
| listeners.NewWebsocket | A Websocket listener |
| listeners.NewHTTPStats | An HTTP $SYS info dashboard |

> Use the `listeners.Listener` interface to develop new listeners. If you do, please let us know!

A `*listeners.Config` may be passed to configure TLS.

Examples of usage can be found in the [mqtt/examples](mqtt/examples) folder or [cmd/single/main.go](cmd/single/main.go).

### Server Options and Capabilities
A number of configurable options are available which can be used to alter the behaviour or restrict access to certain features in the server.

```go
server := mqtt.New(&mqtt.Options{
  Capabilities: mqtt.Capabilities{
    MaximumSessionExpiryInterval: 3600,
    Compatibilities: mqtt.Compatibilities{
      ObscureNotAuthorized: true,
    },
  },
  SysTopicResendInterval: 10,
})
```

Review the mqtt.Options, mqtt.Capabilities, and mqtt.Compatibilities structs for a comprehensive list of options.


## Event Hooks
A universal event hooks system allows developers to hook into various parts of the server and client life cycle to add and modify functionality of the broker. These universal hooks are used to provide everything from authentication, persistent storage, to debugging tools.

Hooks are stackable - you can add multiple hooks to a server, and they will be run in the order they were added. Some hooks modify values, and these modified values will be passed to the subsequent hooks before being returned to the runtime code.

| Type | Import | Info |
| -- | -- |  -- |
| Access Control | [mqtt/hooks/auth . AllowHook](mqtt/hooks/auth/allow_all.go) | Allow access to all connecting clients and read/write to  all topics. |
| Access Control | [mqtt/hooks/auth . Auth](mqtt/hooks/auth/auth.go) | Rule-based access control ledger.  |
| Persistence | [mqtt/hooks/storage/bolt](mqtt/hooks/storage/bolt/bolt.go)  | Persistent storage using [BoltDB](https://dbdb.io/db/boltdb) (deprecated). |
| Persistence | [mqtt/hooks/storage/badger](mqtt/hooks/storage/badger/badger.go) | Persistent storage using [BadgerDB](https://github.com/dgraph-io/badger). |
| Persistence | [mqtt/hooks/storage/redis](mqtt/hooks/storage/redis/redis.go)  | Persistent storage using [Redis](https://redis.io). |
| Debugging | [mqtt/hooks/debug](mqtt/hooks/debug/debug.go) | Additional debugging output to visualise packet flow. |

Many of the internal server functions are now exposed to developers, so you can make your own Hooks by using the above as examples. If you do, please [Open an issue](https://github.com/wind-c/comqtt/issues) and let everyone know!

### Access Control
#### Allow Hook
By default, Comqtt uses a DENY-ALL access control rule. To allow connections, this must overwritten using an Access Control hook. The simplest of these hooks is the `auth.AllowAll` hook, which provides ALLOW-ALL rules to all connections, subscriptions, and publishing. It's also the simplest hook to use:

```go
server := mqtt.New(nil)
_ = server.AddHook(new(auth.AllowHook), nil)
```

> Don't do this if you are exposing your server to the internet or untrusted networks - it should really be used for development, testing, and debugging only.

#### Auth Ledger
The Auth Ledger hook provides a sophisticated mechanism for defining access rules in a struct format. Auth ledger rules come in two forms: Auth rules (connection), and ACL rules (publish subscribe).

Auth rules have 4 optional criteria and an assertion flag:
| Criteria | Usage |
| -- | -- |
| Client | client id of the connecting client |
| Username | username of the connecting client |
| Password | password of the connecting client |
| Remote | the remote address or ip of the client |
| Allow | true (allow this user) or false (deny this user) |

ACL rules have 3 optional criteria and an filter match:
| Criteria | Usage |
| -- | -- |
| Client | client id of the connecting client |
| Username | username of the connecting client |
| Remote | the remote address or ip of the client |
| Filters | an array of filters to match |

Rules are processed in index order (0,1,2,3), returning on the first matching rule. See [mqtt/hooks/auth/ledger.go](mqtt/hooks/auth/ledger.go) to review the structs.

```go
server := mqtt.New(nil)
err := server.AddHook(new(auth.Hook), &auth.Options{
    Ledger: &auth.Ledger{
    Auth: auth.AuthRules{ // Auth disallows all by default
      {Username: "peach", Password: "password1", Allow: true},
      {Username: "melon", Password: "password2", Allow: true},
      {Remote: "127.0.0.1:*", Allow: true},
      {Remote: "localhost:*", Allow: true},
    },
    ACL: auth.ACLRules{ // ACL allows all by default
      {Remote: "127.0.0.1:*"}, // local superuser allow all
      {
        // user melon can read and write to their own topic
        Username: "melon", Filters: auth.Filters{
          "melon/#":   auth.ReadWrite,
          "updates/#": auth.WriteOnly, // can write to updates, but can't read updates from others
        },
      },
      {
        // Otherwise, no clients have publishing permissions
        Filters: auth.Filters{
          "#":         auth.ReadOnly,
          "updates/#": auth.Deny,
        },
      },
    },
  }
})
```

The ledger can also be stored as JSON or YAML and loaded using the Data field:
```go
err = server.AddHook(new(auth.Hook), &auth.Options{
    Data: data, // build ledger from byte slice: yaml or json
})
```
See [examples/auth/encoded/main.go](mqtt/examples/auth/encoded/main.go) for more information.

### Persistent Storage
#### Redis
A basic Redis storage hook is available which provides persistence for the broker. It can be added to the server in the same fashion as any other hook, with several options. It uses github.com/go-redis/redis/v8 under the hook, and is completely configurable through the Options value.
```go
err := server.AddHook(new(redis.Hook), &redis.Options{
  Options: &rv8.Options{
    Addr:     "localhost:6379", // default redis address
    Password: "",               // your password
    DB:       0,                // your redis db
  },
})
if err != nil {
  log.Fatal(err)
}
```
For more information on how the redis hook works, or how to use it, see the [mqtt/examples/persistence/redis/main.go](mqtt/examples/persistence/redis/main.go) or [hooks/storage/redis](hooks/storage/redis) code.

#### Badger DB
There's also a BadgerDB storage hook if you prefer file based storage. It can be added and configured in much the same way as the other hooks (with somewhat less options).
```go
err := server.AddHook(new(badger.Hook), &badger.Options{
  Path: badgerPath,
})
if err != nil {
  log.Fatal(err)
}
```
For more information on how the badger hook works, or how to use it, see the [mqtt/examples/persistence/badger/main.go](mqtt/examples/persistence/badger/main.go) or [hooks/storage/badger](hooks/storage/badger) code.

There is also a BoltDB hook which has been deprecated in favour of Badger, but if you need it, check [mqtt/examples/persistence/bolt/main.go](mqtt/examples/persistence/bolt/main.go).



## Developing with Event Hooks
Many hooks are available for interacting with the broker and client lifecycle.
The function signatures for all the hooks and `mqtt.Hook` interface can be found in [mqtt/hooks.go](mqtt/hooks.go).

> The most flexible event hooks are OnPacketRead, OnPacketEncode, and OnPacketSent - these hooks be used to control and modify all incoming and outgoing packets.

| Function | Usage |
| -------------------------- | -- |
| OnStarted | Called when the server has successfully started.|
| OnStopped | Called when the server has successfully stopped. |
| OnConnectAuthenticate | Called when a user attempts to authenticate with the server. An implementation of this method MUST be used to allow or deny access to the server (see hooks/auth/allow_all or basic). It can be used in custom hooks to check connecting users against an existing user database. Returns true if allowed. |
| OnACLCheck | Called when a user attempts to publish or subscribe to a topic filter. As above. |
| OnSysInfoTick | Called when the $SYS topic values are published out. |
| OnConnect | Called when a new client connects |
| OnSessionEstablished | Called when a new client successfully establishes a session (after OnConnect) |
| OnDisconnect | Called when a client is disconnected for any reason. |
| OnAuthPacket | Called when an auth packet is received. It is intended to allow developers to create their own mqtt v5 Auth Packet handling mechanisms. Allows packet modification. |
| OnPacketRead | Called when a packet is received from a client. Allows packet modification. |
| OnPacketEncode | Called immediately before a packet is encoded to be sent to a client. Allows packet modification. |
| OnPacketSent | Called when a packet has been sent to a client. |
| OnPacketProcessed | Called when a packet has been received and successfully handled by the broker. |
| OnSubscribe | Called when a client subscribes to one or more filters. Allows packet modification. |
| OnSubscribed | Called when a client successfully subscribes to one or more filters. |
| OnSelectSubscribers | Called when subscribers have been collected for a topic, but before shared subscription subscribers have been selected. Allows receipient modification.|
| OnUnsubscribe | Called when a client unsubscribes from one or more filters. Allows packet modification. |
| OnUnsubscribed | Called when a client successfully unsubscribes from one or more filters. |
| OnPublish | Called when a client publishes a message. Allows packet modification. |
| OnPublished | Called when a client has published a message to subscribers. |
| OnPublishDropped | Called when a message to a client is dropped before delivery, such as if the client is taking too long to respond. |
| OnRetainMessage | Called then a published message is retained. |
| OnQosPublish | Called when a publish packet with Qos >= 1 is issued to a subscriber. |
| OnQosComplete | Called when the Qos flow for a message has been completed. |
| OnQosDropped | Called when an inflight message expires before completion. |
| OnWill | Called when a client disconnects and intends to issue a will message. Allows packet modification. |
| OnWillSent | Called when an LWT message has been issued from a disconnecting client. |
| OnClientExpired | Called when a client session has expired and should be deleted. |
| OnRetainedExpired | Called when a retained message has expired and should be deleted. |
| StoredClients |  Returns clients, eg. from a persistent store. |
| StoredSubscriptions |  Returns client subscriptions, eg. from a persistent store. |
| StoredInflightMessages | Returns inflight messages, eg. from a persistent store.  |
| StoredRetainedMessages | Returns retained messages, eg. from a persistent store. |
| StoredSysInfo | Returns stored system info values, eg. from a persistent store. |

If you are building a persistent storage hook, see the existing persistent hooks for inspiration and patterns. If you are building an auth hook, you will need `OnACLCheck` and `OnConnectAuthenticate`.


### Direct Publish
To publish basic message to a topic from within the embedding application, you can use the `server.Publish(topic string, payload []byte, retain bool, qos byte) error` method.

```go
err := server.Publish("direct/publish", []byte("packet scheduled message"), false, 0)
```
> The Qos byte in this case is only used to set the upper qos limit available for subscribers, as per MQTT v5 spec.

### Packet Injection
If you want more control, or want to set specific MQTT v5 properties and other values you can create your own publish packets from a client of your choice. This method allows you to inject MQTT packets (no just publish) directly into the runtime as though they had been received by a specific client. Most of the time you'll want to use the special client flag `inline=true`, as it has unique privileges: it bypasses all ACL and topic validation checks, meaning it can even publish to $SYS topics.

Packet injection can be used for any MQTT packet, including ping requests, subscriptions, etc. And because the Clients structs and methods are now exported, you can even inject packets on behalf of a connected client (if you have a very custom requirements).

```go
cl := server.NewClient(nil, "local", "inline", true)
server.InjectPacket(cl, packets.Packet{
  FixedHeader: packets.FixedHeader{
    Type: packets.Publish,
  },
  TopicName: "direct/publish",
  Payload: []byte("scheduled message"),
})
```

> MQTT packets still need to be correctly formed, so refer our [the test packets catalogue](mqtt/packets/tpackets.go) and [MQTTv5 Specification](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html) for inspiration.

See the [hooks example](mqtt/examples/hooks/main.go) to see this feature in action.

### Testing
#### Unit Tests
Comqtt tests over a thousand scenarios with thoughtfully hand written unit tests to ensure each function does exactly what we expect. You can run the tests using go:
```
go run --cover ./...
```

#### Paho Interoperability Test
You can check the broker against the [Paho Interoperability Test](https://github.com/eclipse/paho.mqtt.testing/tree/master/interoperability) by starting the broker using `examples/paho/main.go`, and then running the mqtt v5 and v3 tests with `python3 client_test5.py` from the _interoperability_ folder.

> Note that there are currently a number of outstanding issues regarding false negatives in the paho suite, and as such, certain compatibility modes are enabled in the `paho/main.go` example.


## Performance Benchmarks
Comqtt performance is comparable with popular brokers such as Mosquitto, EMQX, and others.

Performance benchmarks were tested using [MQTT-Stresser](https://github.com/inovex/mqtt-stresser) on a Apple Macbook Air M2, using `cmd/main.go` default settings. Taking into account bursts of high and low throughput, the median scores are the most useful. Higher is better.

> The values presented in the benchmark are not representative of true messages per second throughput. They rely on an unusual calculation by mqtt-stresser, but are usable as they are consistent across all brokers.
> Benchmarks are provided as a general performance expectation guideline only.

`mqtt-stresser -broker tcp://localhost:1883 -num-clients=2 -num-messages=10000`
| Broker            | publish fastest | median | slowest | receive fastest | median | slowest |
| --                | --             | --   | --   | --             | --   | --   |
| Comqtt v2.0.0      | 127,216 | 125,748 | 124,279 | 319,250 | 309,327 | 299,405 |
| Mosquitto v2.0.15 | 155,920 | 155,919 | 155,918 | 185,485 | 185,097 | 184,709 |
| EMQX v5.0.11      | 156,945 | 156,257 | 155,568 | 17,918 | 17,783 | 17,649 |

`mqtt-stresser -broker tcp://localhost:1883 -num-clients=10 -num-messages=10000`
| Broker            | publish fastest | median | slowest | receive fastest | median | slowest |
| --                | --             | --   | --   | --             | --   | --   |
| Comqtt v2.0.0      | 45,615 | 30,129 | 21,138 | 232,717 | 86,323 | 50,402 |
| Mosquitto v2.0.15 | 42,729 | 38,633 | 29,879 | 23,241 | 19,714 | 18,806 |
| EMQX v5.0.11      | 21,553 | 17,418 | 14,356 | 4,257 | 3,980 | 3,756 |

Million Message Challenge (hit the server with 1 million messages immediately):

`mqtt-stresser -broker tcp://localhost:1883 -num-clients=100 -num-messages=10000`
| Broker            | publish fastest | median | slowest | receive fastest | median | slowest |
| --                | --             | --   | --   | --             | --   | --   |
| Comqtt v2.0.0      | 51,044 | 4,682 | 2,345 | 72,634 | 7,645 | 2,464 |
| Mosquitto v2.0.15 | 3,826 | 3,395 | 3,032 | 1,200 | 1,150 | 1,118 |
| EMQX v5.0.11      | 4,086 | 2,432 | 2,274 | 434 | 333 | 311 |

> Not sure what's going on with EMQX here, perhaps the docker out-of-the-box settings are not optimal, so take it with a pinch of salt as we know for a fact it's a solid piece of software.

## Contributions
Contributions and feedback are both welcomed and encouraged! Open an [issue](https://github.com/wind-c/comqtt/issues) to report a bug, ask a question, or make a feature request.



