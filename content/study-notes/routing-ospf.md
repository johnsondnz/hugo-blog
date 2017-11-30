---
title: "OSPF"
date: 2017-11-29T16:35:19+13:00
draft: false
categories: [ cisco, routing ]
tags: [ ospf ]
---

## OSPF
* IP protocol 89.
* Link state protocol.
* Uses dijkstra algorithm.
* Multi-area and single area topology options.

## Data structures (tables)
* Topology database or link-state database.
* Neighbour table or adjacency table.
* Routing table or forwarding table.

## What needs to match between OSPF routers?
* Interface netmask except on point-to-point links.
* Hello and dead timers.
* Area.
* Stub flag.
* Authentication password.

*__Notes__*:

* Unique RID is required for each router in the the OSPF domain.
* MTU should match too, else you risk getting stuck in EXSTART due to one neighbour maximising MTU to a point that one neighbour cannot accept the frames due to them being too large.

## OSPF Networks
* __Point-to-point__: Network that joins a pair of routers.
  * No DR or BDR is elected.
  * All updates sent to 224.0.0.5.
* __Broadcast__:
  * DR and BDR elected.
  * Routers flood Type-1 LSAs to DR and DBR only on 224.0.0.6.
  * DR floods Type-2 LSAs to all OSPF routers by DR on 224.0.0.5.
* [Nonbroadcast multiacces (NBMA)](/posts/routing-ospf-nbma/)

## DR and BDR election
* Only performed on multiacces networks such as broadcast and NBMA is special configurations.
* Criteria:
  * Highest OSPF interface priority wins and is the DR.
  * Second highest OSPF interface priority is the BDR.
  * IN the event all priorities are the same highest RID is the tie-breaker.
* Router with an OSPF interface priority of 0 cannot be a DR or BDR.
* Preempt does not occur.  Only way for a DR or BDR to the superceeded is through a loss of that routers OSPF process, i.e. `clear ip ospf process <num>`, `clear ip ospf neighbor <ip>` etc.

## Timers
* Hello and dead timers must match between OSPF routers.
* Dead timer is 4x hello.
* Hello-interval: Per-network type.
  * Point-to-point: 10 seconds
  * Broadcast: 10 seconds
  * Nonbroadcast mutliaccess: 30 seconds
* LSA update on Cisco is 30 minutes, other vendors vary.

## Network command
* `network <network> <wildcard mask> area <area>` route process command.
* The above command is used to select interface matching this network and mask for entry into the OSPF process. 
* I prefer `ip ospf <process> area <area>` interface command.  This more explicit and avoids wildcard mask traps.

### Hub and Spoke
* `neighbor <ip>` route process command is required to start unicast OSPF neighbor process.
  * Only required on the hub in hub-n-spoke networks.
  * Doesn't hurt to place this on the spokes as well.
* Hub __must__ be the DR and there can be no BDR.
  * All spoke interfaces __must__ be configured with `ip ospf priority 0` interface command.
  * Adjacencies will come up if this isn't done, however it is inefficient to have a spoke as the DR.  If the hub goes down, then a BDR also serves no purpose as all inter-site communications must go via the hub.

## Passive interface
Useful for when an interface has no potential neighbours but you want the network assoicated with the interface injected into OSPF.

Passive interfaces do not send or process received OSPF packets.

## Configure
### Network type
* broadcast: Cisco mode.
* nonbroadcast: RFC-compliant.
* point-to-multipoint: RFC-compliant.
* point-to-multipoint nonbroadcast: Cisco mode
* point-to-point: Cisco Mode

```
router(config)# interface <iface>
router(config-if)# ip ospf network <type>
```
### Hello interval
Changing the `hello-interval` interface command also changes the dead timer interval.

```
router(config#) interface <iface>
router(config-if)# ip ospf hello-interval <sec>
```

### OSPF interface priority
Effects the DR and BDR election.

```
router(config)# interface <iface>
router(config-if)# ip ospf priroity <0-255>
```

