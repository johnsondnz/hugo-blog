---
title: "Converting SNMP Textual Objects to Numerical"
date: 2017-03-31T14:19:19+13:00
draft: false
categories: [ snmp ]
tags: [ ]
---

Today I have been working on SNMP monitoring standards for RAD G.SHDSL modems used within our network.  Part of this required reading MIB files and extracting the objects for us to monitor network health of these links, not very exciting but very important to our customers.

The issue with MIB files is that unless they are compiled into your NMS database the textual objects are not very useful.  Therefore, I went about taking the known textual object-id from the MIB files and turning them into numerical object-ids.  This way we can tell our NMS to poll the numerical object-ids without the need to compile the MIB database for each NMS.

To do this I first took the zipped MIB file kindly provided by RAD and placed its contents into `C:\usr\local\share\mibs\` (I know Windows right!! not my first choice).  After this was done I run the following command to ensure that the MIB is found and parsed.

```
C:\ snmptranslate -m +RAD-RadShdsl-MIB -IR -Td hdsl2ShdslEndpointCurrLineStatus
RAD-RadShdsl-MIB::hdsl2ShdslEndpointCurrLineStatus
hdsl2ShdslEndpointCurrLineStatus OBJECT-TYPE
 -- FROM RAD-RadShdsl-MIB
 SYNTAX INTEGER {noSync(2), sync(3)}
 MAX-ACCESS read-only
 STATUS current
 DESCRIPTION "The current line synchronization status."
::= { iso(1) org(3) dod(6) internet(1) private(4) enterprises(1) rad(164) radWan(3) wanGen(1) diverseIfWanGen(6) shdslIf(12) shdslEndpointCurrTable(1) shdslEndpointCurrEntry(1) 4 }
```

Now, lets use the `-m` flag to snmptranslate to tell it to load that mib. We'll use `-m +RAD-RadShdsl-MIB` to indicate that we want the tool to load not only the default set of mibs, but the RAD-RadShdsl-MIB as well (the leading `+` plus means "also").

Once I knew the MIB was being found I rerun with different output options to obtain the decimal oid.

```
C:\ snmptranslate -m +RAD-RadShdsl-MIB -IR -On hdsl2ShdslEndpointCurrLineStatus
 .1.3.6.1.4.1.164.3.1.6.12.1.1.4
```

Job done!!


### Loading additonal MIBs
It is also possible to load all MIBS automatically at run-time by adding the following to `/etc/snmp/snmp.conf`.  Just be sure all the MIBs that each file tries to import are present or you will get a lot of errors.

`mibs +ALL`

Individual MIBs can be automatically loaded at run-time with:

```
mibs=+RAD-RadShdsl-MIB
mibs=+Other-MIB
```

For more reading on net-snmp [check out this tutorial](http://net-snmp.sourceforge.net/tutorial/tutorial-5/commands/output-options.html) or this [explanation](http://net-snmp.sourceforge.net/tutorial/tutorial-5/commands/mib-options.html)

-Donald