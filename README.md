# Asynchronous Messaging and Eventing Resources

Once you've read through the intro, you will find that this document contains a
(growing) list of resources that cover whether and how to use asynchronous
messaging/queueing and eventing infrastructures with your applications
effectively and which you can use for self-study purposes. 

I am the product architect of the messaging and eventing services in the
Microsoft Azure cloud and have helped building these capabilities for over 15
years now, starting well before the name "Azure" even existed. Our team owns
multiple hyperscale broker platforms (Azure Service Bus, Event Hubs, Event Grid,
and Relay) that are deployed in hundreds of large clusters in over 60 Azure
regions all around the world and handle many (!) trillion (!) requests each day
(!). We also invented, prototyped, and productized several other Azure
capabilities (Azure IoT Hub, Azure Notification Hubs, Tenant Resource
Provisioning) that are now owned by different teams at Microsoft. 

In addition to the product work, I am deeply engaged in open interoperability
standards and represent Microsoft on the OASIS AMQP Technical Committee (as
co-chair), the OASIS MQTT Technical Committee, the CNCF Open Telemetry Working
Group, the CNCF CloudEvents project, and the CNCF Serverless Working Group. In
the past, I've also been a member of the OPC Foundations Technical Advisory
Council and the OPC UA Working Group where I've worked on the PubSub
specification in particular. Relevant technical documents are also linked below.

## "Why would I care?"

In addition to pointing to resources, allow me to also set some context with a
few definitions (some of which may be surprising) of some key concepts of
messaging and eventing. I will keep these brief since the concepts are covered
more broadly in many of the resources I point to.

Generally, if you are building software that requires more than one computer to do its job, you should know about asynchronous messaging concepts.

"Asynchronous" means here: Your application sends a message or event and then
carries on doing something else. It does not sit around waiting for an
outcome. For many developers, that is the first giant mental hurdle, since we've
all been raised on "imperative" programming models. You make a call and cause
some work to happen and only once that work has been reported as done (or
failed) your own code continues with its work.

Yet, we all use asynchronous messaging every day in the physical world. If we
want to send a gift to someone in a different city, we will go to the post
office and entrust the postal service with the package and that takes care of
getting it there and also tracking whether the package has been delivered
successfully. Once the package has been delivered, your friend might then call
you and excitedly thank you for the gift you sent. If you ask for it you will
get feedback, but maybe on a different channel and after however long the
handling and delivery took and usually with a clearly reference to the package
you sent.

That is what asynchronous messaging infrastructure does, but for your apps.

* Decoupling: A system handling work behind a messaging infrastructure can be
  running at capacity and yet not be overwhelmed and can even be down while the
  messaging infrastructure still accepts messages on its behalf.
* Delivery: You can entrust over your messages and the messaging system will try
  its best not lose them. It will then attempt to deliver them to the right
  parties and will retry as often as necessary.
* Buffering: A messaging infrastructure is generally great at accepting bursts
  of messages at once and organizing them for later retrieval. The retrieval can
  then occur at the pace that your application can handle. That is alow called
  load-leveling.
* Network-Bridging: Messaging infrastructures can often be attached to multiple
  networks, allowing information to pass between applications in those networks
  without there being IP-level connectivity between them.

### Messaging or eventing infrastructure

"Message broker", "queue", "service bus", "event router", "event stream engine",
"event aggregator", are all names for asynchronous messaging and eventing
infrastructure elements and the list is by no means exhaustive. I will give you
a brief definitions for all the words in those names for orientation.

* **Producer** - A producer (or _sender_ or _publisher_) is a role in a software
  system that wants to share/distribute information and therefore produces
  messages and makes them available via a messaging infrastructure.
* **Consumer**  - A consumer (or _receiver_ or _subscriber_) is a role in a
  software system that retrieves/gets messages from an messaging infrastructure and consumes them. Consuming often, but not always, means to act on the information.
* **Content** - Content (or _payload_ or _body_) is the information the producer
  wants the consumer(s) to receive and handle. Content may be any kind of data
  in any format.
