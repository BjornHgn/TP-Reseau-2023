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

[Capture dhcp](./TP4/tp4_dhcp_client.pcapng)

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
- un `systemctl status dhcpd` qui affiche l'état du serveur (je dois voir qu'il est actif)
- je veux aussi un `cat /etc/dhcp/dhcpd.conf` dans le compte-rendu, pour que je vois le fichier de configuration
  - habituez-vous à me montrer vos fichiers de conf avec la commande `cat` dans les compte-rendus
  - **votre serveur DHCP doit attribuer des IPs entre `10.4.1.137`  et `10.4.1.237`**

> *Bon bah c'est pas tout mais c'est qu'il s'agirait de voir s'il fonctionne ce serveur DHCP !*

### 5. Client DHCP

➜ **Petite astuce**

- pour avoir un peu plus de détails sur l'interaction entre le client et votre serveur DHCP, vous pouvez lancer la commande `sudo journalctl -xe -u dhcpd -f` sur `dhcp.tp4.b1`
- cette commande permet de suivre en temps réel l'arrivée de nouveaux logs
- si vous laissez tourner cette commande pendant l'étape qui suit, vous allez voir arriver en temps réel les requêtes DHCP du client dnas les logs

🌞 **Test !**

- utilisez `node1.tp4.b1` pour faire les tests : il va récupérer une IP avec votre serveur
- référez-vous au [mémo](../../cours/memo/rocky_network.md) pour voir comment configurer une interface pour qu'elle récupère une IP dynamiquement en DHCP

🌞 **Prouvez que**

- `node1.tp4.b1` a bien récupéré une IP **dynamiquement**
- `node1.tp4.b1` a enregistré un bail DHCP
  - déterminer la date exacte de création du bail
  - déterminer la date exacte d'expiration
  - déterminer l'adresse IP du serveur DHCP (depuis `node1.tp4.b1` : il a enregistré l'adresse IP du serveur DHCP)
- vous pouvez ping `router.tp4.b1` et `node2.tp4.b1` grâce à cette nouvelle IP récupérée

🌞 **Bail DHCP serveur**

- sur `dhcp.tp4.b1` montrez le fichier qui contient le bail DHCP de `node1.tp4.b1`

### 6. Options DHCP

Dans cette partie, vous allez modifier la conf de votre serveur DHCP. Vous allez utiliser les deux options suivantes dans la conf :

- `option routers x.x.x.x;`
  - permet de préciser l'IP de la passerelle du réseau au client
  - remplacer `x.x.x.x` par l'adresse IP de `router.tp4.b1`
- `option domain-name-servers x.x.x.x;`
  - permet de préciser au client l'adresse IP d'un serveur DNS joignable depuis ce réseau
  - remplacer `x.x.x.x` par l'adresse IP d'un serveur DNS public que vous connaissez
- `default-lease-time xxx;` et `max-lease-time xxx;`
  - qui permettent de modifier la durée du bail DHCP
  - `xxx` est une valeur en seconde
  - vous devrez indiquer une durée de bail de 6 heures

🌞 **Nouvelle conf !**

- montrez la nouvelle conf (avec la commande `cat`)
- redémarrage du service DHCP (`sudo systemctl restart dhcpd`)

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