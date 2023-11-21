# TP6 : Un LAN maîtrisé meo ?

## II. Serveur DNS

### 2. Setup

🌞 **Dans le rendu, je veux**

```shell
options {
        listen-on port 53 { 127.0.0.1; any; };
        listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        secroots-file   "/var/named/data/named.secroots";
        recursing-file  "/var/named/data/named.recursing";
        allow-query     { localhost; any; };
        allow-query-cache       { localhost; any; };

        recursion yes;

        dnssec-validation yes;

        managed-keys-directory "/var/named/dynamic";
        geoip-directory "/usr/share/GeoIP";

        pid-file "/run/named/named.pid";
        session-keyfile "/run/named/session.key";

        include "/etc/crypto-policies/back-ends/bind.config";
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

zone "tp6.b1" IN {
        type master;
        file "tp6.b1.db";
        allow-update { none; };
        allow-query {any; };
};

zone "1.4.10in-addr.arpa" IN {
        type master;
        file "tp6.b1.rev";
        allow-update    { none; };
        allow-query     { any; };
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```

```shell
[bjorn@dns ~]$ sudo cat /var/named/tp6.b1.db
$TTL 86400
@ IN SOA dns.tp6.b1. admin.tp6.b1. (
    2019061800 ;Serial
    3600 ;Refresh
    1800 ;Retry
    604800 ;Expire
    86400 ;Minimum TTL
)

; Infos sur le serveur DNS lui même (NS = NameServer)
@ IN NS dns.tp6.b1.

dns       IN A 10.6.1.101
john      IN A 10.6.1.11
```

```shell
[bjorn@dns ~]$ sudo cat /var/named/tp6.b1.rev
$TTL 86400
@ IN SOA dns.tp6.b1. admin.tp6.b1. (
    2019061800 ;Serial
    3600 ;Refresh
    1800 ;Retry
    604800 ;Expire
    86400 ;Minimum TTL
)

@ IN NS dns.tp6.b1.

101 IN PTR dns.tp6.b1.
11 IN PTR john.tp6.b1.
```

```shell
[bjorn@dns ~]$ sudo systemctl status named
● named.service - Berkeley Internet Name Domain (DNS)
     Loaded: loaded (/usr/lib/systemd/system/named.service; enabled; preset: disabled)
     Active: active (running) since Fri 2023-11-17 16:05:10 CET; 7min ago
   Main PID: 2073 (named)
      Tasks: 5 (limit: 4672)
     Memory: 17.3M
        CPU: 194ms
     CGroup: /system.slice/named.service
             └─2073 /usr/sbin/named -u named -c /etc/named.conf
```

```shell
[bjorn@dns ~]$ ss -nl
tcp        LISTEN      0           10                                                10.6.1.101:53                           0.0.0.0:*
```

🌞 **Ouvrez le bon port dans le firewall**

```shell
[bjorn@dns ~]$ history
38  sudo firewall-cmd --add-port=53/udp --permanent
39  sudo firewall-cmd --reload
```

### 3. Test

🌞 **Sur la machine `john.tp6.b1`**

```shell
[bjorn@jhon ~]$ dig john.tp6

; <<>> DiG 9.16.23-RH <<>> john.tp6
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 46063
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1
```

```shell
[bjorn@jhon ~]$ dig dns.tp6

; <<>> DiG 9.16.23-RH <<>> dns.tp6
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 5871
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1
```

```shell
[bjorn@jhon ~]$ dig ynov.com
;; ANSWER SECTION:
ynov.com.               300     IN      A       104.26.10.233
ynov.com.               300     IN      A       104.26.11.233
ynov.com.               300     IN      A       172.67.74.226
```

🌞 **Sur votre PC**

```shell
PS C:\Users\fayer> nslookup john.tp6.b1 dns.tp6
```

[tp6_dns.pcapng](./tp6_dns.pcapng)

```shell
[bjorn@dns ~]$ sudo tcpdump -i enp0s3 -c 10 -n -s 0 port 53 -w tp6_dns.pcapng
```

## III. Serveur DHCP

🌞 **Installer un serveur DHCP**

```shell
[bjorn@dhcp ~]$ sudo cat /etc/dhcp/dhcpd.conf
# create new
# specify domain name
option domain-name     "dhcp.tp6";
# specify DNS server's hostname or IP address
option domain-name-servers 10.6.1.101;
# default lease time
default-lease-time 600;
# max lease time
max-lease-time 7200;
# this DHCP server to be declared valid
authoritative;
# specify network address and subnetmask
subnet 10.6.1.0 netmask 255.255.255.0 {
    # specify the range of lease IP address
    range dynamic-bootp 10.6.1.13 10.6.1.37;
    # specify broadcast address
    option broadcast-address 10.6.1.255;
    # specify gateway
    option routers 10.6.1.254;
}
```

🌞 **Test avec `john.tp6.b1`**

```shell
[bjorn@jhon ~]$ ip a
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:d5:98:24 brd ff:ff:ff:ff:ff:ff
    inet 10.6.1.13/24 brd 10.6.1.255 scope global dynamic enp0s3
```

```shell
[bjorn@jhon ~]$ ip r s
default via 10.6.1.254 dev enp0s3
```

```shell
[bjorn@jhon ~]$ cat /etc/resolv.conf
; generated by /usr/sbin/dhclient-script
search dhcp.tp6 tp6
nameserver 10.6.1.101
```

```shell
[bjorn@jhon ~]$ ping 1.1.1.1
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=55 time=22.6 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=55 time=21.9 ms
64 bytes from 1.1.1.1: icmp_seq=3 ttl=55 time=22.7 ms
```

🌞 **Requête web avec `john.tp6.b1`**

[tp6_web.pcapng](./tp6_web.pcapng)


## IV. Bonus : Serveur Web HTTPS

Cette partie est un **bonus** : juste pour les téméraires donc.

L'objectif est simple : monter un serveur web comme dans le TP précédent, mais le servir avec HTTPS et pas juste HTTP, afin de fournir une connexion de confiance aux visiteurs du site.

| Name            | LAN1 `10.6.1.0/24` |
| --------------- | ------------------ |
| `router.tp6.b1` | `10.6.1.254`       |
| `john.tp6.b1`   | `DHCP`             |
| `dns.tp6.b1`    | `10.6.1.101`       |
| `dhcp.tp6.b1`   | `10.6.1.102`       |
| `web.tp6.b1`    | `10.6.1.103`       |

Pour passer de HTTP à HTTPS avec un serveur NGINX ça peut se résumer à :

- taper une commande pour générer une clé et un certificat
- ajouter 3 lignes de conf dans la conf existante

C'est si peu que je vous laisse google "simple nginx https configuration" par exemple ! Hésitez-pas à me demander si besoin d'aide, mais c'est un exercice classique et formateur :)

➜ **On doit donc pouvoir visiter votre site web localement, depuis les VMs, en tapant `https://web.tp6.b1`.**
