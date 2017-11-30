---
title: "BGP"
date: 2017-11-30T20:16:19+13:00
draft: false
categories: [ cisco, routing ]
tags: [ bgp ]
---

## Basics
* Distance vector protocol.
* Uses autonomous systems numbers (ASNs).
* Uses for connecting to external autonomous systems.
* eBGP and iBGP.
  * eBGP is where two routers are in different ASs.
  * iBGP is where two routers are in the sam AS.
* Runs over tcp/179.
* Neighbours (also known as peers) are explicitly defined.
  * TCP keepalives used for maintaining BGP adjacencies.
* Routes must be in the route table to be placed into the BGP table.
* Full updates with BGP peers when forming an adjacency.
  * After that BGP updates are sent.
* Holds a tonne of information about routes.
* eBGP always lists the advertising router as the `next-hop` for the destination.
* iBGP does not change the `next-hop` attribute when advertising the route to iBGP speakers.
  * `next-hop-self` can be used on iBGP peering to force the iBGP speaker to change the `next-hop` attribute.
* Selects a single route as the best-path.
* Only advertises its best route.

## Data structures
* Neighbor table
* BGP table.
* Route table.

## 2-byes AS number range
* 0 cannot be used.
* 1 - 64511 is the public ASN range.
* 64512 - 65535 is the private ASN range.