### Add interface to routing process
```
router(config)# router ospf <process>
router(config-router)# network <network> <wildcard mask> area <area>
```

or

```
router(config)# interface <iface>
router(config-if)# ip ospf <process> area <area>
```

### Passive Interface
If using the `default` knob be sure to issue `no passive-interface <iface>` on interfaces where you expect neighbours to form.

```
router(config)# router ospf <process>
router(config-router)# passive-interface <iface>
```

or

```
router(config)# interface <iface>
router(config-if)# ip ospf passive-interface
```

### Advertise default route
A default route must exist in the routers route table.

```
router(config)# router ospf <process>
router(config-router)# default information-originate
```

or always, a default route is advertise regardless of the presence of a default route.

```
router(config)# router ospf <process>
router(config-router)# default information-originate always
```

## Verify
### Interfaces
```
R3#show ip ospf interface 
Serial1/0.123 is up, line protocol is up 
  Internet Address 10.1.1.6/30, Area 0, Attached via Interface Enable
  Process ID 1, Router ID 10.1.1.3, Network Type POINT_TO_POINT, Cost: 64
  Topology-MTID    Cost    Disabled    Shutdown      Topology Name
        0           64        no          no            Base
  Enabled by interface config, including secondary ip addresses
  Transmit Delay is 1 sec, State POINT_TO_POINT
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
    oob-resync timeout 40
    Hello due in 00:00:03
  Supports Link-local Signaling (LLS)
  Cisco NSF helper support enabled
  IETF NSF helper support enabled
  Index 1/1/1, flood queue length 0
  Next 0x0(0)/0x0(0)/0x0(0)
  Last flood scan length is 7, maximum is 7
  Last flood scan time is 0 msec, maximum is 0 msec
  Neighbor Count is 1, Adjacent neighbor count is 1 
    Adjacent with neighbor 10.1.1.2
  Suppress hello for 0 neighbor(s)
Serial1/0.134 is up, line protocol is up 
  Internet Address 10.1.1.9/30, Area 34, Attached via Interface Enable
  Process ID 1, Router ID 10.1.1.3, Network Type POINT_TO_POINT, Cost: 64
  Topology-MTID    Cost    Disabled    Shutdown      Topology Name
        0           64        no          no            Base
  Enabled by interface config, including secondary ip addresses
  Transmit Delay is 1 sec, State POINT_TO_POINT
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
    oob-resync timeout 40
    Hello due in 00:00:06
  Supports Link-local Signaling (LLS)
  Cisco NSF helper support enabled
  IETF NSF helper support enabled
  Index 1/1/2, flood queue length 0
  Next 0x0(0)/0x0(0)/0x0(0)
  Last flood scan length is 2, maximum is 2
  Last flood scan time is 0 msec, maximum is 1 msec
  Neighbor Count is 1, Adjacent neighbor count is 1 
    Adjacent with neighbor 10.1.1.4
  Suppress hello for 0 neighbor(s)
```

or filtered

```
R3#show ip ospf interface | i (line|Address|Process|Timer)
Serial1/0.123 is up, line protocol is up 
  Internet Address 10.1.1.6/30, Area 0, Attached via Interface Enable
  Process ID 1, Router ID 10.1.1.3, Network Type POINT_TO_POINT, Cost: 64
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
Serial1/0.134 is up, line protocol is up 
  Internet Address 10.1.1.9/30, Area 34, Attached via Interface Enable
  Process ID 1, Router ID 10.1.1.3, Network Type POINT_TO_POINT, Cost: 64
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
```

### Neighbours
```
R3#show ip ospf neighbor 

Neighbor ID     Pri   State           Dead Time   Address         Interface
10.1.1.2          0   FULL/  -        00:00:37    10.1.1.5        Serial1/0.123
10.1.1.4          0   FULL/  -        00:00:37    10.1.1.10       Serial1/0.134
```

