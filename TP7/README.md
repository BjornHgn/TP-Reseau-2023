# TP7 : Do u secure

## II. SSH

### 1. Fingerprint

#### A. Explications

🌞 **Effectuez une connexion SSH en vérifiant le fingerprint**

```shell
PS C:\Users\fayer> ssh bjorn@10.7.1.11
The authenticity of host '10.7.1.11 (10.7.1.11)' can't be established.
ED25519 key fingerprint is SHA256:wJhOzlmF9vZF7kmwOQRoWe+RuA0Z746PwzhmRUDX2cw.
This host key is known by the following other names/addresses:
    C:\Users\fayer/.ssh/known_hosts:1: 10.3.1.12
    C:\Users\fayer/.ssh/known_hosts:4: 10.3.1.11
    C:\Users\fayer/.ssh/known_hosts:5: 10.3.1.254
    C:\Users\fayer/.ssh/known_hosts:6: 10.3.2.12
    C:\Users\fayer/.ssh/known_hosts:7: 10.4.1.254
    C:\Users\fayer/.ssh/known_hosts:8: 10.4.1.12
    C:\Users\fayer/.ssh/known_hosts:9: 10.4.1.253
    C:\Users\fayer/.ssh/known_hosts:10: 10.4.1.2
    (13 additional names omitted)
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.7.1.11' (ED25519) to the list of known hosts.
```

```shell
[bjorn@john ~]$ sudo ssh-keygen -l -f /etc/ssh/ssh_host_ed25519_key
[sudo] password for bjorn:
256 SHA256:wJhOzlmF9vZF7kmwOQRoWe+RuA0Z746PwzhmRUDX2cw /etc/ssh/ssh_host_ed25519_key.pub (ED25519)
```

## 2. Conf serveur SSH

🌞 **Consulter l'état actuel**

```shell
[bjorn@john ~]$ ss -ln
tcp        LISTEN      0           128                                                  0.0.0.0:22                           0.0.0.0:*
tcp        LISTEN      0           128                                                     [::]:22                              [::]:*
```

🌞 **Modifier la configuration du serveur SSH**

```shell
[bjorn@routeur ~]$ sudo cat /etc/ssh/sshd_config
Port 25000
#AddressFamily any
ListenAddress 10.7.1.254
#ListenAddress ::
```

🌞 **Prouvez que le changement a pris effet**

```shell
[bjorn@routeur ~]$ ss -ln
tcp        LISTEN      0           128                                               10.7.1.254:25000                        0.0.0.0:*
```

🌞 **Effectuer une connexion SSH sur le nouveau port**

```shell
PS C:\Users\fayer> ssh bjorn@10.7.1.254 -p 25000
Last login: Thu Nov 23 15:43:02 2023

[bjorn@routeur ~]$
```

## 3. Connexion par clé

🌞 **Générer une paire de clés**

```bash
$ PS C:\Users\fayer> ssh-keygen -t rsa -b 4096
Generating public/private rsa key pair.
```

🌞 **Déposer la clé publique sur une VM**

```bash
$ ssh-copy-id bjorn@10.7.1.11
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/c/Users/fayer/.ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
bjorn@10.7.1.11's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'bjorn@10.7.1.11'"
and check to make sure that only the key(s) you wanted were added.
```

🌞 **Connectez-vous en SSH à la machine**

```shell
$ ssh bjorn@10.7.1.11
Last login: Thu Nov 23 15:21:43 2023 from 10.7.1.1
[bjorn@john ~]$ 
```

### C. Changement de fingerprint

🌞 **Supprimer les clés sur la machine `router.tp7.b1`**

```shell
[bjorn@john ~]$ sudo rm /etc/ssh/ssh_host_*
```

🌞 **Regénérez les clés sur la machine `router.tp7.b1`**

```shell
[bjorn@john ~]$ sudo ssh-keygen -A
ssh-keygen: generating new host keys: RSA DSA ECDSA ED25519
```

🌞 **Tentez une nouvelle connexion au serveur**

```shell
PS C:\Users\fayer> ssh bjorn@10.7.1.11
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
```

```shell
PS C:\Users\fayer> ssh bjorn@10.7.1.11
The authenticity of host '10.7.1.11 (10.7.1.11)' can't be established.
ED25519 key fingerprint is SHA256:pBntND2BfSQ2kH5ypyWDMdbtK1s/VZDJMPVTOEBwht8.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.7.1.11' (ED25519) to the list of known hosts.
Last login: Thu Nov 23 16:00:54 2023 from 10.7.1.1
```

