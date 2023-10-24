# I. ARP

## 1. Echange ARP

🌞**Générer des requêtes ARP**
```
[bjorn@localhost ~]$ ping 10.3.1.11
PING 10.3.1.11 (10.3.1.11) 56(84) bytes of data.
64 bytes from 10.3.1.11: icmp_seq=1 ttl=64 time=3.63 ms
64 bytes from 10.3.1.11: icmp_seq=2 ttl=64 time=2.18 ms
64 bytes from 10.3.1.11: icmp_seq=3 ttl=64 time=1.37 ms
```
```
[bjorn@localhost ~]$ ping 10.3.1.12
PING 10.3.1.12 (10.3.1.12) 56(84) bytes of data.
64 bytes from 10.3.1.12: icmp_seq=1 ttl=64 time=0.977 ms
64 bytes from 10.3.1.12: icmp_seq=2 ttl=64 time=1.69 ms
```
Table ARP
```
[bjorn@localhost ~]$ ip neigh show 10.3.1.11
10.3.1.11 dev enp0s3 lladdr 08:00:27:a9:cc:35 STALE
```
```
[bjorn@localhost ~]$ ip neigh show 10.3.1.12
10.3.1.12 dev enp0s3 lladdr 08:00:27:4a:21:7f STALE
```
Adresse MAC de Jhon: ```08:00:27:a9:cc:35```

Adresse MAC de Marcel: ```08:00:27:4a:21:7f```
```
[bjorn@localhost ~]$ ip neigh show 10.3.1.12
10.3.1.12 dev enp0s3 lladdr 08:00:27:4a:21:7f STALE
```
```
[bjorn@localhost ~]$ ip a
link/ether 08:00:27:4a:21:7f
```
## 2. Analyse de trames

🌞**Analyse de trames**
```
[bjorn@localhost ~]$ sudo tcpdump
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode

listening on enp0s3, link-type EN10MB (Ethernet), snapshot length 262144 bytes

14:54:02.962176 IP localhost.localdomain.ssh > 10.3.1.1.58578: Flags [P.], seq 4059327614:4059327674, ack 1251598489, win 501, length 60
```

🦈 **Capture réseau `tp3_arp.pcapng`** qui contient un ARP request et un ARP reply
