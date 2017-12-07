---
title: "Cisco SSH without password"
date: 2017-12-07T20:49:19+13:00
draft: false
categories: [ snippet ]
tags: [ ssh, cisco ]
highlight: true
---

## Requirements
* Hostname and domain configured.
* SSH Version 2.
* RSA with minimum 768bits.
* `line vty` configured to include `transport ssh`.
* Appropriate local or remote users.

## Configuration
### Unhashed public-key
```
router# configure terminal
router(config)# hostname <hostname>
router(config)# ip domain-name <domain>
router(config)# crypto key generate rsa modulus <bit-length>
router(config)# ip ssh version 2
router(config)# username <user> secret <secret>
!
router(config)# ip ssh pubkey-chain 
router(conf-ssh-pubkey)# username <username>
router(conf-ssh-pubkey-user)# key-string 
router(conf-ssh-pubkey-data)# <key string | max 256 characters per line>
router(conf-ssh-pubkey-data)# <can be multiple lines>
router(conf-ssh-pubkey-data)# exit
router(conf-ssh-pubkey-user)# exit
router(conf-ssh-pubkey)# exit
router(config)#
router#
```

### Hashed public-key
```
router# configure terminal
router(config)# hostname <hostname>
router(config)# ip domain-name <domain>
router(config)# crypto key generate rsa modulus <bit-length>
router(config)# ip ssh version 2
router(config)# username <user> secret <secret>
!
router(config)# ip ssh pubkey-chain
router(conf-ssh-pubkey)# username <username>
router(conf-ssh-pubkey-user)# key-hash <hashed public key>
router(conf-ssh-pubkey-user)# exit
router(conf-ssh-pubkey)# exit
router(config)# exit
router#
```