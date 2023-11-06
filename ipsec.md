---
tags: security-tutorials
---

# IPSEC

**In this short demo you will:** 
- How to run a simple IPSEC network topology

**Requirements:** 
- Mininet-WiFi - https://github.com/intrig-unicamp/mininet-wifi


Network topology:
```
              10.0.0.1                             10.0.0.2           
              h1 (h1-eth0) ----------------------- (h2-eth0) h2
```     

The script below produces the network topology illustrated above.

```python=
#!/usr/bin/python

"""
   Example of IPSEC
"""

import sys

from mininet.net import Mininet
from mininet.cli import CLI
from mininet.log import setLogLevel


def topology():

    "Create a network."
    net = Mininet()

    print("*** Creating nodes")
    h1 = net.addHost('h1', mac='00:00:00:00:00:01', ip='10.0.0.1/8')
    h2 = net.addHost('h2', mac='00:00:00:00:00:02', ip='10.0.0.2/8')
    s1 = net.addSwitch('s1', failMode='standalone')

    print("*** Creating links")
    net.addLink(h1, s1)
    net.addLink(h2, s1)

    print("*** Building network")
    net.start()

    print("*** Adding some commands")
    if 'ESPTR' in sys.argv[1]:
        #ESP TRANSPORT (from h1 to h2)
        h1.cmd('ip xfrm policy add dir out src 10.0.0.1 dst 10.0.0.2 tmpl proto esp mode transport')
        h1.cmd('ip xfrm state add src 10.0.0.1 dst 10.0.0.2 proto esp spi 1 enc \'cbc(aes)\' 0x3ed0af408cf5dcbf5d5d9a5fa806b224 mode transport')
        h2.cmd('ip xfrm policy add dir in src 10.0.0.1 dst 10.0.0.2 tmpl proto esp mode transport')
        h2.cmd('ip xfrm state add src 10.0.0.1 dst 10.0.0.2 proto esp spi 1 enc \'cbc(aes)\' 0x3ed0af408cf5dcbf5d5d9a5fa806b224 mode transport')
        #ESP TRANSPORT (from h2 to h1)
        h1.cmd('ip xfrm policy add dir in src 10.0.0.2 dst 10.0.0.1 tmpl proto esp mode transport')
        h1.cmd('ip xfrm state add src 10.0.0.2 dst 10.0.0.1 proto esp spi 1 enc \'cbc(aes)\' 0x3ed0af408cf5dcbf5d5d9a5fa806b224 mode transport')
        h2.cmd('ip xfrm policy add dir out src 10.0.0.2 dst 10.0.0.1 tmpl proto esp mode transport')
        h2.cmd('ip xfrm state add src 10.0.0.2 dst 10.0.0.1 proto esp spi 1 enc \'cbc(aes)\' 0x3ed0af408cf5dcbf5d5d9a5fa806b224 mode transport')
    elif  'AHTR' in sys.argv[1]:
        #AH TRANSPORT (from h1 to h2)
        h1.cmd('ip xfrm policy add dir out src 10.0.0.1 dst 10.0.0.2 tmpl proto ah mode transport')
        h1.cmd('ip xfrm state add src 10.0.0.1 dst 10.0.0.2 proto ah spi 0x401 mode transport auth \"hmac(sha1)\" 0x12345678123456781234567812345678')
        h2.cmd('ip xfrm policy add dir in src 10.0.0.1 dst 10.0.0.2 tmpl proto ah mode transport')
        h2.cmd('ip xfrm state add src 10.0.0.1 dst 10.0.0.2 proto ah spi 0x401 mode transport auth \"hmac(sha1)\" 0x12345678123456781234567812345678')
        #AH TRANSPORT (from h2 to h1)
        h1.cmd('ip xfrm policy add dir in src 10.0.0.2 dst 10.0.0.1 tmpl proto ah mode transport')
        h1.cmd('ip xfrm state add src 10.0.0.2 dst 10.0.0.1 proto ah spi 0x401 mode transport auth \"hmac(sha1)\" 0x12345678123456781234567812345678')
        h2.cmd('ip xfrm policy add dir out src 10.0.0.2 dst 10.0.0.1 tmpl proto ah mode transport')
        h2.cmd('ip xfrm state add src 10.0.0.2 dst 10.0.0.1 proto ah spi 0x401 mode transport auth \"hmac(sha1)\" 0x12345678123456781234567812345678')
    elif 'ESPTU' in sys.argv[1]:
        #ESP TUNNEL (from h1 to h2)
        h1.cmd('ip xfrm policy add dir in src 10.0.0.0/8 dst 10.0.0.0/8 tmpl src 10.0.0.2 dst 10.0.0.1 proto esp mode tunnel')
        h1.cmd('ip xfrm state add src 10.0.0.2 dst 10.0.0.1 proto esp spi 0x201 mode tunnel enc \"cbc(aes)\" 0x303631383332323363323361323165386233366335363662')
        h2.cmd('ip xfrm policy add dir out src 10.0.0.0/8 dst 10.0.0.0/8 tmpl src 10.0.0.2 dst 10.0.0.1 proto esp mode tunnel')
        h2.cmd('ip xfrm state add src 10.0.0.2 dst 10.0.0.1  proto esp spi 0x201 mode tunnel enc \"cbc(aes)\" 0x303631383332323363323361323165386233366335363662')
        #ESP TUNNEL (from h2 to h1)
        h1.cmd('ip xfrm policy add dir out src 10.0.0.0/8 dst 10.0.0.0/8 tmpl src 10.0.0.1 dst 10.0.0.2 proto esp mode tunnel')
        h1.cmd('ip xfrm state add src 10.0.0.1 dst 10.0.0.2 proto esp spi 0x201 mode tunnel enc \"cbc(aes)\" 0x303631383332323363323361323165386233366335363662')
        h2.cmd('ip xfrm policy add dir in src 10.0.0.0/8 dst 10.0.0.0/8 tmpl src 10.0.0.1 dst 10.0.0.2 proto esp mode tunnel')
        h2.cmd('ip xfrm state add src 10.0.0.1 dst 10.0.0.2 proto esp spi 0x201 mode tunnel enc \"cbc(aes)\" 0x303631383332323363323361323165386233366335363662')
    elif 'AHTU' in sys.argv[1]:
        #AH TUNNEL (from h1 to h2)
        h1.cmd('ip xfrm policy add dir in src 10.0.0.0/8 dst 10.0.0.0/8 tmpl src 10.0.0.2 dst 10.0.0.1 proto ah mode tunnel')
        h1.cmd('ip xfrm state add src 10.0.0.2 dst 10.0.0.1 proto ah spi 0x201 mode tunnel auth \"hmac(sha1)\" 0x12345678123456781234567812345678')
        h2.cmd('ip xfrm policy add dir out src 10.0.0.0/8 dst 10.0.0.0/8 tmpl src 10.0.0.2 dst 10.0.0.1 proto ah mode tunnel')
        h2.cmd('ip xfrm state add src 10.0.0.2 dst 10.0.0.1  proto ah spi 0x201 mode tunnel auth \"hmac(sha1)\" 0x12345678123456781234567812345678')
        #AH TUNNEL (from h2 to h1)
        h1.cmd('ip xfrm policy add dir out src 10.0.0.0/8 dst 10.0.0.0/8 tmpl src 10.0.0.1 dst 10.0.0.2 proto ah mode tunnel')
        h1.cmd('ip xfrm state add src 10.0.0.1 dst 10.0.0.2 proto ah spi 0x201 mode tunnel auth \"hmac(sha1)\" 0x12345678123456781234567812345678')
        h2.cmd('ip xfrm policy add dir in src 10.0.0.0/8 dst 10.0.0.0/8 tmpl src 10.0.0.1 dst 10.0.0.2 proto ah mode tunnel')
        h2.cmd('ip xfrm state add src 10.0.0.1 dst 10.0.0.2 proto ah spi 0x201 mode tunnel auth \"hmac(sha1)\" 0x12345678123456781234567812345678')

    print("*** Running CLI")
    CLI(net)

    print("*** Stopping network")
    net.stop()


if __name__ == '__main__':
    setLogLevel('info')
    topology()
```

You can run the code with the command below (considering that the filename is ipsec.py)
```
sudo python ipsec.py [-ESPTR,AHTR,ESPTU,AHTU]
```

Then, you can run Wireshark on s1-eth1

```
mininet-wifi> xterm s1
s1# wireshark s1-eth1
```

And you can try to communicate h1 and h2 as follows:

```
mininet-wifi> h2 ping -c2 h3
```
