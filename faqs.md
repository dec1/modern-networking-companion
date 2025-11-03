## Answers to some Frequently asked Questions (FAQs)

### Layer Structure & Encapsulation

#### <a name="layer-1-base-message"></a>Does layer 1 contain the base message that we want to send, with higher layers just adding detail?

No, it's actually the reverse. The _base message_ — the actual content you want to communicate — _lives_ at the _highest layer_. Each lower layer wraps this with _packaging_ needed to deliver it, at a lower lever of detail.

_Think_: postal system. The words on your letter are the base message (your data/payload). This letter goes inside an envelope (e.g. _packet_ of this higher layer) that adds the recipient's address and postage—information needed to deliver it. That envelope might go inside a box (which you can think of as a lower layer) in the postman's van (which you can think of as part of an even lower layer), which doesn't care what's written on your letter or even what's in the box—it just provides the physical transport mechanism (electromagnetic waves on cables, radio signals through air).

Layer 1 is the _outermost_, most physical layer. It converts whatever bits it receives into electromagnetic waves for transmission, completely unaware of what those bits represent. It doesn't understand or interpret your message—it just transports whatever bits it's given, like a delivery truck that carries any kind of sealed box without knowing or caring what's inside.

Your actual message lives at the _innermost_ layer, progressively wrapped by each lower layer's addressing and delivery information (_think_: nested envelopes). The higher (more abstract) the layer, the closer you get to the actual content you wanted to send.


#### <a name="russian-doll-structure"></a>Why are the higher layers deeper inside the Russian doll instead of outside?

So as to hide higher layer data from lower layer devices that wouldn't understand, and might be confused by it.  Each such device processes what's relevant to its job — everything else is just opaque (inner) payload. As mentioned in "Layer specific devices" — a device operating at a given layer is capable of operating at all lower layers. Such a higher layer device is familiar with the doll shell hierarchy down as far as its own "layer" and can purposefully unpack the outer shells to "get at" its data. Lower layer shells are "shielded" from this by the shells in between.

The "higher" (more abstract) layer goes inside because it's being protected and served. The "lower" (more physical) layer wraps around the outside to provide its delivery service.

---

### Switches & Layer 2

#### <a name="switch-unknown-destination"></a>Switch figures out which interfaces are connected based on source addresses. So how does switch know where to send packets if it hasn't received one from that destination yet?

It doesn't—so it _sends the packet out every port_ (a process called _flooding_). 

Here's what happens: The switch receives a Layer 2 packet with a specific _unicast destination MAC address_ (say, AA:BB:CC:DD:EE:FF). The switch checks its CAM table to see which port that MAC address is connected to. If there's no entry, the switch doesn't know which port leads to that interface.

So the switch forwards the _exact same packet_ (destination MAC address unchanged) out every port except the one it arrived on. Every connected interface receives the identical packet. Each interface examines the destination MAC address in the header and asks "is this my address?" Only the interface with MAC address AA:BB:CC:DD:EE:FF accepts and processes the packet. All other interfaces ignore and discard it.

This is inefficient—unnecessary copies go everywhere—but it's safe: the intended recipient will definitely get the packet.

Once that destination interface replies (sending its own packet back), the switch observes the _source address_ of the reply and learns "ah, MAC address AA:BB:CC:DD:EE:FF is connected to port 5." The switch adds this to its CAM table. Subsequent packets destined for AA:BB:CC:DD:EE:FF go directly to port 5.

_**Note**_: Flooding is different to _broadcasting_, where the sender intentionally uses the broadcast MAC address (FF:FF:FF:FF:FF:FF) to tell *all* interfaces to accept the packet. With flooding, the destination address used is that of one specific interface—the switch just doesn't know which port it's on yet.



#### <a name="switch-empty-cam"></a>How can using a switch (even with an empty CAM table) yield better performance than without a switch?

Two main reasons: **buffering** and **full-duplex coordination**. Even without knowing where interfaces are, the switch can queue incoming packets and send them sequentially, preventing simultaneous transmissions from colliding. It's like having a traffic controller who doesn't know which road goes where yet, but can still prevent cars from crashing into each other at an intersection by making them go one at a time. Without the switch, all devices transmit whenever they want, leading to frequent collisions.


#### <a name="lan-vs-layer2"></a>Is there a difference between a LAN and a layer 2 network?

No, they're synonyms. _LAN (Local Area Network)_ is just a more common name for a layer 2 network. 

Both are defined by their use of MAC addresses and the basic Layer 2 packet structure: a payload wrapped in a header (containing source and destination MAC addresses) and trailer (containing error checking). Different Layer 2 technologies like Ethernet and Wi-Fi share this fundamental structure. Their packets do differ in some details, however, — Wi-Fi packets include additional fields for handling the less reliable wireless medium. Despite these differences, both use MAC addresses and the same header-payload-trailer structure and semantics, which is what makes them both Layer 2 technologies suitable for building LANs.

Historically, Layer 2 networks were confined to physical proximity (e.g., Ethernet cables in a single building or campus), aligning perfectly with the "Local" in LAN. However, advancements like VXLAN (Virtual Extensible LAN) introduced overlay technologies that encapsulate Layer 2 frames over IP networks, enabling LANs to span arbitrary distances—potentially globally. This is why it's important to clarify that it's the protocol used - especially the packet structure and MAC addressing - that defines a LAN and not its _locality_. 

---
### Upper Layers and Applications

#### <a name="dns-ttl"></a>Does the TTL of a DNS record depend on the frequency it is being accessed?

No. The _TTL (Time To Live)_ is set by the domain owner/administrator when they configure their DNS records. It specifies how long other servers should cache the answer before checking again. High-traffic sites might use short TTLs (minutes) if they need flexibility to change IPs quickly, or long TTLs (hours/days) if their infrastructure is stable and they want to reduce DNS query load. The access frequency doesn't automatically change the TTL—it's a policy decision.


