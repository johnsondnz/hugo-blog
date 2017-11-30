---
title: "EIGRP"
date: 2017-11-30T09:50:19+13:00
draft: false
categories: [ cisco, routing ]
tags: [ eigrp, ipv6 ]
---

## Features
* IP protocol 88.
* Used to be Cisco propetary.  Now open [RFC7868](https://tools.ietf.org/html/rfc7868).
* Hybrid protocol, more like a link-state protocol than a vector distance protocol.
* Multiprotocol support including IPv4, IPv6 etc.
* Support for CIDR and VLSM.
* Diffusing update algorithm (DUAL).
* Complex metric based on:
  * Bandwidth, on by default.
  * Delay, on by default.
  * Load, off by default.
  * Reliability, off by default.
* Accepts CIDR and VLSM wildcard in `network` route process command.
* Load-balacing support.
  * Defaults to 4.
  * `maximum-paths` route process command can tunes this.
  * Equal-cost load-balancing on by default.
  * Support for unequal-cost load sharing via the `variance <multiplier>` route process command.
* RTO calculated per-neighbour to determine time to wait before sending unicast packets for reliable update and query processing.
* Reliable delivery for some packets.
* Reliable packets are sent via RTP (reliable transport protocol).
  * update, query and reply packets
* Uses split-horizon which can pose problems on point-to-multipoing links.
* `router-id` defaults to highest loopback IP or in absense of a loopback interface the highest IP of a physical interface.
* Stub support.


## Data structures
* Neighbour table: Neighbours.
* Topology table: Known loop-free paths to each destination.
* Route table: Best successor route(s) are placed here.


## Packet Types
* Hello.
  * Sent to 224.0.0.10 multicast address.
* Update.
  * Sent when there is a change to 224.0.0.10.
  * Ack expected from every neighbour.
  * Unrepsonsive neighbours are sent the update via unicast up to 16 times.  If there is no further response the neighbour is declared dead.
  * Uses RTP.
  * After inital neighbour discovery full topology exchange occurs via unicast.
* Ack.
  * Acknowledgement of a received update.
  * Unicast
* Query.
  * Sent to 224.0.0.10 when a route to a destination is lost.
  * Expects a response from all neighbours.
  * If a neighbour does not respond up to 16 unicast queries are sent before it gives up.
  * Uses RTP.
* Reply.
  * Unicast repsonse to a query.
  * Indicate the alternative path or absense of one.
  * Uses RTP.


## Required to form neighbour
* Same AS.
* Hello packet exchange.
* Same K values.
  * K1: Bandwidth, on by default.
  * K2: Load, off by default.
  * K3: Delay, on by default.
  * K4: Reliability, off by default.
  * K5: MTU, off by default.

__Notes__:

* Timers don't need to match but it is recommended to prevent flapping neghbours.  This prevents one router's hold-time (three missed hellos) expiring before one router sends its periodic hello.


## Timers
* Hello:
  * T1 or slower: 60 seconds.
  * Broadcast: 5 seconds.
  * Configurable with `ip hello-interval eigrp <as> <seconds>` interface command.
* Hold time: 3 x the hello.
  * Configurable with `ip hold-time eigrp <as> <seconds>` interface command.

## Passive interface
* Same as OSPF, uses the `passive-interface <iface>` route process command.
* Passive interface prefix can be in the EIGRP topology but the interface not actively sending or processing EIGRP hello packets.

## Advertised Distance (AD) vs Feasible Distance (FD)
* Advertised distance is also known as reported distance.
* Advertised distance is the distance from the next-hop router to a particular desitnation prefix.
* Feasible distance is the metric for the local router to reach a particular destination.
  * It is the cost to reach the next-hop route plus the advertise distance for that route.

### Successor
* The route with the lowest FD is placed into the routing table.  This is called the successor and is the best path to the destination.

### Feasible successor
* Routes with an AD less than that of the FD are placed into the topology table and is called a feasible successor.
  * This is the sum of the AD plus the metric to reach the next-hop router advertising that AD.
* For a feasible successor route to be deemed loop free the __AD for a route must be less than the FD__ for the route.


### Unequal-cost load-sharing
* By default variance is set to 1 which is equal-cost load-balancing only.
* FD of successor routes are multiplied by the vaiance multipler.
* Any feasible successor whos FD is less than the resulting successor FD is used for load-sharing.
* Metrics are not changed in any way.
* Command is locally significant only.
* Some IOS versions require you to clear the eigrp routes out of the route table.
* All of nothing, so this will impact all routes on the local router.

#### Traffic share
* `traffic-share balanced` route process command.
  * Balances the traffic proportionally to the ratios of the metrics associated with the different routes.
  * Default behaviour.
* `traffic-share min across-interfaces` route process command.
* CEF and fast-switched processing will use per-destination load-balancing.
* Process-switched packets are per-packet load-balanced.
* Load-balacing is performed only on packet that pass through the router.  This will thereore exclude router generated packets such as ICMP.

## Split-horizon on p2mp interfaces
* Split-horizon rules state that "a route from being advertised out an interface to which it was learned".
* Solution is to turn of split-horizon.
  * `no ip split-horizon eigrp <as>` interface command can be used.  __Be very careful__.
  * Later IOS versions resync the neighbour relationship.
  * Older IOS versions drop and reestablish the neighbour relationship.
  * Risk of routing loops.  FD/AD EIGRP rules should prevent this.
* __Another, better solution is to simply use sub-interfaces__.

## Passive vs Active Routes
* Passive is good.  There is no active query for the destination.
* Active is bad.  The route is missing from the local router and it is actively querying for a alternative path.

### Stuck in Active (SIA)
* Queries are propogarted throughout the entire EIGRP AS.
* Normally routes go from Active to Passive or gone very quickly.

## Stub routing
* Stubs are never queried for alternative paths.
* Limits the route table required on stubs as there are no alternative paths out of a stub.
  * Only a default route is advertised to stub routers.
* Often done in hub-n-spoke topologies.
* EIGRP stub routers advertise two types of routes:
  * Directly connected.
  * Summary routes.
* Stub routes can also advertise static routes via redistribtion.  This is off by default.
* Later versions of IOS allow for the advertisement of dynamic prefixes via leak-maps.
* Only one router in a given adjacency can be a stub.  Two stub routers cannot for an adjacency.


## Redistribution
When redistributing routes you ned to set a seed metric either per redistribution command on with the `default-metric` route process command.

## Configure
### Manuall configure router-id
```
router(config)# router eigrp <as>
router(condif-router)# router-id <router-id>
```

### Place interface into EIGRP
```
router(config)# router eigrp <as>
router(config)# no auto-summary
router(config-router)# network <address> <wildcard mask>
```

### Timers
```
router(config)# interface <iface> 
router(config-if)# ip hello-interval eigrp <as> <seconds> 
router(config-if)# ip hold-time eigrp <as> <seconds>
```

### Maximum-paths
```
router(config)# router eigrp <as>
router(config-router)# maximum-paths <1-16>
```

### Unequal-cost load-balancing
```
router(config)# router eigrp <as>
router(config-router)# variance <1-128>
```

### Stub routing
```
router(config)# router eigrp <as>
router(config-router)# eigrp stub
```

```
R4(config-router)#eigrp stub ?
  connected      Do advertise connected routes
  leak-map       Allow dynamic prefixes based on the leak-map
  receive-only   Set receive only neighbor
  redistributed  Do advertise redistributed routes
  static         Do advertise static routes
  summary        Do advertise summary routes
  <cr>
```

## Propogate default route
### Redistribute static route
Creates a D*EX with an AD of 170 on downstream routers.

```
router(config)# 0.0.0.0 0.0.0.0 (<next-hop-address>|iface>)
router(config)# router eigrp <as>
router(config-router)# redistribute static 100000 10 255 1 1500
```

### ip default-network
### Redistribute static route
The router advertising via this command first needs to the network be in the EIGRP topology table and route table.

This create a D* route with an AD of 90.

```
router(config)# if default-network <network>
```

## Redistribute something
```
router(config)# router eigrp <as>
routerconfig-router)# redistribute <protocol> <speed> <delay> <reliability> <load> <mtu>
```

or

```
router(config)# router eigrp <as>
routerconfig-router)# default-metric <speed> <delay> <reliability> <load> <mtu>
routerconfig-router)# redistribute <protocol>
```

## Configuring EIGRP for IPv6
Easy as!!

```
router(config)# ipv6 router eigrp <as>
router(config-rtr)# router-id <address>
router(config-rtr)# interface <iface>
router(config-if)# ipv6 eigrp <as>
```

## Verify
### Topology
* FD is the first part of the metric before the backslash.
* AD is the second part of the metric after the backslash.
* P = Passive.
* A = Active

```
R4#show ip eigrp topology 
EIGRP-IPv4 Topology Table for AS(10)/ID(10.1.4.9)
Codes: P - Passive, A - Active, U - Update, Q - Query, R - Reply,
       r - reply Status, s - sia Status 

P 10.1.4.4/30, 1 successors, FD is 281600
        via Connected, Ethernet0/0
        via 10.1.4.10 (282112/3072), Ethernet0/1
P 10.2.2.0/24, 2 successors, FD is 281856
        via 10.1.4.6 (281856/2816), Ethernet0/0
        via 10.1.4.10 (281856/2816), Ethernet0/1
P 10.1.1.8/30, 1 successors, FD is 28160
        via Redistributed (28160/0)
P 0.0.0.0/0, 1 successors, FD is 28160
        via Redistributed (28160/0)
P 10.2.1.0/24, 2 successors, FD is 281856
        via 10.1.4.6 (281856/2816), Ethernet0/0
        via 10.1.4.10 (281856/2816), Ethernet0/1
P 10.2.4.12/30, 1 successors, FD is 281856
        via 10.1.4.6 (281856/2816), Ethernet0/0
P 10.1.4.8/30, 1 successors, FD is 281600
        via Connected, Ethernet0/1
        via 10.1.4.6 (282112/3072), Ethernet0/0
```

### Neighbours
```
R4#show ip eigrp neighbors 
EIGRP-IPv4 Neighbors for AS(10)
H   Address                 Interface              Hold Uptime   SRTT   RTO  Q  Seq
                                                   (sec)         (ms)       Cnt Num
1   10.1.4.6                Et0/0                    13 1d21h       1   100  0  78
0   10.1.4.10               Et0/1                    11 1d22h       1   100  0  106
```

### Interfaces
```
R4#show ip eigrp interfaces 
EIGRP-IPv4 Interfaces for AS(10)
                  Xmit Queue   PeerQ        Mean   Pacing Time   Multicast    Pending
Interface  Peers  Un/Reliable  Un/Reliable  SRTT   Un/Reliable   Flow Timer   Routes
Et0/0        1        0/0       0/0           1       0/2           50           0
Et0/1        1        0/0       0/0           1       0/2           50           0
```

### Interface timers
```
R4#show ip eigrp interfaces detail 
EIGRP-IPv4 Interfaces for AS(10)
                  Xmit Queue   PeerQ        Mean   Pacing Time   Multicast    Pending
Interface  Peers  Un/Reliable  Un/Reliable  SRTT   Un/Reliable   Flow Timer   Routes
Et0/0        1        0/0       0/0           1       0/2           50           0
  Hello-interval is 5, Hold-time is 15
  Split-horizon is enabled
  Next xmit serial <none>
  Packetized sent/expedited: 9/0
  Hello's sent/expedited: 36248/2
  Un/reliable mcasts: 0/4  Un/reliable ucasts: 31/7
  Mcast exceptions: 0  CR packets: 0  ACKs suppressed: 0
  Retransmissions sent: 1  Out-of-sequence rcvd: 0
  Topology-ids on interface - 0 
  Authentication mode is not set
Et0/1        1        0/0       0/0           1       0/2           50           0
  Hello-interval is 5, Hold-time is 15
  Split-horizon is enabled
  Next xmit serial <none>
  Packetized sent/expedited: 10/0
  Hello's sent/expedited: 36244/2
  Un/reliable mcasts: 0/6  Un/reliable ucasts: 30/7
  Mcast exceptions: 0  CR packets: 0  ACKs suppressed: 0
  Retransmissions sent: 1  Out-of-sequence rcvd: 1
  Topology-ids on interface - 0 
  Authentication mode is not set
```

and filtered

```
R4#show ip eigrp interfaces detail | i (Et|Ser|interval|Split|Xmit|Int)
EIGRP-IPv4 Interfaces for AS(10)
                  Xmit Queue   PeerQ        Mean   Pacing Time   Multicast    Pending
Interface  Peers  Un/Reliable  Un/Reliable  SRTT   Un/Reliable   Flow Timer   Routes
Et0/0        1        0/0       0/0           1       0/2           50           0
  Hello-interval is 5, Hold-time is 15
  Split-horizon is enabled
Et0/1        1        0/0       0/0           1       0/2           50           0
  Hello-interval is 5, Hold-time is 15
  Split-horizon is enabled

```

### Routes
```
R4#show ip route eigrp 
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 10.1.1.9 to network 0.0.0.0

      10.0.0.0/8 is variably subnetted, 9 subnets, 3 masks
D        10.2.1.0/24 [90/281856] via 10.1.4.10, 1d21h, Ethernet0/1
                     [90/281856] via 10.1.4.6, 1d21h, Ethernet0/0
D        10.2.2.0/24 [90/281856] via 10.1.4.10, 1d21h, Ethernet0/1
                     [90/281856] via 10.1.4.6, 1d21h, Ethernet0/0
D        10.2.4.12/30 [90/281856] via 10.1.4.6, 1d21h, Ethernet0/0
```

### Traffic Share
Look for the traffic share count.  This is the ratio of traffic that CEF will use to place traffic only this link.

```
R4#show ip route 10.2.1.0
Routing entry for 10.2.1.0/24
  Known via "eigrp 10", distance 90, metric 281856, type internal
  Redistributing via ospf 1, eigrp 10
  Advertised by ospf 1 subnets
  Last update from 10.1.4.6 on Ethernet0/0, 1d22h ago
  Routing Descriptor Blocks:
  * 10.1.4.10, from 10.1.4.10, 1d22h ago, via Ethernet0/1
      Route metric is 281856, traffic share count is 1 <<<<<<<<<<<<<<<<<<<<<<<< HERE
      Total delay is 1010 microseconds, minimum bandwidth is 10000 Kbit
      Reliability 255/255, minimum MTU 1500 bytes
      Loading 1/255, Hops 1
    10.1.4.6, from 10.1.4.6, 1d22h ago, via Ethernet0/0
      Route metric is 281856, traffic share count is 1 <<<<<<<<<<<<<<<<<<<<<<<< HERE
      Total delay is 1010 microseconds, minimum bandwidth is 10000 Kbit
      Reliability 255/255, minimum MTU 1500 bytes
      Loading 1/255, Hops 1
```

## Debug
* To see EIGRP packets use the `debug eigrp packets` command.

```
R4#debug eigrp packets ?
  <1-99>       Access list
  <1300-1999>  Access list (expanded range)
  SIAquery     EIGRP SIA-Query packets
  SIAreply     EIGRP SIA-Reply packets
  ack          EIGRP ack packets
  all          Display all EIGRP packets
  hello        EIGRP hello packets
  query        EIGRP query packets
  reply        EIGRP reply packets
  request      EIGRP request packets
  retry        EIGRP retransmissions
  stub         EIGRP stub packets
  terse        Display all EIGRP packets except Hellos
  update       EIGRP update packets
  <cr>
```

```
R4#debug ip eigrp ?
  <1-65535>      Autonomous System
  A.B.C.D        IPv4 Address
  neighbor       Neighbor debugging
  notifications  Event notifications
  summary        Summary Route Processing
  vrf            Select a VPN Routing/Forwarding instance
  <cr>

R4#debug eigrp ?  
  address-family  EIGRP address-family debug commands
  fsm             EIGRP Dual Finite State Machine events/actions
  neighbors       EIGRP neighbors
  nsf             EIGRP Non-Stop Forwarding events/actions
  packets         EIGRP packets
  service-family  EIGRP service-family debug commands
  timers          EIGRP Timer Events
  transmit        EIGRP transmission events
```

```
R4#show ip eigrp traffic 
EIGRP-IPv4 Traffic Statistics for AS(10)
  Hellos sent/received: 72429/71694
  Updates sent/received: 14/54
  Queries sent/received: 0/10
  Replies sent/received: 10/0
  Acks sent/received: 61/22
  SIA-Queries sent/received: 0/0
  SIA-Replies sent/received: 0/0
  Hello Process ID: 399
  PDM Process ID: 397
  Socket Queue: 0/10000/2/0 (current/max/highest/drops)
  Input Queue: 0/10000/2/0 (current/max/highest/drops)
```