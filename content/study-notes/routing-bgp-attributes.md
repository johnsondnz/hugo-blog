---
title: "BGP Attributes"
date: 2017-11-30T20:26:19+13:00
draft: false
categories: [ cisco, routing ]
tags: [ bgp ]
---

## Attribute Types
* __Well-known mandatory__: Must appear in all BGP updates.  BGP speakers must undertand these.
  * AS_PATH.
  * origin.
  * next-hop.
* __Well-known descrentionary__: Must be recognised by all BGP speakers but does not have to be included in every BGP udpdate.
  * LOCAL_PREF.
  * atomic aggregate.
* __Optional Transitive__:
  * aggregator.
  * community.
* __Optional non-transitive__:.
  * MED (multi-exit discriminator).
  * Cluster List

### Optional attributes
Optional attributes that are those that not every BGP speaker will understand.


__Optional Transitive__:

* The `partial bit` is set.
* The advertisement is accepted.
* BGP speakers that do not recognise these must include them in advertisements them to other BGP speakers.

__Optional non-transitive__:

* The `partial bit` is __not__ set.
* BGP speakers not understanding these will not include them in advertisements to other BGP speakers.

## Attribute Details
### Origin
Listed in order of preference in route selection.

* __i__: IGP.  The `network` command was used.
* __e__: EGP.  Legacy from the days of EGP.  Not normally seen today.
* __?__: incomplete.  Route was injected into BGP using redistribution.

### AS_PATH
* Shortest path win.
* Records the ASNs the route has gone through including the destination AS.
* __Helps prevent routing loops__.  If a router receives a route update an its own AS is in the AS_PATH it is rejected.
* AS_PATH with the letter `i` means the route was sourced locally.

### next-hop
* Needs to be reachable via an IGP for a route to be selected as best.
* If BGP cannot resolve the next-hop address is it deemed inaccessible.  The route is therefore invalidated.
  * To view this use `show ip bgp <address>` to see status.
* eBGP speakers set this to themselves.
* iBGP speakers do not change this attribute unless `next-hop-self` is configured on the `neighbor` route process command.

### Local preference
* Default is 100, default LOCAL_PREF values are hidden in the bgp table unless `show ip bgp <address>` is used.
  * If LOCAL_PREF is set to any value, including 100 it will appear in `show ip bgp` outputs.
* Highest wins as step two of the BGP best path selection process.
* Describes how to exit the local AS to an external AS.
  * For pure iBGP networks (like MPLS) it can also be used to advertise the best router to route towards for a specific destination, i,e. used for outbound route policy.

### Multi-exit discriminator (MED)
* Influences a remote AS on how to route to a local AS.
* Can be ignored and may not have an impact as it is step six in the best path selection process.
  * It is also optional non-transitive.  So may not get recognised.
* My preference is to use `AS_PATH` prepend to have the same impact.  It is higher up the path selection process and __well-known mandatory__.

### Weight
* Cisco propietary.
* Locally significant only, never advertised to other BGP speakers.
* Highest weight wins.
* Default is 32768 for local routes and 0 for all others.