## III. Web sécurisé

🌞 **Montrer sur quel port est disponible le serveur web**

```shell
[bjorn@web ~]$ ss -lnt
LISTEN            0                 511                                   [::]:80                                   [::]:*
```

## 1. Setup HTTPS

🌞 **Générer une clé et un certificat sur `web.tp7.b1`**

```bash
$ openssl req -new -newkey rsa:2048 -days 365 -nodes -x509 -keyout server.key -out server.crt

$ sudo mv server.key /etc/pki/tls/private/web.tp7.b1.key

$ sudo mv server.crt /etc/pki/tls/certs/web.tp7.b1.crt

$ sudo chown nginx:nginx /etc/pki/tls/private/web.tp7.b1.key
$ sudo chown nginx:nginx /etc/pki/tls/certs/web.tp7.b1.crt
$ sudo chmod 0400 /etc/pki/tls/private/web.tp7.b1.key
$ sudo chmod 0444 /etc/pki/tls/certs/web.tp7.b1.crt
```

🌞 **Modification de la conf de NGINX**

```bash
[bjorn@vpn ~]$ sudo cat /etc/nginx/conf.d/site_web_tp7.conf
server {

    listen 10.7.1.12:443 ssl;

    ssl_certificate /etc/pki/tls/certs/web.tp7.b1.crt;
    ssl_certificate_key /etc/pki/tls/private/web.tp7.b1.key;

    server_name www.site_web_nul.b1;
    root /var/www/site_web_nul;
}
```

🌞 **Conf firewall**

`[bjorn@web ~]$ sudo firewall-cmd --add-port=443/tcp --permanent`

🌞 **Redémarrez NGINX**

`[bjorn@web ~]$ sudo systemctl restart nginx`

🌞 **Prouvez que NGINX écoute sur le port 443/tcp**

```shell
[bjorn@web ~]$ ss -lnt
LISTEN            0                 511                              10.7.1.12:443                               0.0.0.0:*
```

🌞 **Visitez le site web en https**

```shell
[bjorn@john ~]$ curl -k https://10.7.1.12
<h1>Oui</h1>
```

## IV. VPN

🌞 **Installer le serveur Wireguard sur `vpn.tp7.b1`**

```bash
[bjorn@vpn ~]$
   11  sudo modprobe wireguard
   12  echo wireguard | sudo tee /etc/modules-load.d/wireguard.conf
   13  sudo nano /etc/sysctl.conf
   14  sudo sysctl -p
   15  sudo dnf install wireguard-tools -y
   16  sudo nano /etc/resolv.conf
```

🌞 **Générer la paire de clé du serveur sur `vpn.tp7.b1`**

```bash
[bjorn@vpn ~]$ wg genkey | sudo tee /etc/wireguard/server.key
YOJrUcgoEmMvzh+m8h46Q/XwgoIem8bs6BjpRZjnHEw=
[bjorn@vpn ~]$ sudo chmod 0400 /etc/wireguard/server.key
[bjorn@vpn ~]$ sudo cat /etc/wireguard/server.key | wg pubkey | sudo tee /etc/wireguard/server.pub
4liBqThfbgK/AzYjd2SppUsoMfsGOanPKKvtRLDjUH8=
[bjorn@vpn ~]$
```

🌞 **Générer la paire de clé du client sur `vpn.tp7.b1`**

```bash
[bjorn@vpn ~]$ sudo mkdir -p /etc/wireguard/clients
[bjorn@vpn ~]$ wg genkey | sudo tee /etc/wireguard/client/john.key
tee: /etc/wireguard/client/john.key: No such file or directory
kHzcKC+vwDrq33Xvdzy6f+98xYu13FxkXm3g7uUxCmQ=
[bjorn@vpn ~]$ wg genkey | sudo tee /etc/wireguard/clients/john.key
SLhXGHZQxsVi9DAAhcIvB8gb53dloQLJOBgPjDH5H1g=
[bjorn@vpn ~]$ sudo chmod 0400 /etc/wireguard/server.key
[bjorn@vpn ~]$ sudo cat /etc/wireguard/clients/john.key | wg pubkey | tee /etc/wireguard/clients/john.pub
```

🌞 **Création du fichier de config du serveur sur `vpn.tp7.b1`**

