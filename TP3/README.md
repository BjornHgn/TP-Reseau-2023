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
```
[bjorn@localhost ~]$ sudo ip neigh flush all
```
🦈 **Capture réseau `tp3_arp.pcapng`** qui contient un ARP request et un ARP reply

# II. Routage

🌞**Ajouter les routes statiques nécessaires pour que `john` et `marcel` puissent se `ping`**
```
[bjorn@VMJhon ~]$ sudo ip route add 10.3.2.0/24 via 10.3.1.254 dev enp0s3
[bjorn@VMJhon ~]$ ping 10.3.2.12
PING 10.3.2.12 (10.3.2.12) 56(84) bytes of data.
64 bytes from 10.3.2.12: icmp_seq=1 ttl=63 time=4.00 ms
64 bytes from 10.3.2.12: icmp_seq=2 ttl=63 time=5.88 ms
64 bytes from 10.3.2.12: icmp_seq=3 ttl=63 time=4.45 ms
```
```
[bjorn@Marcel ~]$ sudo ip route add 10.3.1.0/24 via 10.3.2.254 dev enp0s3
[bjorn@Marcel ~]$ ping 10.3.1.11
PING 10.3.1.11 (10.3.1.11) 56(84) bytes of data.
64 bytes from 10.3.1.11: icmp_seq=1 ttl=63 time=8.26 ms
64 bytes from 10.3.1.11: icmp_seq=2 ttl=63 time=2.18 ms
```
## 2. Analyse de trames

🌞**Analyse des échanges ARP**


| ordre | type trame  | IP source | MAC source                | IP destination | MAC destination            |
| ----- | ----------- | --------- | ------------------------- | -------------- | -------------------------- |
| 1     | Requête ARP | x         | `Jhon` `08:00:27:a9:cc:35` | x              | Broadcast `FF:FF:FF:FF:FF` |
| 2     | Réponse ARP | x         | `marcel` `08:00:27:4a:21:7f` | x              | `08:00:27:a9:cc:35` |
| ...   | ...         | ...       | ...                       |                |                            |
| ?     | Ping        | 10.3.1.11 | `08:00:27:a9:cc:35`  | 10.3.2.12   |`marcel` `08:00:27:4a:21:7f`           |
| ?     | Pong        | 10.3.2.12 | `08:00:27:4a:21:7f`  | 10.3.1.11   | `Jhon` `08:00:27:a9:cc:35`          |

## 3. Accès internet

🌞**Donnez un accès internet à vos machines** - config routeur


🌞**Donnez un accès internet à vos machines** - config clients


🌞**Analyse de trames**

| ordre | type trame | IP source            | MAC source                | IP destination | MAC destination |     |
| ----- | ---------- | -------------------- | ------------------------- | -------------- | --------------- | --- |
| 1     | ping       | `marcel` `10.3.1.12` | `marcel` `AA:BB:CC:DD:EE` | `8.8.8.8`      | ?               |     |
| 2     | pong       | ...                  | ...                       | ...            | ...             | ... |

🦈 **Capture réseau `tp3_routage_lan1.pcapng`**  
🦈 **Capture réseau `tp3_routage_lan2.pcapng`**