* **Message** - A message is an envelope that wraps the content for transfer. It
  contains metadata annotations that helps the messaging infrastructure and
  frameworks understand how to route and handle the information, just like the
  addressing information and "express" stickers on a postal package or letter
  envelope.
* **Event** - An event is a variation of a message whose content reflects a
  fact. A fact is a historical statement of some past activity: "the milk carton
  dropped on the floor" is a fact that will forever be true when looking back at
  the morning of this day if it happened to you. The great thing about facts is
  that they can be easily distributed and cached and copied and transformed
  because the exact information they carry will forever be true and never again
  change. (These philosophical considerations matter a lot for arriving at smart architectures).
* **Job** - A job is a variation of a message whose content reflects an intent.
  The producer sends content with the intent of a consumer doing some work based
  on that content. "I just took this purchase order from a customer, please
  package and deliver these goods". Getting the job done might again involve
  multiple parties that handle parts if it, each being instructed through jobs.
* **Queue** - A queue is a messaging infrastructure entity that assigns the
  exclusive ownership and temporary control over the lifecycle of a message to
  one of potentially many competing consumers. The consumer can decide to
  finally accept the message which removes it from the queue and thus prevents
  the work from being performed again or to make it again available for
  consumption if an error prevented the work from being completed such that a
  retry is possible. The message can also be rejected and sidelined if it cannot
  be processed even with a retry. Message queues may also provide ordering
  assurances which are defining for the "queue" data structure in computer
  science, but that is optional.
* **Router** - A router ( or _topic_) is a messaging infrastructure entity that
  accepts messages from producers and dispatches them onwards to other messaging
  infrastructure entities or to consumers, often considering rules that inspect
  the message metadata annotations. Routers may often be configured dynamically
  and at runtime to deliver messages to interested parties, which are then
  called subscribers. 
* **Stream** - An event stream is a sequence of related events, which typically
  stem from the same producer or at least the same producer context (i.e.
  multiple producers create events about the same thing). An event stream engine
  may multiplex delivery of many concurrent event streams (occasionally also
  called _topic_) and may split those up across several physical logs
  (partitions) while keeping any one event stream together on a partition to
  ensure preserving order.
* **Checkpoint** - Event streams are usually processed by taking several events
  at a time. Since events are not jobs and therefore do not require exclusive
  handling, event stream engines therefore shift the burden of keeping track of
  what events have and have not been consumed to the consumer themselves.
  Consumer will periodically note a checkpoint relative to the stream and/or
  partition and resume from that noted checkpoint when needed. Some event stream
  brokers have internal facilities that help with noting those checkpoints.
* **Broker** - Broker (or _service bus_) is the term for a server or
  infrastructure that brokers messages and does so via queues or routers or
  streams.

## Patterns

