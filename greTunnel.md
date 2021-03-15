---
tags: security-tutorials
---

# GRE Tunnel

**In this short demo you will:** 
- How to run a GRE tunnel network topology

**Requirements:** 
- Mininet-WiFi - https://github.com/intrig-unicamp/mininet-wifi
- bridge-utils


Network topology:
```
               140.116.172.1                     140.116.172.2           
              h9 (h9-eth0) ----------------------- (h10-eth0) h10
              |   192.168.10.253--(tunnel)--192.168.10.253     |
          (h9-eth1)                                        (h10-eth1)   
        192.168.10.254                                   192.168.20.254 
              |                                                |
              |                                                |
            (eth3)                                          (eth3)    
              h1                                              h5
     (eth0) (eth1) (eth2)                            (eth0) (eth1) (eth2)
       |      |      |                                 |       |      | 
      h2      h3     h4                                h6      h7     h8
 ```     
      
**Note**: 
- h9 and h10 are routers (gateways)      
- h1 and h5 are switches
- the rest are simple hosts

The script below produces the network topology illustrated above.

```python=
#!/usr/bin/python

"""
   Example of GRE Tunnel
"""

from mininet.net import Mininet
from mininet.cli import CLI
from mininet.log import setLogLevel


def topology():

    "Create a network."
    net = Mininet()

    print("*** Creating nodes")
    h1 = net.addHost('h1', mac='00:00:00:00:00:01')
    h2 = net.addHost('h2', mac='00:00:00:00:00:02', ip='192.168.10.1/24')
    h3 = net.addHost('h3', mac='00:00:00:00:00:03', ip='192.168.10.2/24')
    h4 = net.addHost('h4', mac='00:00:00:00:00:04', ip='192.168.10.3/24')
    h5 = net.addHost('h5', mac='00:00:00:00:00:05')
    h6 = net.addHost('h6', mac='00:00:00:00:00:06', ip='192.168.20.1/24')
    h7 = net.addHost('h7', mac='00:00:00:00:00:07', ip='192.168.20.2/24')
    h8 = net.addHost('h8', mac='00:00:00:00:00:08', ip='192.168.20.3/24')
    h9 = net.addHost('h9', mac='00:00:00:00:00:09', ip='140.116.172.1/24')
    h10 = net.addHost('h10', mac='00:00:00:00:00:0A', ip='140.116.172.2/24')

    print("*** Creating links")
    net.addLink(h1, h2)
    net.addLink(h1, h3, intfName1='h1-eth1')
    net.addLink(h1, h4, intfName1='h1-eth2')
    net.addLink(h5, h6)
    net.addLink(h5, h7, intfName1='h5-eth1')
    net.addLink(h5, h8, intfName1='h5-eth2')
    net.addLink(h9, h10)
    net.addLink(h9, h1, intfName1='h9-eth1', intfName2='h1-eth3')
    net.addLink(h10, h5, intfName1='h10-eth1', intfName2='h5-eth3')

    print("*** Building network")
    net.build()

    print("*** Adding some commands")
    h1.cmd("sudo ifconfig h1-eth0 0")
    h1.cmd("sudo ifconfig h1-eth1 0")
    h1.cmd("sudo ifconfig h1-eth2 0")
    h1.cmd("sudo ifconfig h1-eth3 0")
    h1.cmd("sudo brctl addbr mybr")
    h1.cmd("sudo brctl addif mybr h1-eth0")
    h1.cmd("sudo brctl addif mybr h1-eth1")
    h1.cmd("sudo brctl addif mybr h1-eth2")
    h1.cmd("sudo brctl addif mybr h1-eth3")
    h1.cmd("sudo ifconfig mybr up")
    h5.cmd("sudo ifconfig h5-eth0 0")
    h5.cmd("sudo ifconfig h5-eth1 0")
    h5.cmd("sudo ifconfig h5-eth2 0")
    h5.cmd("sudo ifconfig h5-eth3 0")
    h5.cmd("sudo brctl addbr mybr2")
    h5.cmd("sudo brctl addif mybr2 h5-eth0")
    h5.cmd("sudo brctl addif mybr2 h5-eth1")
    h5.cmd("sudo brctl addif mybr2 h5-eth2")
    h5.cmd("sudo brctl addif mybr2 h5-eth3")
    h5.cmd("sudo ifconfig mybr2 up")
    h9.cmd("sudo echo 1 > /proc/sys/net/ipv4/ip_forward")
    h10.cmd("sudo echo 1 > /proc/sys/net/ipv4/ip_forward")
    h9.cmd("sudo ifconfig h9-eth1 192.168.10.254 netmask 255.255.255.0")
    h10.cmd("sudo ifconfig h10-eth1 192.168.20.254 netmask 255.255.255.0")

    # No NAT setting: 192.168.10.0/24 can not talk to 192.168.20.0
    # configure gre tunnel
    h9.cmd("sudo ip tunnel add netb mode gre remote 140.116.172.2 local 140.116.172.1 ttl 255")
    h9.cmd("sudo ip addr add 192.168.10.253 dev netb")
    h9.cmd("sudo ifconfig netb up")
    h9.cmd("sudo ip route add 192.168.20.0/24 via 192.168.10.253")
    h10.cmd("sudo ip tunnel add neta mode gre remote 140.116.172.1 local 140.116.172.2 ttl 255")
    h10.cmd("sudo ip addr add 192.168.20.253 dev neta")
    h10.cmd("sudo ifconfig neta up")
    h10.cmd("sudo ip route add 192.168.10.0/24 via 192.168.20.253")
    h2.cmd("sudo ip route add 192.168.20.0/24 via 192.168.10.253")
    h3.cmd("sudo ip route add 192.168.20.0/24 via 192.168.10.253")
    h4.cmd("sudo ip route add 192.168.20.0/24 via 192.168.10.253")
    h6.cmd("sudo ip route add 192.168.10.0/24 via 192.168.20.253")
    h7.cmd("sudo ip route add 192.168.10.0/24 via 192.168.20.253")
    h8.cmd("sudo ip route add 192.168.10.0/24 via 192.168.20.253")

    print("*** Running CLI")
    CLI(net)

    print("*** Stopping network")
    net.stop()


if __name__ == '__main__':
    setLogLevel('info')
    topology()
```

