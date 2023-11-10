# TP5 : TCP, UDP et services réseau

## I. First steps

🌞 **Déterminez, pour ces 5 applications, si c'est du TCP ou de l'UDP**

Steam : `185.25.182.40`, port `443`. Port ouvert local : `57527`

Youtube : `20.50.201.200`, port `443`. Port ouvert local : `57695`

Xbox : `23.15.179.208`, port `80`. Port ouvert local : `57881`

Discord : `66.22.199.200`, port `50002`. Port ouvert local : `50700`

Twitch : `52.223.202.183`, port `443`. Port ouvert local : `58245`

🌞 **Demandez l'avis à votre OS**

- votre OS est responsable de l'ouverture des ports, et de placer un programme qui se "lie" à un port (on dit généralement qu'il se *bind* à un port)
- l'OS garde une trace en permanence de quelle programme utilise quel port
- utilisez la commande adaptée à votre OS pour repérer, dans la liste de toutes les connexions réseau établies, la connexion que vous voyez dans Wireshark, pour chacune des 5 applications
- **lancez votre terminal en admin pour avoir toutes les infos**

**Il faudra ajouter des options adaptées aux commandes pour y voir clair. Pour rappel, vous cherchez des connexions TCP ou UDP.**

```
# MacOS
$ netstat

# GNU/Linux
$ ss

# Windows
$ netstat
```

🦈🦈🦈🦈🦈 **Bah ouais, 5 captures Wireshark à l'appui évidemment.** Ce sera `tp5_service_1.pcapng` jusqu'à `tp5_service_5.pcapng`.

> Une capture pour chaque application, qui met bien en évidence le trafic en question. C'est à dire on ne doit voir QUE le trafic de l'application en question. Vous devez donc utiliser un filtre dans Wireshark pour isoler le trafic que vous souhaitez.

## II. Setup Virtuel

### 1. SSH

🌞 **déterminez si SSH utilise TCP ou UDP**

SSH utilise TCP

🌞 **Demandez aux OS**

```shell
PS C:\Users\fayer> netstat -ni
TCP    10.5.1.1:55726         10.5.1.11:22           ESTABLISHED        102442
```

```shell
[bjorn@node1 ~]$ ss -an
tcp    ESTAB     0        52           10.5.1.11:22                10.5.1.1:55726
```

- vous devez repérer clairement une ligne qui indique que c'est le programme `sshd` (le serveur SSH) qui est responsable de la connexion

🦈 **[tp5_3_way.pcapng](./tp5_3_way.pcapng)

### 2. Routage

🌞 **Prouvez que**

```shell
[bjorn@node1 ~]$ ping 1.1.1.1
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=55 time=25.5 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=55 time=25.1 ms
64 bytes from 1.1.1.1: icmp_seq=3 ttl=55 time=23.9 ms
```

```shell
[bjorn@node1 ~]$ ping ynov.com
PING ynov.com (104.26.11.233) 56(84) bytes of data.
64 bytes from 104.26.11.233 (104.26.11.233): icmp_seq=1 ttl=55 time=23.0 ms
64 bytes from 104.26.11.233 (104.26.11.233): icmp_seq=2 ttl=55 time=25.6 ms
64 bytes from 104.26.11.233 (104.26.11.233): icmp_seq=3 ttl=55 time=24.8 ms
```

### 3. Serveur Web

🌞 **Créer le site web**

```shell
   31  sudo mkdir www
   32  cd www
   33  sudo mkdir site_web_tp5
   34  cd site_web_tp5/
   35  sudo nano index.html
```

🌞 **Donner les bonnes permissions**

```shell
[bjorn@web ~]$ sudo chown -R nginx:nginx /var/www/site_web_tp5
```

🌞 **Démarrer le serveur web !**

```nginx
[bjorn@web ~]$ systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; preset: disabled)
     Active: active (running) since Fri 2023-11-10 16:38:50 CET; 48s ago
    Process: 11546 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
    Process: 11547 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
    Process: 11548 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
   Main PID: 11549 (nginx)
      Tasks: 2 (limit: 4672)
     Memory: 1.9M
        CPU: 40ms
     CGroup: /system.slice/nginx.service
             ├─11549 "nginx: master process /usr/sbin/nginx"
             └─11550 "nginx: worker process"
```

🌞 **Ouvrir le port firewall**

```shell
[bjorn@web ~]$ sudo sh -c firewall-cmd --permanent --zone=public --add-service=http
[bjorn@web ~]$ sudo sh -c firewall-cmd --reload"
```

🌞 **Visitez le serveur web !**

```shell
[bjorn@node1 ~]$ sudo curl http://10.5.1.12
[sudo] password for bjorn:
<h1>SITE</h1>
```

🌞 **Visualiser le port en écoute**

```shell
[bjorn@web ~]$ ss -nlt
LISTEN            0                 511                                0.0.0.0:80                                0.0.0.0:*
LISTEN            0                 511                                   [::]:80                                   [::]:*
```

🌞 **Analyse trafic**

- pendant que vous effectuez une connexion d'un client vers le serveur web qui tourne sur `web.tp5.b1`, lancez une capture réseau
- repérez :
  - le 3 way handshake TCP
  - du trafic HTTP
  - le contenu de la page HTML retourné (oui c'est visible direct dans Wireshark)

🦈 **`tp5_web.pcapng`** avec le 3-way handshake, la page HTML retournée, et une fin de connexion propre si vous en avez une :)
