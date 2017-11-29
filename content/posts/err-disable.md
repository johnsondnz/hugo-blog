---
title: "Error Disable"
date: 2017-11-29T10:54:19+13:00
draft: false
categories: [ cisco, switching ]
tags: [ security ]
---

# Recovery
* All detectors are on by default.
* All recovery modes are disabled by default.
* Default recovery timer is 300 seconds.
  * Min is 30 seconds.
* `detect` and `recovery` modes configured globally.

## Configure
### Timer interval
`err-disable recovery interval <30-86400>`

### Detect
```
ASW1(config)#errdisable detect cause ?
  all                  Enable error detection on all cases
  arp-inspection       Enable error detection for arp inspection
  dhcp-rate-limit      Enable error detection on dhcp-rate-limit
  dtp-flap             Enable error detection on dtp-flapping
  gbic-invalid         Enable error detection on gbic-invalid
  inline-power         Enable error detection for inline-power
  l2ptguard            Enable error detection on l2protocol-tunnel
  link-flap            Enable error detection on linkstate-flapping
  loopback             Enable error detection on loopback
  pagp-flap            Enable error detection on pagp-flapping
  pppoe-ia-rate-limit  Enable error detection on PPPoE IA rate-limit
  psp                  Enable error detection on PSP
  security-violation   Enable error detection on 802.1x-guard
  sfp-config-mismatch  Enable error detection on SFP config mismatch
```

### Recovery
```
ASW1(config)#errdisable recovery cause ?
  all                   Enable timer to recover from all error causes
  arp-inspection        Enable timer to recover from arp inspection error
                        disable state
  bpduguard             Enable timer to recover from BPDU Guard error
  channel-misconfig     Enable timer to recover from channel misconfig error
                        (STP)
  dhcp-rate-limit       Enable timer to recover from dhcp-rate-limit error
  dtp-flap              Enable timer to recover from dtp-flap error
  gbic-invalid          Enable timer to recover from invalid GBIC error
  inline-power          Enable timer to recover from inline-power error
  l2ptguard             Enable timer to recover from l2protocol-tunnel error
  link-flap             Enable timer to recover from link-flap error
  link-monitor-failure  Enable timer to recover from link monitoring failure
  loopback              Enable timer to recover from loopback error
  mac-limit             Enable timer to recover from mac limit disable state
  oam-remote-failure    Enable timer to recover from OAM detected remote
                        failure
  pagp-flap             Enable timer to recover from pagp-flap error
  port-mode-failure     Enable timer to recover from port mode change failure
  pppoe-ia-rate-limit   Enable timer to recover from PPPoE IA rate-limit error
  psecure-violation     Enable timer to recover from psecure violation error
  psp                   Enable timer to recover from psp
  security-violation    Enable timer to recover from 802.1x violation error
  sfp-config-mismatch   Enable timer to recover from SFP config mismatch error
  storm-control         Enable timer to recover from storm-control error
  udld                  Enable timer to recover from udld error
  unicast-flood         Enable timer to recover from unicast flood error
  vmps                  Enable timer to recover from vmps shutdown error

```

## Verify
```
ASW1#show errdisable detect 
ErrDisable Reason            Detection        Mode
-----------------            ---------        ----
arp-inspection               Enabled          port
bpduguard                    Enabled          port
channel-misconfig (STP)      Enabled          port
community-limit              Enabled          port
dhcp-rate-limit              Enabled          port
dtp-flap                     Enabled          port
ekey                         Enabled          port
gbic-invalid                 Enabled          port
iif-reg-failure              Enabled          port
inline-power                 Enabled          port
invalid-policy               Enabled          port
l2ptguard                    Enabled          port
link-flap                    Enabled          port
link-monitor-failure         Enabled          port
loopback                     Enabled          port
lsgroup                      Enabled          port
oam-remote-failure           Enabled          port
mac-limit                    Enabled          port
pagp-flap                    Enabled          port
port-mode-failure            Enabled          port
pppoe-ia-rate-limit          Enabled          port
psecure-violation            Enabled          port
security-violation           Enabled          port
sfp-config-mismatch          Enabled          port
sgacl_limitation:enforcem    Enabled          port
sgacl_limitation:multiple    Enabled          port
storm-control                Enabled          port
udld                         Enabled          port
unicast-flood                Enabled          port
vmps                         Enabled          port
psp                          Enabled          port
dual-active-recovery         Enabled          port
evc-lite input mapping fa    Enabled          port
vsl-and-non-vsl-port-pair    Enabled          port
Recovery command: "clear     Enabled          port
fasthello-and-non-fasthel    Enabled          port
```

```
ASW1#show errdisable recovery 
ErrDisable Reason            Timer Status
-----------------            --------------
arp-inspection               Disabled
bpduguard                    Disabled
channel-misconfig (STP)      Disabled
dhcp-rate-limit              Disabled
dtp-flap                     Disabled
gbic-invalid                 Disabled
inline-power                 Disabled
l2ptguard                    Disabled
link-flap                    Disabled
mac-limit                    Disabled
link-monitor-failure         Disabled
loopback                     Disabled
oam-remote-failure           Disabled
pagp-flap                    Disabled
port-mode-failure            Disabled
pppoe-ia-rate-limit          Disabled
psecure-violation            Disabled
security-violation           Disabled
sfp-config-mismatch          Disabled
storm-control                Disabled
udld                         Disabled
unicast-flood                Disabled
vmps                         Disabled
psp                          Disabled
dual-active-recovery         Disabled
evc-lite input mapping fa    Disabled
Recovery command: "clear     Disabled

Timer interval: 300 seconds

Interfaces that will be enabled at the next timeout:

```