You can run the code with the command below (considering that the filename is greTunnel.py)
```
sudo python greTunnel.py
```

Then, you can try communicate h1 and h2 as follows:

```
mininet-wifi> h2 ping -c2 h3
```

as well as h2 and h8:

```
mininet-wifi> h2 ping -c2 h8
```

**Questions**:

01. What is the routing table of h2 and h8? Use the appropriated network commands and provide the outcome.
02. Through the analysis of the packets that cross h9 and h10, how many IP headers can you observe from each packet? Please explaing your answer based on how the GRE tunnel works.



## Troubleshooting
The expected commands and results after running the `greTunnel.py` code are depicted below (you have to modify `setLogLevel('info')` to `setLogLevel('debug')` - only use this troubleshooting step if you have problems with the _ping_ command):

```
*** errRun: ['grep', '-c', 'processor', '/proc/cpuinfo'] 
8
  0*** Setting resource limits
*** Creating nodes
*** errRun: ['which', 'mnexec'] 
/usr/bin/mnexec
  0*** errRun: ['which', 'ifconfig'] 
/sbin/ifconfig
  0_popen ['mnexec', '-cdn', 'env', 'PS1=\x7f', 'bash', '--norc', '--noediting', '-is', 'mininet:h1'] 21004*** h1 : ('unset HISTFILE; stty -echo; set +m',)
unset HISTFILE; stty -echo; set +m
_popen ['mnexec', '-cdn', 'env', 'PS1=\x7f', 'bash', '--norc', '--noediting', '-is', 'mininet:h2'] 21006*** h2 : ('unset HISTFILE; stty -echo; set +m',)
unset HISTFILE; stty -echo; set +m
_popen ['mnexec', '-cdn', 'env', 'PS1=\x7f', 'bash', '--norc', '--noediting', '-is', 'mininet:h3'] 21008*** h3 : ('unset HISTFILE; stty -echo; set +m',)
unset HISTFILE; stty -echo; set +m
_popen ['mnexec', '-cdn', 'env', 'PS1=\x7f', 'bash', '--norc', '--noediting', '-is', 'mininet:h4'] 21010*** h4 : ('unset HISTFILE; stty -echo; set +m',)
unset HISTFILE; stty -echo; set +m
_popen ['mnexec', '-cdn', 'env', 'PS1=\x7f', 'bash', '--norc', '--noediting', '-is', 'mininet:h5'] 21012*** h5 : ('unset HISTFILE; stty -echo; set +m',)
unset HISTFILE; stty -echo; set +m
_popen ['mnexec', '-cdn', 'env', 'PS1=\x7f', 'bash', '--norc', '--noediting', '-is', 'mininet:h6'] 21014*** h6 : ('unset HISTFILE; stty -echo; set +m',)
unset HISTFILE; stty -echo; set +m
_popen ['mnexec', '-cdn', 'env', 'PS1=\x7f', 'bash', '--norc', '--noediting', '-is', 'mininet:h7'] 21016*** h7 : ('unset HISTFILE; stty -echo; set +m',)
unset HISTFILE; stty -echo; set +m
_popen ['mnexec', '-cdn', 'env', 'PS1=\x7f', 'bash', '--norc', '--noediting', '-is', 'mininet:h8'] 21018*** h8 : ('unset HISTFILE; stty -echo; set +m',)
unset HISTFILE; stty -echo; set +m
_popen ['mnexec', '-cdn', 'env', 'PS1=\x7f', 'bash', '--norc', '--noediting', '-is', 'mininet:h9'] 21020*** h9 : ('unset HISTFILE; stty -echo; set +m',)
unset HISTFILE; stty -echo; set +m
_popen ['mnexec', '-cdn', 'env', 'PS1=\x7f', 'bash', '--norc', '--noediting', '-is', 'mininet:h10'] 21022*** h10 : ('unset HISTFILE; stty -echo; set +m',)
unset HISTFILE; stty -echo; set +m
*** Creating links
*** h1 : ('ip link add name h1-eth0 address 6a:7e:66:ae:4f:0a type veth peer name h2-eth0 address 82:38:ff:02:56:6a netns 21006',)

added intf h1-eth0 (0) to node h1
moving h1-eth0 into namespace for h1 
*** h1 : ('ifconfig', 'h1-eth0', 'up')

added intf h2-eth0 (0) to node h2
moving h2-eth0 into namespace for h2 
*** h2 : ('ifconfig', 'h2-eth0', 'up')
*** h1 : ('ip link add name h1-eth1 address 72:48:bc:41:0f:13 type veth peer name h3-eth0 address 7a:c1:fa:70:ef:b5 netns 21008',)

added intf h1-eth1 (1) to node h1
moving h1-eth1 into namespace for h1 
*** h1 : ('ifconfig', 'h1-eth1', 'up')

added intf h3-eth0 (0) to node h3
moving h3-eth0 into namespace for h3 
*** h3 : ('ifconfig', 'h3-eth0', 'up')
*** h1 : ('ip link add name h1-eth2 address c6:21:63:d3:95:e0 type veth peer name h4-eth0 address 02:55:fb:76:36:52 netns 21010',)

added intf h1-eth2 (2) to node h1
moving h1-eth2 into namespace for h1 
*** h1 : ('ifconfig', 'h1-eth2', 'up')

added intf h4-eth0 (0) to node h4
moving h4-eth0 into namespace for h4 
*** h4 : ('ifconfig', 'h4-eth0', 'up')
*** h5 : ('ip link add name h5-eth0 address 76:92:53:ae:90:4b type veth peer name h6-eth0 address 86:34:9b:60:78:c7 netns 21014',)

added intf h5-eth0 (0) to node h5
moving h5-eth0 into namespace for h5 
*** h5 : ('ifconfig', 'h5-eth0', 'up')

added intf h6-eth0 (0) to node h6
moving h6-eth0 into namespace for h6 
*** h6 : ('ifconfig', 'h6-eth0', 'up')
*** h5 : ('ip link add name h5-eth1 address 52:7e:37:cc:3c:37 type veth peer name h7-eth0 address b2:33:30:e1:22:96 netns 21016',)

added intf h5-eth1 (1) to node h5
moving h5-eth1 into namespace for h5 
*** h5 : ('ifconfig', 'h5-eth1', 'up')

added intf h7-eth0 (0) to node h7
moving h7-eth0 into namespace for h7 
*** h7 : ('ifconfig', 'h7-eth0', 'up')
*** h5 : ('ip link add name h5-eth2 address c6:75:da:11:69:6e type veth peer name h8-eth0 address de:f0:3f:c6:20:80 netns 21018',)

added intf h5-eth2 (2) to node h5
moving h5-eth2 into namespace for h5 
*** h5 : ('ifconfig', 'h5-eth2', 'up')

added intf h8-eth0 (0) to node h8
moving h8-eth0 into namespace for h8 
*** h8 : ('ifconfig', 'h8-eth0', 'up')
*** h9 : ('ip link add name h9-eth0 address 26:04:b0:1d:07:67 type veth peer name h10-eth0 address be:3c:1e:a0:6d:fc netns 21022',)

added intf h9-eth0 (0) to node h9
moving h9-eth0 into namespace for h9 
*** h9 : ('ifconfig', 'h9-eth0', 'up')

added intf h10-eth0 (0) to node h10
moving h10-eth0 into namespace for h10 
*** h10 : ('ifconfig', 'h10-eth0', 'up')
*** h9 : ('ip link add name h9-eth1 address a2:13:26:52:64:2f type veth peer name h1-eth3 address ca:d1:6b:28:de:31 netns 21004',)

added intf h9-eth1 (1) to node h9
moving h9-eth1 into namespace for h9 
*** h9 : ('ifconfig', 'h9-eth1', 'up')

added intf h1-eth3 (3) to node h1
moving h1-eth3 into namespace for h1 
*** h1 : ('ifconfig', 'h1-eth3', 'up')
*** h10 : ('ip link add name h10-eth1 address ea:5c:68:63:66:f3 type veth peer name h5-eth3 address 2e:19:6d:73:2c:50 netns 21012',)

added intf h10-eth1 (1) to node h10
moving h10-eth1 into namespace for h10 
*** h10 : ('ifconfig', 'h10-eth1', 'up')

added intf h5-eth3 (3) to node h5
moving h5-eth3 into namespace for h5 
*** h5 : ('ifconfig', 'h5-eth3', 'up')
*** Building network
*** Configuring hosts
h1 *** h1 : ('ifconfig', 'h1-eth0', 'down')
*** h1 : ('ifconfig', 'h1-eth0', 'hw', 'ether', '00:00:00:00:00:01')
*** h1 : ('ifconfig', 'h1-eth0', 'up')
*** h1 : ('ifconfig', 'h1-eth0', '10.0.0.1/8', 'up')
*** h1 : ('ifconfig lo up',)
h2 *** h2 : ('ifconfig', 'h2-eth0', 'down')
*** h2 : ('ifconfig', 'h2-eth0', 'hw', 'ether', '00:00:00:00:00:02')
*** h2 : ('ifconfig', 'h2-eth0', 'up')
*** h2 : ('ifconfig', 'h2-eth0', '192.168.10.1/24', 'up')
*** h2 : ('ifconfig lo up',)
h3 *** h3 : ('ifconfig', 'h3-eth0', 'down')
*** h3 : ('ifconfig', 'h3-eth0', 'hw', 'ether', '00:00:00:00:00:03')
*** h3 : ('ifconfig', 'h3-eth0', 'up')
*** h3 : ('ifconfig', 'h3-eth0', '192.168.10.2/24', 'up')
*** h3 : ('ifconfig lo up',)
h4 *** h4 : ('ifconfig', 'h4-eth0', 'down')
*** h4 : ('ifconfig', 'h4-eth0', 'hw', 'ether', '00:00:00:00:00:04')
*** h4 : ('ifconfig', 'h4-eth0', 'up')
*** h4 : ('ifconfig', 'h4-eth0', '192.168.10.3/24', 'up')
*** h4 : ('ifconfig lo up',)
h5 *** h5 : ('ifconfig', 'h5-eth0', 'down')
*** h5 : ('ifconfig', 'h5-eth0', 'hw', 'ether', '00:00:00:00:00:05')
*** h5 : ('ifconfig', 'h5-eth0', 'up')
*** h5 : ('ifconfig', 'h5-eth0', '10.0.0.5/8', 'up')
*** h5 : ('ifconfig lo up',)
h6 *** h6 : ('ifconfig', 'h6-eth0', 'down')
*** h6 : ('ifconfig', 'h6-eth0', 'hw', 'ether', '00:00:00:00:00:06')
*** h6 : ('ifconfig', 'h6-eth0', 'up')
*** h6 : ('ifconfig', 'h6-eth0', '192.168.20.1/24', 'up')
*** h6 : ('ifconfig lo up',)
h7 *** h7 : ('ifconfig', 'h7-eth0', 'down')
*** h7 : ('ifconfig', 'h7-eth0', 'hw', 'ether', '00:00:00:00:00:07')
*** h7 : ('ifconfig', 'h7-eth0', 'up')
*** h7 : ('ifconfig', 'h7-eth0', '192.168.20.2/24', 'up')
*** h7 : ('ifconfig lo up',)
h8 *** h8 : ('ifconfig', 'h8-eth0', 'down')
*** h8 : ('ifconfig', 'h8-eth0', 'hw', 'ether', '00:00:00:00:00:08')
*** h8 : ('ifconfig', 'h8-eth0', 'up')
*** h8 : ('ifconfig', 'h8-eth0', '192.168.20.3/24', 'up')
*** h8 : ('ifconfig lo up',)
h9 *** h9 : ('ifconfig', 'h9-eth0', 'down')
*** h9 : ('ifconfig', 'h9-eth0', 'hw', 'ether', '00:00:00:00:00:09')
*** h9 : ('ifconfig', 'h9-eth0', 'up')
*** h9 : ('ifconfig', 'h9-eth0', '140.116.172.1/24', 'up')
*** h9 : ('ifconfig lo up',)
h10 *** h10 : ('ifconfig', 'h10-eth0', 'down')
*** h10 : ('ifconfig', 'h10-eth0', 'hw', 'ether', '00:00:00:00:00:0A')
*** h10 : ('ifconfig', 'h10-eth0', 'up')
*** h10 : ('ifconfig', 'h10-eth0', '140.116.172.2/24', 'up')
*** h10 : ('ifconfig lo up',)

*** Adding some commands
*** h1 : ('sudo ifconfig h1-eth0 0',)
*** h1 : ('sudo ifconfig h1-eth1 0',)
*** h1 : ('sudo ifconfig h1-eth2 0',)
*** h1 : ('sudo ifconfig h1-eth3 0',)
*** h1 : ('sudo brctl addbr mybr',)
*** h1 : ('sudo brctl addif mybr h1-eth0',)
*** h1 : ('sudo brctl addif mybr h1-eth1',)
*** h1 : ('sudo brctl addif mybr h1-eth2',)
*** h1 : ('sudo brctl addif mybr h1-eth3',)
*** h1 : ('sudo ifconfig mybr up',)
*** h5 : ('sudo ifconfig h5-eth0 0',)
*** h5 : ('sudo ifconfig h5-eth1 0',)
*** h5 : ('sudo ifconfig h5-eth2 0',)
*** h5 : ('sudo ifconfig h5-eth3 0',)
*** h5 : ('sudo brctl addbr mybr2',)
*** h5 : ('sudo brctl addif mybr2 h5-eth0',)
*** h5 : ('sudo brctl addif mybr2 h5-eth1',)
*** h5 : ('sudo brctl addif mybr2 h5-eth2',)
*** h5 : ('sudo brctl addif mybr2 h5-eth3',)
*** h5 : ('sudo ifconfig mybr2 up',)
*** h9 : ('sudo echo 1 > /proc/sys/net/ipv4/ip_forward',)
*** h10 : ('sudo echo 1 > /proc/sys/net/ipv4/ip_forward',)
*** h9 : ('sudo ifconfig h9-eth1 192.168.10.254 netmask 255.255.255.0',)
*** h10 : ('sudo ifconfig h10-eth1 192.168.20.254 netmask 255.255.255.0',)
*** h9 : ('sudo ip tunnel add netb mode gre remote 140.116.172.2 local 140.116.172.1 ttl 255',)
*** h9 : ('sudo ip addr add 192.168.10.253 dev netb',)
*** h9 : ('sudo ifconfig netb up',)
*** h9 : ('sudo ip route add 192.168.20.0/24 via 192.168.10.253',)
*** h10 : ('sudo ip tunnel add neta mode gre remote 140.116.172.1 local 140.116.172.2 ttl 255',)
*** h10 : ('sudo ip addr add 192.168.20.253 dev neta',)
*** h10 : ('sudo ifconfig neta up',)
*** h10 : ('sudo ip route add 192.168.10.0/24 via 192.168.20.253',)
*** h2 : ('sudo ip route add 192.168.20.0/24 via 192.168.10.253',)
*** h3 : ('sudo ip route add 192.168.20.0/24 via 192.168.10.253',)
*** h4 : ('sudo ip route add 192.168.20.0/24 via 192.168.10.253',)
*** h6 : ('sudo ip route add 192.168.10.0/24 via 192.168.20.253',)
*** h7 : ('sudo ip route add 192.168.10.0/24 via 192.168.20.253',)
*** h8 : ('sudo ip route add 192.168.10.0/24 via 192.168.20.253',)
*** Running CLI
*** Starting CLI:
*** errRun: ['stty', 'echo', 'sane', 'intr', '^C'] 
```

