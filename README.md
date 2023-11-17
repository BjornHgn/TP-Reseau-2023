# TP6 : Un LAN maîtrisé meo ?

# II. Serveur DNS

## 2. Setup

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
38  sudo firewall-cmd --add-port=53/tcp --permanent
39  sudo firewall-cmd --reload
```

## 3. Test

🌞 **Sur la machine `john.tp6.b1`**

```shell
[bjorn@jhon ~]$ ping dns.tp6.b1
PING dns.tp6.b1 (10.6.1.101) 56(84) bytes of data.
64 bytes from 10.6.1.101 (10.6.1.101): icmp_seq=1 ttl=64 time=2.57 ms
64 bytes from 10.6.1.101 (10.6.1.101): icmp_seq=2 ttl=64 time=1.34 ms
64 bytes from 10.6.1.101 (10.6.1.101): icmp_seq=3 ttl=64 time=3.02 ms
```

```shell
[bjorn@jhon ~]$ ping www.ynov.com
PING www.ynov.com (104.26.11.233) 56(84) bytes of data.
64 bytes from 104.26.11.233 (104.26.11.233): icmp_seq=1 ttl=55 time=21.7 ms
64 bytes from 104.26.11.233 (104.26.11.233): icmp_seq=2 ttl=55 time=21.3 ms
64 bytes from 104.26.11.233 (104.26.11.233): icmp_seq=3 ttl=55 time=21.4 ms
```

🌞 **Sur votre PC**

- utilisez une commande pour résoudre le nom `john.tp6.b1` en utilisant `10.6.1.201` comme serveur DNS


🦈 **Capture `tp6_dns.pcapng`**

- une requête DNS vers le nom `john.tp6.b1` ainsi que la réponse
- prenez deux minutes pour regarder le contenu de la trame
- les requêtes DNS passent en clair, rien n'est chiffré : on voit tout !
- capture à réaliser avec une commande `tcpdump` sur l'un des VMs

# III. Serveur DHCP

> *On prend les mêmes et on r'commence.*

![Topo 3](./img/topo3.svg)

| Name            | LAN1 `10.6.1.0/24` |
| --------------- | ------------------ |
| `router.tp6.b1` | `10.6.1.254`       |
| `john.tp6.b1`   | `DHCP`             |
| `dns.tp6.b1`    | `10.6.1.101`       |
| `dhcp.tp6.b1`   | `10.6.1.102`       |

🖥️ **Machine `dhcp.tp6.b1`**

- déroulez la checklisteuh
- indiquez votre propre serveur DNS pour la résolution de noms

🌞 **Installer un serveur DHCP**

- référez-vous à vos TPs précédents
- il doit attribuer des adresses IP entre `10.6.1.13` et `10.6.1.37`
- il doit informer les clients de l'adresse de la passerelle du réseau : `10.6.1.254`
- il doit informer les clients de l'adresse du serveur DNS du réseau : `10.6.1.101`

🌞 **Test avec `john.tp6.b1`**

- enlevez-lui son IP statique, et récupérez une IP en DHCP
- prouvez-que
  - vous avez une IP dans la bonne range
  - vous avez votre routeur configuré automatiquement comme passerelle
  - vous avez votre serveur DNS automatiquement configuré
  - vous avez un accès internet

🌞 **Requête web avec `john.tp6.b1`**

- utilisez la commande `curl` pour faire une requête HTTP vers le site de votre choix
- par exemple `curl www.ynov.com`

🦈 **Capture `tp6_web.pcapng`**

- capture à réaliser avec une commande `tcpdump` sur l'une des machines
- je veux voir dans la capture :
  - la requête DNS qui est effectuée automatiquement
  - la réponse DNS
  - le 3-way handshake TCP vers le serveur web
  - l'échange HTTP/HTTPS

> *Hé. Tu le sens qu'on se rapproche vraiment d'une situation réelle là ?*

# IV. Bonus : Serveur Web HTTPS

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