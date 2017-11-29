---
title: "OSPF LSA Types and Stubs"
date: 2017-11-29T21:48:19+13:00
draft: false
categories: [ cisco, routing ]
tags: [ ospf, stubs ]
---

## LSAs
* __Type-1__: Router link.
  * Flooded by all routers in an area.
  * Route types: `(O)`.
* __Type-2__: Network link.
  * Sent by the DR.  Lists all the routers on the segment it is adjacent to.
  * Flooded within the area.
  * Route types: `(O)`.
* __Type-3__: Network summary link.
  * Sent by the ASBR between areas.  Typically from/to area 0.
  * Lists all the prefixes avaialble in a given area.
  * Route types: `(O IA)`.
* __Type-4__: AS external ASBR summary link.
  * Sent by ASBRs.
  * Describes how to reach the ASBR.
  * Route types: `(O IA)`.
* __Type-5__: External link.
  * Sent by ASBRs.
  * Describes routes external to OSPF.
  * Route types: `(O E1)` or `(O E2)` with E2 the default.
* __Type-7__: NSSA external.
  * Sent by ABSR inside an NSSA.
  * Converted to type-5 LSA by ABRs.
  * Route types: `(O N1)` or `(O N2)` with N2 the default.

*__Notes__*:

* N2 and E2 types are default for external routes.  These types do not increment the cost as they are propogarted within OSPF.  Cost remains 20.
* N1 and E1 are used if you want to increment the cost to the ABSR as the route is propogated within OSPF.  __Best routes!!__

## Stub Areas
* Used when an area is a spur of the network.
* Saves CPU cycles by sending a subset or only default route to the stub area router.
* BAckbone area 0 can never be a stub area.

## Types
* Stub area.
  * Cannot contain and ASBR.
  * Blocks type-5 LSAs.
  * ABR converts type-5 LSAs to a type-3 LSA (default route).
  * Route types: `(O IA)`.
* Totally stuby area.
  * Cisco propietary.
  * Blocks type-3, type-4 or type-5 LSAs.
  * ABR converts type-5 LSAs to a type-3 LSA (default route).
  * `no-summary` knob only required on the ABR.
  * Route types: `(O IA)`.
* NSSA
  * Accepts type-3 LSAs.  But does not get the default route by default.
  * ASBR sends routes into the area as type-7 LSA.
  * ABR converts type-7 LSA to type-5 LSA before sending to area 0.
  * Route types:
    * `(O IA)`.
    * `(O N1)` or `(O N2)` with N2 the default.
* Totally stubby NSSA.
  * Same as an NSSA but receives the default route from OSPF ABR as a type-3 LSA.
  * `no-summary` knob only required on the ABR.
  * Route types:
    * `(O IA)`.
    * `(O N1)` or `(O N2)` with N2 the default.

*__Notes__*: Candidate default route is represented with `(O*IA) `or `(O*Ex)` or `(O*Nx)`.  Where x = 1 or 2 depending on the OSPF metric-type.

## Configure

### Stub area
```
router(config)# router ospf <process>
router(config-router)# area <area> stub
```

### Totally stubby area
Only required on the ABR.

```
router(config)# router ospf <process>
router(config-router)# area <area> stub no-summary
```

### NSSA (Not so stubby stub area)
```
router(config)# router ospf <process>
router(config-router)# area <area> nssa
```

### Totally NSSA (Not so stubby stub area)
Only required on the ABR.

```
router(config)# router ospf <process>
router(config-router)# area <area> nssa no-summary
```

## Verify
[OSPF](/posts/routing-ospf/)
