---
title: "Spanning-Tree Guards"
date: 2017-11-28T16:18:19+13:00
draft: false
---

# BPDU Guard
* *Portfast* is a prerequisite.
* Can be applied globally or per port.
* Should be applied to all **edge ports**.
* *err-disable* a port on detection of a BPDU.
* Recovery is not automated by default.
  * *shut*, *no shut* restores the interface but doesn't guarantee the absence of a BPDU.
  * *err-disable recovery* can be configured to attempt a restore every *x* seconds.
  * either option results in an *err-diabled* port in the presence of BPDUs.

### Configure per interface
```
interface <iface>
 spanning-tree bpduguard enable
```

### Configure globally
```
spanning-tree portfast edge bpduguard default
```

or

```
spanning-tree portfast bpduguard default
```



# Loop Guard vs Root Guard

## Loop Guard
* Should be applied on **all non-designated ports**.
* Can be configured globally and per interface.
* Prevents an interface from transitioning to designated in the absence of BPDUs.
* Is exclusive from root guard.  The two cannot cannot be active at the same time.
* Interfaces are transitioned to *blocking*, *loop-inconsistent* state.
* Works very well with UDLD which can better detect shared link issues where loop guard cannot.
  * UDLD will also supports detection of unidirectional links on boot.. 
* When applied globally the switch only ports considered to be point-to-point links are enabled.
* Ports auto recover once BPDUs start being received again.

### Configure per interface
```
interface <iface>
 spanning-tree guard loop
!
```

### Configure by default
```
spanning-tree gaurd loop default
```


## Root Guard
* Configured on all **designated ports**.
* Prevents a normally *designated* port from transitioning to *root*.
* Prevent surrounding switches from becoming root.
* Places a port into *listening*, *root-inconsistent* state on the detection of a superior BPDU.
* Ports auto recover once the superior BPDUs stop being received.

### Configure per interface
```
interface <iface>
 spanning-tree guard root
!
```
