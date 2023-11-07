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
h1 - \                               / - h3 
       s2 --- r1 --- s1 --- r2 --- s3
h2 - /                               \ - h4
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
    h1 = net.addHost('h1', mac='00:00:00:00:00:01', ip='192.168.0.1/24')
    h2 = net.addHost('h2', mac='00:00:00:00:00:02', ip='192.168.0.2/24')
    h3 = net.addHost('h3', mac='00:00:00:00:00:03', ip='192.168.1.1/24')
    h4 = net.addHost('h4', mac='00:00:00:00:00:04', ip='192.168.1.2/24')
    r1 = net.addHost('r1', mac='00:00:00:00:00:05', ip='10.0.0.1/8')
    r2 = net.addHost('r2', mac='00:00:00:00:00:06', ip='10.0.0.2/8')
    s1 = net.addSwitch('s1', failMode='standalone')
    s2 = net.addSwitch('s2', failMode='standalone')
    s3 = net.addSwitch('s3', failMode='standalone')

    print("*** Creating links")
    net.addLink(r1, s1)
    net.addLink(r2, s1)
    net.addLink(r1, s2)
    net.addLink(r2, s3)
    net.addLink(h1, s2)
    net.addLink(h2, s2)
    net.addLink(h3, s3)
    net.addLink(h4, s3)

    print("*** Building network")
    net.start()

    r1.cmd('echo 1 > /proc/sys/net/ipv4/ip_forward')
    r2.cmd('echo 1 > /proc/sys/net/ipv4/ip_forward')
    r1.cmd('ifconfig r1-eth1 192.168.0.10')
    r2.cmd('ifconfig r2-eth1 192.168.1.10')
    r1.cmd('ip route add 192.168.1.0/24 via 10.0.0.2')
    r2.cmd('ip route add 192.168.0.0/24 via 10.0.0.1')
    h1.cmd('route add default gw 192.168.0.10')
    h2.cmd('route add default gw 192.168.0.10')
    h3.cmd('route add default gw 192.168.1.10')
    h4.cmd('route add default gw 192.168.1.10')

    print("*** Adding some commands")
    if 'ESPTR' in sys.argv[1]:
        h1.cmd('ip xfrm policy add dir in src 192.168.1.1/32 dst 192.168.0.1/32 tmpl proto esp mode transport')
        h1.cmd('ip xfrm policy add dir out src 192.168.0.1/32 dst 192.168.1.1/32 tmpl proto esp mode transport')
        h3.cmd('ip xfrm policy add dir in src 192.168.0.1/32 dst 192.168.1.1/32 tmpl proto esp mode transport')
        h3.cmd('ip xfrm policy add dir out src 192.168.1.1/32 dst 192.168.0.1/32 tmpl proto esp mode transport')

        h1.cmd('ip xfrm state add src 192.168.0.1 dst 192.168.1.1 proto esp spi 1 enc \'cbc(aes)\' 0x3ed0af408cf5dcbf5d5d9a5fa806b224 mode transport')
        h1.cmd('ip xfrm state add src 192.168.1.1 dst 192.168.0.1 proto esp spi 1 enc \'cbc(aes)\' 0x3ed0af408cf5dcbf5d5d9a5fa806b224 mode transport')
        h3.cmd('ip xfrm state add src 192.168.0.1 dst 192.168.1.1 proto esp spi 1 enc \'cbc(aes)\' 0x3ed0af408cf5dcbf5d5d9a5fa806b224 mode transport')
        h3.cmd('ip xfrm state add src 192.168.1.1 dst 192.168.0.1 proto esp spi 1 enc \'cbc(aes)\' 0x3ed0af408cf5dcbf5d5d9a5fa806b224 mode transport')
    elif 'AHTR' in sys.argv[1]:
        h1.cmd('ip xfrm policy add dir in src 192.168.1.1/32 dst 192.168.0.1/32 tmpl proto ah mode transport')
        h1.cmd('ip xfrm policy add dir out src 192.168.0.1/32 dst 192.168.1.1/32 tmpl proto ah mode transport')
        h3.cmd('ip xfrm policy add dir in src 192.168.0.1/32 dst 192.168.1.1/32 tmpl proto ah mode transport')
        h3.cmd('ip xfrm policy add dir out src 192.168.1.1/32 dst 192.168.0.1/32 tmpl proto ah mode transport')

        h1.cmd('ip xfrm state add src 192.168.0.1 dst 192.168.1.1 proto ah spi 0x401 mode transport auth \"hmac(sha1)\" 0x12345678123456781234567812345678')
        h1.cmd('ip xfrm state add src 192.168.1.1 dst 192.168.0.1 proto ah spi 0x401 mode transport auth \"hmac(sha1)\" 0x12345678123456781234567812345678')
        h3.cmd('ip xfrm state add src 192.168.0.1 dst 192.168.1.1 proto ah spi 0x401 mode transport auth \"hmac(sha1)\" 0x12345678123456781234567812345678')
        h3.cmd('ip xfrm state add src 192.168.1.1 dst 192.168.0.1 proto ah spi 0x401 mode transport auth \"hmac(sha1)\" 0x12345678123456781234567812345678')
    elif 'AHESTR' in sys.argv[1]:
        h1.cmd('ip xfrm policy add dir in src 192.168.1.1/32 dst 192.168.0.1/32 tmpl proto esp tmpl proto ah mode transport')
        h1.cmd('ip xfrm policy add dir out src 192.168.0.1/32 dst 192.168.1.1/32 tmpl proto esp tmpl proto ah mode transport')
        h3.cmd('ip xfrm policy add dir in src 192.168.0.1/32 dst 192.168.1.1/32 tmpl proto esp tmpl proto ah mode transport')
        h3.cmd('ip xfrm policy add dir out src 192.168.1.1/32 dst 192.168.0.1/32 tmpl proto esp tmpl proto ah mode transport')

        h1.cmd('ip xfrm state add src 192.168.0.1 dst 192.168.1.1 proto esp spi 1 enc \'cbc(aes)\' 0x3ed0af408cf5dcbf5d5d9a5fa806b224 mode transport')
        h1.cmd('ip xfrm state add src 192.168.1.1 dst 192.168.0.1 proto esp spi 1 enc \'cbc(aes)\' 0x3ed0af408cf5dcbf5d5d9a5fa806b224 mode transport')
        h3.cmd('ip xfrm state add src 192.168.0.1 dst 192.168.1.1 proto esp spi 1 enc \'cbc(aes)\' 0x3ed0af408cf5dcbf5d5d9a5fa806b224 mode transport')
        h3.cmd('ip xfrm state add src 192.168.1.1 dst 192.168.0.1 proto esp spi 1 enc \'cbc(aes)\' 0x3ed0af408cf5dcbf5d5d9a5fa806b224 mode transport')
        h1.cmd('ip xfrm state add src 192.168.0.1 dst 192.168.1.1 proto ah spi 0x401 mode transport auth \"hmac(sha1)\" 0x12345678123456781234567812345678')
        h1.cmd('ip xfrm state add src 192.168.1.1 dst 192.168.0.1 proto ah spi 0x401 mode transport auth \"hmac(sha1)\" 0x12345678123456781234567812345678')
        h3.cmd('ip xfrm state add src 192.168.0.1 dst 192.168.1.1 proto ah spi 0x401 mode transport auth \"hmac(sha1)\" 0x12345678123456781234567812345678')
        h3.cmd('ip xfrm state add src 192.168.1.1 dst 192.168.0.1 proto ah spi 0x401 mode transport auth \"hmac(sha1)\" 0x12345678123456781234567812345678')
    elif 'ESPTU' in sys.argv[1]:
        r1.cmd('ip xfrm policy add dir in src 192.168.1.1/32 dst 192.168.0.1/32 tmpl src 10.0.0.2 dst 10.0.0.1 proto esp mode tunnel')
        r1.cmd('ip xfrm policy add dir fwd src 192.168.1.1/32 dst 192.168.0.1/32 tmpl src 10.0.0.2 dst 10.0.0.1 proto esp mode tunnel')
        r1.cmd('ip xfrm policy add dir out src 192.168.0.1/32 dst 192.168.1.1/32 tmpl src 10.0.0.1 dst 10.0.0.2 proto esp mode tunnel')
        r2.cmd('ip xfrm policy add dir in src 192.168.0.1/32 dst 192.168.1.1/32 tmpl src 10.0.0.1 dst 10.0.0.2 proto esp mode tunnel')
        r2.cmd('ip xfrm policy add dir fwd src 192.168.0.1/32 dst 192.168.1.1/32 tmpl src 10.0.0.1 dst 10.0.0.2 proto esp mode tunnel')
        r2.cmd('ip xfrm policy add dir out src 192.168.1.1/32 dst 192.168.0.1/32 tmpl src 10.0.0.2 dst 10.0.0.1 proto esp mode tunnel')

        r1.cmd('ip xfrm state add src 10.0.0.1 dst 10.0.0.2 proto esp spi 0x201 mode tunnel enc \"cbc(aes)\" 0x303631383332323363323361323165386233366335363662')
        r1.cmd('ip xfrm state add src 10.0.0.2 dst 10.0.0.1 proto esp spi 0x201 mode tunnel enc \"cbc(aes)\" 0x303631383332323363323361323165386233366335363662')
        r2.cmd('ip xfrm state add src 10.0.0.1 dst 10.0.0.2 proto esp spi 0x201 mode tunnel enc \"cbc(aes)\" 0x303631383332323363323361323165386233366335363662')
        r2.cmd('ip xfrm state add src 10.0.0.2 dst 10.0.0.1 proto esp spi 0x201 mode tunnel enc \"cbc(aes)\" 0x303631383332323363323361323165386233366335363662')
    elif 'AHTU' in sys.argv[1]:
        r1.cmd('ip xfrm policy add dir in src 192.168.1.1/32 dst 192.168.0.1/32 tmpl src 10.0.0.2 dst 10.0.0.1 proto ah mode tunnel')
        r1.cmd('ip xfrm policy add dir fwd src 192.168.1.1/32 dst 192.168.0.1/32 tmpl src 10.0.0.2 dst 10.0.0.1 proto ah mode tunnel')
        r1.cmd('ip xfrm policy add dir out src 192.168.0.1/32 dst 192.168.1.1/32 tmpl src 10.0.0.1 dst 10.0.0.2 proto ah mode tunnel')
        r2.cmd('ip xfrm policy add dir in src 192.168.0.1/32 dst 192.168.1.1/32 tmpl src 10.0.0.1 dst 10.0.0.2 proto ah mode tunnel')
        r2.cmd('ip xfrm policy add dir fwd src 192.168.0.1/32 dst 192.168.1.1/32 tmpl src 10.0.0.1 dst 10.0.0.2 proto ah mode tunnel')
        r2.cmd('ip xfrm policy add dir out src 192.168.1.1/32 dst 192.168.0.1/32 tmpl src 10.0.0.2 dst 10.0.0.1 proto ah mode tunnel')

        r1.cmd('ip xfrm state add src 10.0.0.1 dst 10.0.0.2 proto ah spi 0x201 mode tunnel auth \"hmac(sha1)\" 0x12345678123456781234567812345678')
        r1.cmd('ip xfrm state add src 10.0.0.2 dst 10.0.0.1 proto ah spi 0x201 mode tunnel auth \"hmac(sha1)\" 0x12345678123456781234567812345678')
        r2.cmd('ip xfrm state add src 10.0.0.1 dst 10.0.0.2 proto ah spi 0x201 mode tunnel auth \"hmac(sha1)\" 0x12345678123456781234567812345678')
        r2.cmd('ip xfrm state add src 10.0.0.2 dst 10.0.0.1 proto ah spi 0x201 mode tunnel auth \"hmac(sha1)\" 0x12345678123456781234567812345678')
    elif 'AHESTU' in sys.argv[1]: # not working yet
        r1.cmd('ip xfrm policy add dir in src 192.168.1.1/32 dst 192.168.0.1/32 tmpl src 10.0.0.2 dst 10.0.0.1 proto ah tmpl src 10.0.0.2 dst 10.0.0.1 proto esp mode tunnel')
        r1.cmd('ip xfrm policy add dir fwd src 192.168.1.1/32 dst 192.168.0.1/32 tmpl src 10.0.0.2 dst 10.0.0.1 proto ah tmpl src 10.0.0.2 dst 10.0.0.1 proto esp mode tunnel')
        r1.cmd('ip xfrm policy add dir out src 192.168.0.1/32 dst 192.168.1.1/32 tmpl src 10.0.0.1 dst 10.0.0.2 proto ah tmpl src 10.0.0.1 dst 10.0.0.2 proto esp mode tunnel')
        r2.cmd('ip xfrm policy add dir in src 192.168.0.1/32 dst 192.168.1.1/32 tmpl src 10.0.0.1 dst 10.0.0.2 proto ah tmpl src 10.0.0.1 dst 10.0.0.2 proto esp mode tunnel')
        r2.cmd('ip xfrm policy add dir fwd src 192.168.0.1/32 dst 192.168.1.1/32 tmpl src 10.0.0.1 dst 10.0.0.2 proto ah tmpl src 10.0.0.1 dst 10.0.0.2 proto esp mode tunnel')
        r2.cmd('ip xfrm policy add dir out src 192.168.1.1/32 dst 192.168.0.1/32 tmpl src 10.0.0.2 dst 10.0.0.1 proto ah tmpl src 10.0.0.2 dst 10.0.0.1 proto esp mode tunnel')

        r1.cmd('ip xfrm state add src 10.0.0.1 dst 10.0.0.2 proto esp spi 0x201 mode tunnel enc \"cbc(aes)\" 0x303631383332323363323361323165386233366335363662')
        r1.cmd('ip xfrm state add src 10.0.0.2 dst 10.0.0.1 proto esp spi 0x201 mode tunnel enc \"cbc(aes)\" 0x303631383332323363323361323165386233366335363662')
        r2.cmd('ip xfrm state add src 10.0.0.1 dst 10.0.0.2 proto esp spi 0x201 mode tunnel enc \"cbc(aes)\" 0x303631383332323363323361323165386233366335363662')
        r2.cmd('ip xfrm state add src 10.0.0.2 dst 10.0.0.1 proto esp spi 0x201 mode tunnel enc \"cbc(aes)\" 0x303631383332323363323361323165386233366335363662')
        r1.cmd('ip xfrm state add src 10.0.0.1 dst 10.0.0.2 proto ah spi 0x201 mode tunnel auth \"hmac(sha1)\" 0x12345678123456781234567812345678')
        r1.cmd('ip xfrm state add src 10.0.0.2 dst 10.0.0.1 proto ah spi 0x201 mode tunnel auth \"hmac(sha1)\" 0x12345678123456781234567812345678')
        r2.cmd('ip xfrm state add src 10.0.0.1 dst 10.0.0.2 proto ah spi 0x201 mode tunnel auth \"hmac(sha1)\" 0x12345678123456781234567812345678')
        r2.cmd('ip xfrm state add src 10.0.0.2 dst 10.0.0.1 proto ah spi 0x201 mode tunnel auth \"hmac(sha1)\" 0x12345678123456781234567812345678')


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
sudo python ipsec.py [-ESPTR,AHTR,ESPTU,AHTU,AHESTR,AHESTU]
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