## 4-byte AS number range
[Can't get to sleep?](https://www.cisco.com/c/en/us/products/collateral/ios-nx-os-software/border-gateway-protocol-bgp/white_paper_c11_516829.html)

## router-id
* Same as EIGRP and OSPF.
* `bgp router-id` route process command can be used.
  * This resets any current adjancencies.
* *__Must__* be unique.

## Cisco recommendations
* Use loopbacks for peering.
  * eBGP peering via loopbacks requires the `ebgp-multihop` command to be used on the `neighbor` statement.
  * `update-source loopback` command is also required on the `neighbor` statement.
  * IGP routing to the peering is required.

### When to use it
* When you are connecting to more that one AS or ISP.
* Routing policy of your company and ISP differs.
* You are an ISP.

### When not to use it.
* Single ISP connection.
* When you don't care about the path taken to a destination.
* When router resources are limited.
* Low bandwidth connection between ASs.

## BGP states
* __Idle__: Initial state.  Should never be stuck here.
  * Ensure IP in `neighbor` statement is correct.
  * Connect follows Idle.
* __Connect__: TCP connection request sent but not yet replied to.
  * If the router receives no reply it goes to Active.
  * OpenSent follows Connect.
* __Active__: BGP is continuing to create the peering with the configure neighbour.
* __OpenSent__: TCP request has been replied to.
  * OpenConfirm follows OpenSent
* __OpenConfirm__: BGP speaker is waiting for a keepalive.
  * If a keepalive is received the sessions moves to Established.
  * If not it moves to Active.
* __Established__: Update packets are exchanged.


## BGP Rules
* The `network` route process command requires a route matching exactly be in the routing table.
  * Excluding the `mask` in the `network` command results in CIDR advertisements.

### iBGP rules
* Only advertise routes to an iBGP peer if that route was learned from an eBGP peer.
* __BGP Split-horizon__:
  * iBGP speakers cannot advertises a route learned from one iBGP peer to other iBGP peers.
  * Full mesh is required and assumed.
* Route relectors can be used to reduce the nx(n-1)/2 expotential growth issue for full-mesh networks.
* iBGP will advertise its locally sourced routes to iBGP peers.
* There must be a route in the IGP (and subsequently route table) to the next-hop address.

### Synchronisation
* Only applies when the AS is a transit area and there an non-BGP speakers in the transit area.
* A transit AS will not advertise a route until all routers in that transit AS has the same route in its IGP routing table.
  * This prevents blackholing of packets.
* Switched off by default as of IOS 12.2(8).
* `no-synch` switches synchronisation off.

## Route Reflectors
* A route reflector can take an iBGP learned route and advertise it a other iBGP peers.
* `clients` peer with route-reflectors.
* Route reflectors should be clients or each other as `non-clients`. *__(Need to validate this statement, likely wrong!!)__*

### Reflector rules
* Updates from RR clients a sent to all client and non-client peers.
* Updates from eBGP peers are sent to all client and non-client peers.
* Updates from non-client peers are sent to all clients.
  * other non-client peers are not updated, this is normal BGP split-horizon.


### eBGP peers
* Cisco recommends that eBGP peers be directly connected.

## Best path selection
[How the Best Path Algorithm Works](https://www.cisco.com/c/en/us/support/docs/ip/border-gateway-protocol-bgp/13753-25.html#anc2)

1. Highest weight.
2. Highest LOCAL_PREF.
3. Originated locally.
4. Shortest AS_PATH.
5. Lowest origin.  (i, then e, then ?)
6. Lowest MED.
7. eBGP path over iBGP path.
8. Lowest IGP metric to the BGP next-hop.
9. Determine if multiple paths require installation into the route table for BGP multipath.
10. If both paths are external, prefer the oldest one.
11. Prefer the path that comes from the BGP speaker with the lowest router-id.
12. If the originator or router-id is the same, prefer the path with the minimum cluster list length.
13. Prefer the path that comes from the lowest neighbor address.



## Configure
### eBGP neighbor
`ebgp-multihop` is required when peering using loopback interfaces, otherwise it is not required.

```
router(config)# router bgp <as>
router(config-router)# neighbor <address> remote-as <as>
router(config-router)# neighbor <address> update-source loopback
router(config-router)# neighbor <address> ebgp-multihop 2
```

### iBGP neighbor
`ebgp-multihop` is *__never__* required for iBGP.

```
router(config)# router bgp <as>
router(config-router)# neighbor <address> remote-as <as>
router(config-router)# neighbor <address> update-source loopback
```

### Advertise routes
```
router(config)# router bgp <as>
router(config-router)# network <address> mask <netmask>
```

or, route-map is optional.

```
router(config)# router bgp <as>
router(config-router)# redistribute <protocol> [route-map] <name>
```

### Set iBGP next-hop-self
Force iBGP to set the `next-hop` attribute to the local routers advertising interface.

```
router(config)# router bgp <as>
router(config-router)# neighbor <address> remote-as <as>
router(config-router)# neighbor <address> next-hop-self
```

### Set MED or LOCAL_PREF using route-map.
* Sets the metric value on route advertisements.
* Allows the use of prefix-list or access-list for filtering.
* Use `out` for MED.
* Use `in` for LOCAL_PREF.

```
router(config)# access-list <1-199> <and-so-on>
!
router(config)# route-map <name> permit <seq>
router(config-route-map)# match ip address <acl>
router(config-route-map)# set (<metric>|<local-preference>) <value>
!
router(config)# router bgp <as>
router(config-router)# neighbor <address> remote-as <as>
router(config-router)# neighbor <address> route-map <name> (out|in)
```

### Set LOCAL_PREF, all routes, no filtering possible.
```
router(config)# router bgp <as>
router(config-router)# bgp default local-preference <value>
```

### Set weight
#### Weight for aa neighbor filtering for specific prefixes
```
router(config)# access-list <1-199> <and-so-on>
!
router(config)# route-map <name> permit <seq>
router(config-route-map)# match ip address <acl>
router(config-route-map)# set weight <value>
!
router(config)# router bgp <as>
router(config-router)# neighbor <address> remote-as <as>
router(config-router)# neighbor <address> route-map <name> in
```

#### Weight for all routes from a neighbor
```
router(config)# router bgp <as>
router(config-router)# neighbor <address> weight <value>
```

### Route Reflectors
#### Clients
```
router(config)# router bgp <as>
router(config-router)# neighbor <route-reflector-address> remote-as <as>
```

#### Route Reflector
* Adjacencies will go down as this command is entered.
* `show ip bgp neighbor <address>` will show that neighbor is a client.

```
router(config)# router bgp <as>
router(config-router)# neighbor <client-address> route-reflector-client
```

## Verify
### BGP neighbours (filtered)
```
R1#show ip bgp neighbors | i (remote AS|router ID|BGP state|Last read)       
BGP neighbor is 209.65.200.226,  remote AS 65002, external link
  BGP version 4, remote router ID 209.65.200.226
  BGP state = Established, up for 04:02:51
  Last read 00:00:36, last write 00:00:55, hold time is 180, keepalive interval is 60 seconds
```

### BGP Summary
```
R1#show ip bgp summary 
BGP router identifier 209.65.200.225, local AS number 65001
BGP table version is 1, main routing table version 1

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
209.65.200.226  4        65002     260     260        1    0    0 03:52:17        0
```

### BGP table
```
R1#show ip bgp 
BGP table version is 3, local router ID is 209.65.200.225
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 r>  209.65.200.224/30
                       209.65.200.226           0             0 65002 ?
 *>  209.65.200.240/29
                       209.65.200.226           0             0 65002 ?
```

### BGP routes
```
R1#show ip route bgp
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 209.65.200.226 to network 0.0.0.0

      209.65.200.0/24 is variably subnetted, 3 subnets, 3 masks
B        209.65.200.240/29 [20/0] via 209.65.200.226, 00:00:23
```