```bash
$ sudo cat /etc/wireguard/wg0.conf
[Interface]
Address = 10.107.1.0/24
SaveConfig = false
PostUp = firewall-cmd --zone=public --add-masquerade
PostUp = firewall-cmd --add-interface=wg0 --zone=public
PostDown = firewall-cmd --zone=public --remove-masquerade
PostDown = firewall-cmd --remove-interface=wg0 --zone=public
ListenPort = 13337
PrivateKey = <clé du fichier /etc/wireguard/server.key>

[Peer]
PublicKey = <clé du fichier /etc/wireguard/clients/john.pub>
AllowedIPs = 10.107.1.11/32
```

🌞 **Démarrez le serveur VPN sur `vpn.tp7.b1`**

```bash
[bjorn@vpn ~]$ sudo systemctl start wg-quick@wg0.service`

[bjorn@vpn ~]$ ip a
6: wg0: <POINTOPOINT,NOARP,UP,LOWER_UP> mtu 1420 qdisc noqueue state UNKNOWN group default qlen 1000
    link/none
    inet 10.107.1.0/24 scope global wg0
       valid_lft forever preferred_lft forever

[bjorn@vpn ~]$ sudo wg show
interface: wg0
  public key: 4liBqThfbgK/AzYjd2SppUsoMfsGOanPKKvtRLDjUH8=
  private key: (hidden)
  listening port: 13337

peer: X5oeLW2bN/EBW2gMzIEYChkP8DiJLEluhBvXfR1SPlY=
  allowed ips: 10.107.1.11/32
```

🌞 **Ouvrir le bon port firewall**

```bash
[bjorn@vpn ~]$ ss -lnp
[bjorn@vpn ~]$ sudo firewall-cmd --add-port=13337/udp --permanent
success
[bjorn@vpn ~]$ sudo firewall-cmd --reload
success
```

### B. Setup client

🌞 **On installe les outils Wireguard sur `john.tp7.b1`**

```bash
[bjorn@john ~]$ sudo dnf install wireguard-tools
```

🌞 **Supprimez votre route par défaut**

```bash
[bjorn@john ~]$ sudo ip route del default via 10.7.1.254 dev enp0s3
```

🌞 **Configurer un fichier de conf sur `john.tp7.b1`**

- utilisez `nano` pour éditer le fichier, je vous montre son contenu avec `cat`

```bash
$ mkdir wireguard

[bjorn@john ~]$ sudo nano wireguard/john.conf
[Interface]
Address = 10.107.1.11/24
PrivateKey = X5oeLW2bN/EBW2gMzIEYChkP8DiJLEluhBvXfR1SPlY=

[Peer]
PublicKey = 4liBqThfbgK/AzYjd2SppUsoMfsGOanPKKvtRLDjUH8=
AllowedIPs = 0.0.0.0/0
Endpoint = 10.7.1.101:13337
```

🌞 **Lancer la connexion VPN**

```bash
[bjorn@john ~]$ cd wireguard
[bjorn@john wireguard]$ wg-quick up ./john.conf
```

🌞 **Vérifier la connexion**

```bash
[bjorn@john wireguard]$ curl https://google.com
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="https://www.google.com/">here</A>.
</BODY></HTML>
[bjorn@john wireguard]$ ping 1.1.1.1
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=54 time=26.1 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=54 time=29.2 ms
^C
--- 1.1.1.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 26.054/27.625/29.196/1.571 ms
[bjorn@john wireguard]$ traceroute 1.1.1.1
traceroute to 1.1.1.1 (1.1.1.1), 30 hops max, 60 byte packets
 1  10.107.1.0 (10.107.1.0)  9.238 ms  10.312 ms  15.329 ms
 2  * * *
 3  10.0.3.2 (10.0.3.2)  23.904 ms  23.331 ms  22.886 ms
 ```

```bash
[bjorn@vpn ~]$ sudo wg show
interface: wg0
  public key: jNiJStbFAevdBUWaIfoecECZZW7qt3oVOXAIndZSICE=
  private key: (hidden)
  listening port: 13337

peer: YhawjXF3ZyiWCZbeHK6xFWEJ3xlcNNOik+btEb0PkUQ=
  endpoint: 10.7.1.11:53528
  allowed ips: 10.107.1.11/32
  latest handshake: 21 seconds ago
  transfer: 4.51 KiB received, 2.15 KiB sent
  ```
