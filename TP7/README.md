# TP7 : Do u secure

## II. SSH

## 0. Setup

| Name            | LAN1 `10.7.1.0/24` |
| --------------- | ------------------ |
| `router.tp7.b1` | `10.7.1.254`       |
| `john.tp7.b1`   | `10.7.1.11`        |

Pas de nouvelles machines pour cette section donc.

## 1. Fingerprint

### A. Explications

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

On va faire un truc très basique, pour manipuler un peu toujours les cartes réseau, les IP, les ports, toussa : on va choisir explicitement l'IP et le port où tourne notre serveur SSH sur `router`.

La configuration du serveur SSH se fait dans le fichier `/etc/ssh/sshd_config`, utilisez `nano` pour le modifier par exemple.

Il faut être admin pour modifier le fichier, il faudra donc préfixer votre commande avec `sudo`.

🌞 **Consulter l'état actuel**

```shell
[bjorn@john ~]$ ss -ln
tcp        LISTEN      0           128                                                  0.0.0.0:22                           0.0.0.0:*
tcp        LISTEN      0           128                                                     [::]:22                              [::]:*
```

🌞 **Modifier la configuration du serveur SSH**

```shell
[bjorn@routeur ~]$ sudo cat /etc/ssh/sshd_config
# If you want to change the port on a SELinux system, you have to tell
# SELinux about this change.
# semanage port -a -t ssh_port_t -p tcp #PORTNUMBER
#
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
# génération de la clé et du certificat
$ openssl req -new -newkey rsa:2048 -days 365 -nodes -x509 -keyout server.key -out server.crt

# on déplace la clé dans un répertoire standard pour les clés
# et on la renomme au passage
$ sudo mv server.key /etc/pki/tls/private/web.tp7.b1.key

# pareil pour le cert
$ sudo mv server.crt /etc/pki/tls/certs/web.tp7.b1.crt

# on définit des permissions restrictives sur les deux fichiers
$ sudo chown nginx:nginx /etc/pki/tls/private/web.tp7.b1.key
$ sudo chown nginx:nginx /etc/pki/tls/certs/web.tp7.b1.crt
$ sudo chmod 0400 /etc/pki/tls/private/web.tp7.b1.key
$ sudo chmod 0444 /etc/pki/tls/certs/web.tp7.b1.crt
```

🌞 **Modification de la conf de NGINX**

- je vous montre le fichier avec `cat`, éditez-le avec `nano` de votre côté

```bash
[it4@web ~]$ sudo cat /etc/nginx/conf.d/site_web_nul.conf
server {
    # on change la ligne listen
    listen 10.7.1.12:443 ssl;

    # et on ajoute deux nouvelles lignes
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