Specific neighbor information.
```
R3#show ip ospf neighbor 10.1.1.2
 Neighbor 10.1.1.2, interface address 10.1.1.5
    In the area 0 via interface Serial1/0.123
    Neighbor priority is 0, State is FULL, 6 state changes
    DR is 0.0.0.0 BDR is 0.0.0.0
    Options is 0x12 in Hello (E-bit, L-bit)
    Options is 0x52 in DBD (E-bit, L-bit, O-bit)
    LLS Options is 0x1 (LR)
    Dead timer due in 00:00:39
    Neighbor is up for 1d09h   
    Index 1/1/2, retransmission queue length 0, number of retransmission 0
    First 0x0(0)/0x0(0)/0x0(0) Next 0x0(0)/0x0(0)/0x0(0)
    Last retransmission scan length is 0, maximum is 0
    Last retransmission scan time is 0 msec, maximum is 0 msec
```

### Database
```
R3#show ip ospf database 

            OSPF Router with ID (10.1.1.3) (Process ID 1)

		Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
10.1.1.2        10.1.1.2        735         0x8000003F 0x00DE3D 2
10.1.1.3        10.1.1.3        228         0x80000042 0x00D242 2

		Summary Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.1.1.0        10.1.1.2        735         0x8000003C 0x007436
10.1.1.8        10.1.1.3        228         0x80000036 0x002A7D

		Summary ASB Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.1.1.1        10.1.1.2        217         0x8000003A 0x007235

		Router Link States (Area 34)

Link ID         ADV Router      Age         Seq#       Checksum Link count
10.1.1.3        10.1.1.3        228         0x80000044 0x000BF8 2
10.1.1.4        10.1.1.4        410         0x80000038 0x0014FA 2

		Summary Net Link States (Area 34)

Link ID         ADV Router      Age         Seq#       Checksum
0.0.0.0         10.1.1.3        228         0x80000036 0x00539E

		Type-7 AS External Link States (Area 34)

Link ID         ADV Router      Age         Seq#       Checksum Tag
10.1.4.4        10.1.1.4        410         0x80000037 0x0084A5 0
10.1.4.8        10.1.1.4        410         0x80000037 0x005CC9 0
10.2.1.0        10.1.1.4        410         0x80000036 0x00D558 0
10.2.2.0        10.1.1.4        410         0x80000036 0x00CA62 0
10.2.4.12       10.1.1.4        167         0x80000036 0x002AF7 0

		Type-5 AS External Link States

Link ID         ADV Router      Age         Seq#       Checksum Tag
0.0.0.0         10.1.1.1        1323        0x80000034 0x00650D 1
10.1.4.4        10.1.1.3        228         0x80000036 0x002115 0
10.1.4.8        10.1.1.3        228         0x80000036 0x00F839 0
10.2.1.0        10.1.1.3        228         0x80000036 0x0070C8 0
10.2.2.0        10.1.1.3        228         0x80000036 0x0065D2 0
10.2.4.12       10.1.1.3        228         0x80000036 0x00C468
```

### Routes
```
R3#show ip route ospf
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 10.1.1.5 to network 0.0.0.0

O*E2  0.0.0.0/0 [110/1] via 10.1.1.5, 1d04h, Serial1/0.123
      10.0.0.0/8 is variably subnetted, 10 subnets, 3 masks
O IA     10.1.1.0/30 [110/128] via 10.1.1.5, 1d05h, Serial1/0.123
O N2     10.1.4.4/30 [110/20] via 10.1.1.10, 1d05h, Serial1/0.134
O N2     10.1.4.8/30 [110/20] via 10.1.1.10, 1d05h, Serial1/0.134
O N2     10.2.1.0/24 [110/20] via 10.1.1.10, 1d05h, Serial1/0.134
O N2     10.2.2.0/24 [110/20] via 10.1.1.10, 1d05h, Serial1/0.134
O N2     10.2.4.12/30 [110/20] via 10.1.1.10, 1d05h, Serial1/0.134
```


## Debug
### Debug the OSPF adjacency
* `router# ddebugebig ip ospf hello`
  * To see the hello packets sent and received.
  * The message output can be cryptic.
    * R = received
    * C = configured
* `router# debug ip ospf adj`


