---
title: "ED-137 - VHF over IP Bandwidth Requirements"
date: 2017-04-06T14:19:19+13:00
draft: true
categories: [ design ]
tags: [ mpls, voip ]
highlight: true
---

In 2014 during a new enterprise MPLS network build we needed to better understand the bandwidth requirements for VoIP services.  The project involved extensive training on Eurocae standards ED-137 and ED-138, their parts as well as our selected manufacturer's hardware and software.  These standards outline the way Air Traffic Control services such as VHF, data and other services should interoperate and be carried over IP networks.

I noticed early on in a section around bandwidth requirements that layer-1 was not accounted for and some layer-2 header information were missing in the bandwidth calculations.  It also did not account MPLS 4-byte tags, critical to our network services segmentation.

The basics are that SIP sets up the sessions and RTP carries the audio data from the controller to the aircraft.  Traditional SIP/RTP VoIP means audio is full-duplex.  For VHF transmission and receive operates in half-duplex. There needs to be a way to determine the state of a transmitter/receiver and how to setup the push-to-talk (PTT) of the transmitter or squelch (SQU) of the receiver.

ED-137 created a new ED-137 RTP header extension previously known as WG67 (working group 67) to achieve this.  It adds new fields to RTP packets.  These headers are used to signal to a transmitter to gate open for transmission to an aircraft and for receive to signal to the voice switch that receive audio is being transmitted to the controller.

* PTT Type `3-bits`.
* SQU `1-bit`.
* PTT-id `4-bits`.
* Simultaneous Call Transmission `1-bit`.
* X `1-bit`.
* Not Used `21-bits`.
* VF `1-bit`.

## RTP packet with all bits set to zero.
To ensure that the radio always knows the PTT and SQU state mode, these bits within the ED-137 headers are set to zero (except VF: VF ON) and are transmitted at a rate of 5pps (200ms intervals), even in absence of audio.  When PTT or SQU is set to 1 audio begins to transmit at a rate of 50pps (20ms intervals) and the all bits set to 0 packets cease only returning when the is no longer a PTT or SQU.

![RTP ED-137 Headers](/img/ED137-RTP-Headers.png)

## PTT on, no audio payload
To prevent the start of audio transmission being cut off a PTT on packet is sent in advance of the first 20ms audio sample.

![PTT on, no audio](/img/ED137-PTT-On-no-audio.png)

## PTT is on, the audio codec is G.711 ulaw
PTT is on, the audio codec is G.711 ulaw.

![PTT on, with audio](/img/ED137-PTT-On.png)


## Onto the Math

### Terminology
* 8 bits = 1 octet.
* 1 octet = 1 byte.

### Headers
* Layer-1 - 20 octets (preamble, start delimiter and interpacket gap).
* Layer-2 = 18 octets (excluding 802.1q header)
* 802.1q field = 4 octets each.
* MPLS field = 4 octets each.
* IP Header = 20 octets.
* UDP Header = 8 octets.
* RTP Header = 16 octets.
* ED-137 RTP Header Extension = 4 octets.

### Calculating packet overheads
Each packet on our network will include two MPLS fields, one for transport the other for the VPN.  In a igh number of cases one 802.1q field is also use.  The total size of overhead is:

```
20 + 18 + 4 + 4 + 4 + 20 + 8 + 16 + 4 = 98 octets.
98 octets x 8-bits = 784-bits.
```

* 784-bits is for a single packet.
* The audio sample rate is set to 20ms which is 50pps.
* Without audio the sample rate is set to 200ms which is 5pps. (referred to as keepalive).

#### Keepalive overhead.
```
784-bits x 5pps = 3,920bps.
3,920bps / 1000 = 3.92kbps.
```

When there is no PTT or SQU the network requires 3.92kbps for each radio frequency.

#### With audio overhead
```
784-bits x 50pps = 39,200bps.
39,200bps / 1000 = 39.2kbps.
```

When there is audio transmission the application overhead is equal to 39.2kbps.  The rest is easy.

* G.711 is uncompressed PCM at 64kbps.
  * `39.2kbps + 64kbps = 103.2kbps`.
* G.729 is compress audio at 8kbps.
  * `39.2kbps + 8kbps = 47.2kbps`.

### Adding in audio 
Generally speaking, only a single air traffic controller position requires access to a radio for the purpose of transmitting to aircraftat any given time.  This means on a G.729 VHF circuit we expect 47.2kbps for all audio transmissions.

It is possible to channel receive audio to multiple location.  This in turn means a 1:1 growth in bandwidth per destination fpr that audio.  As SIP/RTP sessions are unicast this means we have a situation whereby for every position that is setup as a receiver, the audio is multiplied by the number of receiving controller positions

```
* Required bandwidth = number receiving controller positions x 47.2kbps.
* Required bandwidth example = 6 receiving positions x 47.2kbps = 283.2kbps.
```

VHF radio sites are typically on hilltop or mountains, this ensure that operators gain the maximum amount of coverage for each installation. Connectivity to these locations using service providers can be cost prohibitive or impossible. As such operators will run a combination of service provider’s WANs and their own WAN infrastructure where the returns on investment is better than leasing or contributing towards building core network infrastrucutre for a service provider circuit.

G.729 will normally be chosen due to the savings on WAN bandwidth costs.  It also doesn't from my testing have any recognisable difference to PCM audio quality.  G.711 is also used, however, typically only where technical limitations in applications and/or hardware prevent the use of G.729.  Licensing could also be a business restriction in that every G.729 codec incurs a licensing fee.

-Donald
