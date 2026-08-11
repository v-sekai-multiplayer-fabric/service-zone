# fabric-zone-domain

One zone, as one deployable thing. A **domain** is a packing of planes and edge planes, and a
ring forces co-location, so this is the packing that shares a ring.

## What is in it, and why these

The membership is derived rather than chosen. iceoryx2 is shared memory, so anything that
exchanges per-tick data is on one machine. Follow that and the list falls out.

| member | why it is here |
| --- | --- |
| `fabric-weft-plane` | the BEAM reaches the data plane **through a NIF**, which is in process, so it is here by necessity |
| the data plane | the ring itself, a seqlock ring in `native/dataplane` |
| `fabric-crowd-plane` | reads and writes entity state per tick, over iceoryx2 |
| `fabric-ingest-edge` | decodes player input datagrams and gives them to the data plane over iceoryx2 |
| `fabric-gateway-edge` | decodes control streams and gives them to the control plane over iceoryx2 |
| `fabric-tool-plane` | called from the ring, and a tool call carries a mesh or a stage worth not copying |
| `fabric-janet-plane` | samples the ring at its own rate, and never sits in the per-packet path |

**Nothing here is a copy.** Each member is its own repository and its own image. This
repository holds the manifest that runs them together, which is what a domain is.

## What is not in it

`fabric-store-domain`, because the store tolerates one FoundationDB round trip, about 1 ms,
and therefore needs no ring. `fabric-behaviour-domain`, because a plan changes in seconds and
ARDY wants a GPU, and neither is a reason to make every zone cost one.

That is the whole test. A member is here because it shares a ring, and everything else is free
to live elsewhere.

## A domain is not a machine

A machine is where a domain happens to run today. A domain is the set of processes that have
to be together, which is a property of the data flow rather than of the platform. Fly runs it
now, and the packing would be the same anywhere.

## Ports

None between members. They reach each other over iceoryx2, which is shared memory and has no
port. The only listening sockets belong to the two edge planes, because an edge plane is a
plane with networking.

## State

**Not built.** This holds the packing. The quadlet units and the Fly machine definition come
next.
