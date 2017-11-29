---
title: "Unidirectional Link Detection (UDLD)"
date: 2017-11-29T0:18:19+13:00
draft: false
categories: [ cisco, switching ]
tags: [ loop-prevention ]
---

## Unidirectional Link Detection (UDLD)
* Used to also detect unidirectional links.
* When first enabling agressive mode on the first end-point it waits for the first peering.  This tells the first switch that there is a udld enabled switch attached.
* Has the added benefit of detecting unidirectional links before STP is fully converged after a boot.

## Modes
* Normal
* Agressive

## Timers
* Hello every 15 seconds.
* Dead timer: 3 x hello.
* Uses mutlicast destination of 0180.c200.0002.

## Normal
* Does nothing except logging when the dead timer expires.

## Agressive
* Upon expiration of the dead timer beings sending rapid hellos.
  * Sent every 1 second.
  * Eight total are sent.
* If none of the eight messages return a reply the port is put into **err-dsiable** state and a log generated.

## Configure globlly
`udld (enable|agressive)`

## Configure per interface
```
interface <iface>
 udld (<cr>|agressive|disable)
```
