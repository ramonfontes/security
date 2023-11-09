---
tags: security-tutorials
---

# VPN/SSH

**In this short demo you will:** 
- How to run a simple SSH network topology
- How to run a simple VPN network topology

**Requirements:** 
- Containernet - https://github.com/ramonfontes/containernet


Network topology:
```
                                                               srv1
                                                           (10.200.0.2)
                                                                /
                                                               /
client (10.201.0.1) - s1 - (10.201.0.100) r1 (10.200.0.100) - s2 - vpn (10.200.0.1)
                                                               \
                                                                \
                                                           (10.200.0.3)
                                                                srv2
```     

The script below produces the network topology illustrated above.

```python=
#!/usr/bin/python

'''@author: Ramon Fontes
   @email: ramon.fontes@imd.ufrn.br
'''

import os

from containernet.net import Containernet
from containernet.node import DockerSta
from containernet.cli import CLI
from containernet.term import makeTerm
from mininet.log import info, setLogLevel


def topology():
    "Create a network."
    DISPLAY_ID = 1
    net = Containernet(ipBase='10.200.0.0/24')

    os.system('sudo xhost +local:docker')
    os.system('export DISPLAY=:{}'.format(DISPLAY_ID))

    info("*** Creating nodes\n")
    vpn1 = net.addDocker('vpn', dimage="ramonfontes/vpn", cpu_shares=20,
                         volumes=['/tmp/.X11-unix:/tmp/.X11-unix:rw'],
                         environment={'DISPLAY':":{}".format(DISPLAY_ID)}, mac='00:00:00:00:00:01')
    srv1 = net.addDocker('srv1', dimage="ramonfontes/seguranca", cpu_shares=20,
                            volumes=['/tmp/.X11-unix:/tmp/.X11-unix:rw'],
                            environment={'DISPLAY':":{}".format(DISPLAY_ID)}, mac='00:00:00:00:00:04')
    srv2 = net.addDocker('srv2', dimage="ramonfontes/seguranca", cpu_shares=20,
                          volumes=['/tmp/.X11-unix:/tmp/.X11-unix:rw'],
                          environment={'DISPLAY':":{}".format(DISPLAY_ID)}, mac='00:00:00:00:00:05')
    client1 = net.addDocker('client', dimage="ramonfontes/seguranca", cpu_shares=20,
                            volumes=['/tmp/.X11-unix:/tmp/.X11-unix:rw'],
                            environment={'DISPLAY':":{}".format(DISPLAY_ID)}, mac='00:00:00:00:00:02',
                            ip='10.201.0.1/24')
    s1 = net.addSwitch('s1', failMode='standalone')
    s2 = net.addSwitch('s2', failMode='standalone')
    r1 = net.addHost('r1', ip='10.201.0.100/24')

    info("*** Creating Links\n")
    net.addLink(client1, s1)
    net.addLink(r1, s1)
    net.addLink(r1, s2)
    net.addLink(vpn1, s2)
    net.addLink(srv1, s2)
    net.addLink(srv2, s2)

    info("*** Starting network\n")
    net.build()
    s1.start([])
    s2.start([])

    client1.cmd('route add default gw 10.201.0.100')
    srv1.cmd('ip route del 0/0')
    srv2.cmd('ip route del 0/0')
    vpn1.cmd('ip route del 0/0')
    srv1.cmd('route add default gw 10.200.0.100')
    srv2.cmd('route add default gw 10.200.0.100')
    vpn1.cmd('route add default gw 10.200.0.100')
    r1.cmd('ifconfig r1-eth1 10.200.0.100/24')

    r1.cmd('iptables -F')
    r1.cmd('iptables -A FORWARD -j ACCEPT')
    r1.cmd('iptables -A FORWARD -p tcp --dport 22 -j ACCEPT')
    r1.cmd('iptables -A FORWARD -p tcp --sport 22 -m state --state ESTABLISHED,RELATED -j ACCEPT')
    r1.cmd('iptables -t nat -A PREROUTING -p tcp --dport 22 -j DNAT --to 10.200.0.1:22')
    r1.cmd('iptables -P INPUT DROP')
    r1.cmd('iptables -P OUTPUT DROP')
    r1.cmd('iptables -P FORWARD DROP')
    vpn1.cmd("echo \'10.200.0.1 vpn1\' > /etc/hosts && /etc/init.d/ssh start")
    srv1.cmd("echo \'10.200.0.2 srv1\' > /etc/hosts && service apache2 start")
    srv2.cmd("echo \'10.200.0.3 srv2\' > /etc/hosts && service apache2 start")

    info("*** Running CLI\n")
    CLI(net)

    info("*** Stopping network\n")
    net.stop()


if __name__ == '__main__':
    setLogLevel('info')
    topology()
```

You can run the code with the command below (considering that the filename is vpn-ssh.py)
```
sudo python vpn-ssy.py
```