* [Talk: "Messaging Patterns for Developers" (2021)](https://www.youtube.com/watch?v=ef1DK76rseM). This is a .NET talk, but conceptually also applicable to all other programming languages.
* [Talk: "Eventing, Serverless, and the Extensible Enterprise", Voxxed Athens 2018](https://www.youtube.com/watch?v=qCNXUUlhJJE)
* [Talk: "Events, data Points, and Messages", Thingmonk 2017](https://www.youtube.com/watch?v=ITrlLErsqzY&feature=emb_imp_woyt)
* [Events, Data Points, and Messages - Choosing the right Azure messaging service for your data](https://azure.microsoft.com/en-us/blog/events-data-points-and-messages-choosing-the-right-azure-messaging-service-for-your-data/)
* [Asynchronous messaging options](https://docs.microsoft.com/en-us/azure/architecture/guide/technology-choices/messaging)

## Open Standards

### CNCF CloudEvents

* [CloudEvents 1.0.2. Spec](https://github.com/cloudevents/spec/blob/v1.0.2/cloudevents/spec.md)
* [CloudEvents 1.0.02 Spec Repo](https://github.com/cloudevents/spec/blob/v1.0.2/cloudevents/) where you can find all the protocol bindings and format specs
* [CloudEvents working branch](https://github.com/cloudevents/spec/tree/main)
* [Talk: "CloudEvents Intro and Demos"](https://www.youtube.com/watch?v=yg7RuDWHwV8)
* [Talk: "CloudEvents 1.0 and Beyond"](https://www.youtube.com/watch?v=YpUQbxx3jkk)
* [Talk: "CloudEvents DeepDive", KubeCon Europe 2019](https://www.youtube.com/watch?v=-3gOqR_TGEs)


### OASIS AMQP 

* [AMQP 1.0](https://docs.oasis-open.org/amqp/core/v1.0/os/amqp-core-overview-v1.0-os.html) core standard document. Also ISO 19464:2014.
  * [AMQP Addressing Version 1.0](https://docs.oasis-open.org/amqp/addressing/v1.0/csd01/addressing-v1.0-csd01.html)
  * [AMQP Filter Expressions Version 1.0](https://docs.oasis-open.org/amqp/filtex/v1.0/csd01/filtex-v1.0-csd01.html)
  * [AMQP Link Pairing Version 1.0](https://docs.oasis-open.org/amqp/linkpair/v1.0/cs01/linkpair-v1.0-cs01.html)
  * [AMQP Request-Response Messaging with Link Pairing Version 1.0](https://docs.oasis-open.org/amqp/linkpair/v1.0/cs01/linkpair-v1.0-cs01.html) 
  * [AMQP Claims-based Security Version 1.0](https://docs.oasis-open.org/amqp/amqp-cbs/v1.0/csd01/amqp-cbs-v1.0-csd01.html)
  * [Event Stream Extensions for AMQP Version 1.0](https://docs.oasis-open.org/amqp/event-streams/v1.0/csd01/event-streams-v1.0-csd01.html)
* [Xin Chen's Awesome AMQP resource list](https://github.com/xinchen10/awesome-amqp/blob/master/README.md) which includes links to AMQP stacks and docs.
* [My 6-Part video series](https://www.youtube.com/watch?v=ODpeIdUdClc&list=PLmE4bZU0qx-wAP02i0I7PJWvDWoCytEjD) explaining AMQP

### OASIS MQTT 

* [MQTT 3.1.1](https://docs.oasis-open.org/mqtt/mqtt/v3.1.1/mqtt-v3.1.1.html)
* [MQTT 5.0](https://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html)
* [MQTT Project Site](https://mqtt.org/) including a resource directory


## Products & Cloud Services

### Microsoft Azure
* [Azure Messaging Overview](https://azure.microsoft.com/en-us/solutions/messaging-services/)
* [Azure Service Bus](https://azure.microsoft.com/en-us/services/service-bus)
   * [Federation](https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-federation-overview)
* [Azure Event Hubs](https://azure.microsoft.com/en-us/services/event-hubs/)
   * [Transfers, locks, and settlement](https://docs.microsoft.com/en-us/azure/service-bus-messaging/message-transfers-locks-settlement) 
   * [Sessions](https://docs.microsoft.com/en-us/azure/service-bus-messaging/message-sessions)
   * [Federation](https://docs.microsoft.com/en-us/azure/event-hubs/event-hubs-federation-overview)  
* [Azure Event Grid](https://azure.microsoft.com/en-us/services/event-grid/)
* [Azure Relay](https://docs.microsoft.com/en-us/azure/azure-relay/relay-what-is-it)
* [Azure IoT Hub](https://azure.microsoft.com/en-us/services/iot-hub/)

### Apache 

* [ActiveMQ](https://activemq.apache.org/)
* [Qpid](https://qpid.apache.org/)
* [Pulsar](https://pulsar.apache.org/)
* [Kafka](https://kafka.apache.org/)

### CNCF

* [NATS](https://www.cncf.io/projects/nats/)
* [Pravega](https://cncf.pravega.io/)

### Eclipse

* [Mosquitto](https://mosquitto.org/)
* [Paho](https://www.eclipse.org/paho/)
* [Jakarta Messaging (JMS)](https://projects.eclipse.org/projects/ee4j.messaging)