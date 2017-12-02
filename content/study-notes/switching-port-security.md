---
title: "Port Security"
date: 2017-11-29T10:18:19+13:00
draft: false
categories: [ study-notes ]
tags: [ security, switching ]
highlight: false
---

## Port Security
* Port level security feature.
* Exclusive use from 802.1x.  You cannot use both features on the same port.
* Modes
 * Shutdown: Default state, the port is placed into **err-disabled** mode.  The LED switches off.
  * Protect: Frames from the offending host are dropped.
  * Restrict: Like **protect** frames are droped from the offending host and logged.
* `sticky` used to ensure learned MAC survives a `reload`.  THe MAC is saved in the running and startup configurations.
* Can permit more the one MAC per port.
* Can forbib a mac on a port.
* An attempt to configure secure mac count greater than that of the configured limit would result in a error message.
* If the port in shutdown all dynamically learned mac-addresses are flushed, except for sticky macs.
* Does not work on trunk ports.

## Configure per interface
### Shutdown (Default mode)
```
switch(config)#interface <iface>
switch(config-if)# switchport port-security
``` 

or

```
switch(config)# interface <iface>
switch(config-if)# switchport port-security violation shutdown
```

### Protect
```
switch(config)# interface <iface>
switch(config-if)# switchport port-security violation protect
``` 

### Restrict
```
switch(config)# interface <iface>
switch(config-if)# switchport port-security violation restrict
``` 

### Sticky, static, forbidden MAC
```
switch(config)# interface <iface>
switch(config-if)# switchport port-security mac-address (48bit mac|sticky)
``` 

```
switch(config)# interface <iface>
switch(config-if)# switchport port-security mac-address forbidden <mac>
``` 

### MAC limit
```
switch(config)#interface <iface>
 switchport port-security maximum <1-4097>
```

## Verify
### Operation
```
ASW1#sh port-security                
Secure Port  MaxSecureAddr  CurrentAddr  SecurityViolation  Security Action
                (Count)       (Count)          (Count)
---------------------------------------------------------------------------
      Gi1/0              1            1                  0         Shutdown
---------------------------------------------------------------------------
Total Addresses in System (excluding one mac per port)     : 0
Max Addresses limit in System (excluding one mac per port) : 4096

ASW1#sh port-security interface Gi1/0
Port Security              : Enabled
Port Status                : Secure-up
Violation Mode             : Shutdown
Aging Time                 : 0 mins
Aging Type                 : Absolute
SecureStatic Address Aging : Disabled
Maximum MAC Addresses      : 1
Total MAC Addresses        : 1
Configured MAC Addresses   : 0
Sticky MAC Addresses       : 0
Last Source Address:Vlan   : 0050.0100.0500:10
Security Violation Count   : 0
```

### Sticky MAC
```
ASW1#show run int gi1/0
Building configuration...

Current configuration : 353 bytes
!
interface GigabitEthernet1/0
 description Client1
 switchport access vlan 10
 switchport mode access
 switchport port-security mac-address sticky
 switchport port-security mac-address sticky 0050.0100.0500
 switchport port-security
 spanning-tree portfast edge
 spanning-tree bpduguard enable
end

ASW1#show port-security int gi1/0
Port Security              : Enabled
Port Status                : Secure-up
Violation Mode             : Shutdown
Aging Time                 : 0 mins
Aging Type                 : Absolute
SecureStatic Address Aging : Disabled
Maximum MAC Addresses      : 1
Total MAC Addresses        : 1
Configured MAC Addresses   : 0
Sticky MAC Addresses       : 1
Last Source Address:Vlan   : 0050.0100.0500:10
Security Violation Count   : 0

ASW1# sh ow port-security address
               Secure Mac Address Table
-----------------------------------------------------------------------------
Vlan    Mac Address       Type                          Ports   Remaining Age
                                                                   (mins)    
----    -----------       ----                          -----   -------------
  10    0050.0100.0500    SecureSticky                  Gi1/0        -
-----------------------------------------------------------------------------
Total Addresses in System (excluding one mac per port)     : 0
Max Addresses limit in System (excluding one mac per port) : 4096
```

## Recovery from err-disable
* Issue `shutdown` followed by `no shutdown` on the port.
* or issue the global command `errdisable recovery cause psecure-violation` for automated recovery.
  * not to be confused with `security-violation` which is for 802.1x.
  * recovery timer default is every 300 seconds.