# TP4 : DHCP

# I. DHCP Client

🌞 **Déterminer**

```
PS C:\Users\fayer> ipconfig /all
Serveur DHCP . . . . . . . . . . . . . : 10.33.79.254
Bail obtenu. . . . . . . . . . . . . . : vendredi 27 octobre 2023 10:42:51
Bail expirant. . . . . . . . . . . . . : samedi 28 octobre 2023 10:42:49
```

🌞 **Capturer un échange DHCP**

[Capture DHCP](./tp4_dhcp_client.pcapng)

🌞 **Analyser la capture Wireshark**

C'est la trame `Offer` qui contient les informations

## II. Serveur DHCP

### 3. Setup topologie

➜ **Mettez en place la topologie**

🌞 **Preuve de mise en place**

```
[bjorn@dhcp ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=113 time=21.5 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=113 time=19.3 ms
```

```
[bjorn@node2 ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=113 time=18.7 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=113 time=25.4 ms
```

```
[bjorn@node2 ~]$ traceroute 64.233.160.0
traceroute to 64.233.160.0 (64.233.160.0), 30 hops max, 60 byte packets
gateway (10.4.1.254)  1.120 ms  2.513 ms  2.467 ms
10.0.3.2 (10.0.3.2)  4.857 ms  5.018 ms  4.355 ms
```

### 4. Serveur DHCP

🌞 **Rendu**

```
[bjorn@dhcp ~]$sudo nano /etc/resolv.conf
[bjorn@dhcp ~]$ping ynov.com
[bjorn@dhcp ~]$sudo dnf -y install dhcp-server
[bjorn@dhcp ~]$sudo nano /etc/dhcp/dhcpd.conf
```

```
[bjorn@dhcp ~]$ systemctl status dhcpd
● dhcpd.service - DHCPv4 Server Daemon
     Loaded: loaded (/usr/lib/systemd/system/dhcpd.service; enabled; preset: disabled)
     Active: active (running) since Sun 2023-11-05 00:15:26 CET; 4min 41s ago
       Docs: man:dhcpd(8)
             man:dhcpd.conf(5)
   Main PID: 1501 (dhcpd)
     Status: "Dispatching packets..."
      Tasks: 1 (limit: 4672)
     Memory: 4.6M
        CPU: 29ms
     CGroup: /system.slice/dhcpd.service
             └─1501 /usr/sbin/dhcpd -f -cf /etc/dhcp/dhcpd.conf -user dhcpd -group dhcpd --no-pid
```

```
[bjorn@dhcp ~]$ sudo cat /etc/dhcp/dhcpd.conf
[sudo] password for bjorn:
# Configuration serveur DHCP
# create new specify domain name
option domain-name     "tp4.dhcp";
# specify DNS server's hostname or IP address
option domain-name-servers     dlp.tp4.dhcp;
# default lease time
default-lease-time 600;
# max lease time
max-lease-time 7200;
# this DHCP server to be declared valid
authoritative;
# specify network address and subnetmask
subnet 10.4.1.0 netmask 255.255.255.0 {
    # specify the range of lease IP address
    range dynamic-bootp 10.4.1.137 10.4.1.237;
    # specify broadcast address
    option broadcast-address 10.4.1.255;
    # specify gateway
    option routers 10.4.1.254;
}
```

### 5. Client DHCP

🌞 **Prouvez que**

```
[bjorn@node1 ~]$ ip a
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:6b:5d:e3 brd ff:ff:ff:ff:ff:ff
    inet 10.4.1.137/24 brd 10.4.1.255 scope global dynamic noprefixroute enp0s3
```

- `node1.tp4.b1` a enregistré un bail DHCP
  - déterminer la date exacte de création du bail
  - déterminer la date exacte d'expiration
  - déterminer l'adresse IP du serveur DHCP (depuis `node1.tp4.b1` : il a enregistré l'adresse IP du serveur DHCP)

```
[bjorn@node1 ~]$ ping 10.4.1.254
PING 10.4.1.254 (10.4.1.254) 56(84) bytes of data.
64 bytes from 10.4.1.254: icmp_seq=1 ttl=64 time=2.65 ms
64 bytes from 10.4.1.254: icmp_seq=2 ttl=64 time=2.23 ms
64 bytes from 10.4.1.254: icmp_seq=3 ttl=64 time=2.32 ms
```

```
[bjorn@node1 ~]$  ping 10.4.1.12
PING 10.4.1.12 (10.4.1.12) 56(84) bytes of data.
64 bytes from 10.4.1.12: icmp_seq=1 ttl=64 time=2.51 ms
64 bytes from 10.4.1.12: icmp_seq=2 ttl=64 time=1.59 ms
64 bytes from 10.4.1.12: icmp_seq=3 ttl=64 time=2.35 ms
```

🌞 **Bail DHCP serveur**

```cat /var/lib/dhcpd/dhcpd.leases```

```
lease 10.4.1.137 {
  starts 6 2023/11/04 23:59:36;
  ends 0 2023/11/05 00:09:36;
  cltt 6 2023/11/04 23:59:36;
  binding state active;
  next binding state free;
  rewind binding state free;
  hardware ethernet 08:00:27:6b:5d:e3;
  uid "\001\010\000'k]\343";
}
```

### 6. Options DHCP

🌞 **Nouvelle conf !**

```
[bjorn@dhcp ~]$ sudo cat /etc/dhcp/dhcpd.conf
# create new
# specify domain name
option domain-name     "tp4.dhcp";
# specify DNS server's hostname or IP address
option domain-name-servers     1.1.1.1;
# default lease time
default-lease-time 21600;
# max lease time
max-lease-time 24000;
# this DHCP server to be declared valid
authoritative;
# specify network address and subnetmask
subnet 10.4.1.0 netmask 255.255.255.0 {
    # specify the range of lease IP address
    range dynamic-bootp 10.4.1.137 10.4.1.237;
    # specify broadcast address
    option broadcast-address 10.4.1.255;
    # specify gateway
    option routers 10.4.1.254;
}
```

🌞 **Test !**

- redemandez une IP avec le client `node1.tp4.b1`
- prouvez-que :
  - vous avez enregistré l'adresse d'un serveur DNS
    - sous Linux, on consulte le serveur DNS actuel en affichant le contenu du fichier `/etc/resolv.conf`
  - vous avez une nouvelle route par défaut qui a été récupérée dynamiquement
  - la durée de votre bail DHCP est bien de 6 heures
- prouvez que vous avez un accès Internet après cet échange DHCP

🌞 **Capture Wireshark**

- utilisez `tcpdump` pour capturer un échange DHCP complet entre `node1.tp4.b1` et `dhcp.tp4.b1`

🦈 **`tp4_dhcp_server.pcapng`**

➜ **Un vrai serveur DHCP** qui donne tout ce qu'il faut aux clients pour qu'ils aient un accès au LAN (une adresse IP) et un accès internet en plus (l'adresse de la passerelle et l'adresse d'un serveur DNS joignable).
