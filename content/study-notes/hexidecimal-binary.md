---
title: "Hexidecimal and Binary"
date: 2017-11-30T14:19:19+13:00
draft: false
categories: [  ]
tags: [ remind-me, ipv6, ipv4 ]
---

## Base 16 math.
Hex to decimal.

* 01 = 0 units of 16 + 1 = 1
* 90 = 9 units of 16 = 0 = 144

## Hex to binary
* Base 16 math.
* 0-F

```
0 = 0000    1 = 0001    2 = 0010    3 = 0011
4 = 0100    5 = 0101    6 = 0110    7 = 0111
8 = 1000    9 = 1001    A = 1010    B = 1011
C = 1100    D = 1101    E = 1110    F = 1111
```

## IPv4 to Binary
`128 | 64 | 32 | 16 | 8 | 4 | 2 | 1`

### Example 1
```
Dotted Decimal: 10.1.1.1
Binary: `00001010 00000001 00000001 00000001
```

### Example 2
```
Dotted Decimal: 192.168.1.1
Binary: `11000000 10101000 00000001 00000001
```

## Binary IPv4 to Hex
```
Binary: 11000000 10101000 00000001 00000001
Hex: `C0 A8 01 01
```