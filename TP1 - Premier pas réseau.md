# TP1 - Premier pas réseau

# I. Exploration locale en solo

## 1. Affichage d'informations sur la pile TCP/IP locale

**🌞 Affichez les infos des cartes réseau de votre PC**

```
ipconfig /all

Carte réseau sans fil Wi-Fi : adresse IPv4 10.33.48.141(préféré), adresse physique 98-59-7A-CA-5F-67

Carte Ethernet Ethernet : adresse physique 40-C2-BA-10-D5-74, pas d'adresse IP
```

**🌞 Affichez votre gateway**

```
ipconfig

Passerelle par défaut : 10.33.51.254
```

**🌞 Déterminer la MAC de la passerelle**

```
ipconfig /all

ping 10.33.51254

arp -a

Adresse MAC: 7c-5a-1c-cb-fd-a4
```

### En graphique (GUI : Graphical User Interface)

**🌞 Trouvez comment afficher les informations sur une carte IP (change selon l'OS)**

```
Centre Réseau et partage > Connexion réseau > Choisir la carte réseau > Détails

Adresse physique: 98-59-7A-CA-5F-67

Adresse IPv4: 10.33.48.141

Passerelle par défaut IPv4: 10.33.51.254
```

## 2. Modifications des informations

### A. Modification d'adresse IP (part 1)  

🌞 Utilisez l'interface graphique de votre OS pour **changer d'adresse IP** :

Centre Réseau et partage > Connexion réseau > Choisir la carte réseau > Propriétés > Protocole Internet version 4

Nouvelle IPv4 : 10.33.48.241

🌞 **Il est possible que vous perdiez l'accès internet.**

Car l'IP sert a nous identifier sur le réseaux, en changeant d'IP cela peux nous déconnecter un court instant car c'est comme si on éffectuer une connexion avec un nouvel appareil.

# II. Exploration locale en duo

## 3. Modification d'adresse IP

🌞 **Modifiez l'IP des deux machines pour qu'elles soient dans le même réseau**

Centre Réseau et partage > Connexion réseau > Choisir la carte réseau > Détails

🌞 **Vérifier à l'aide d'une commande que votre IP a bien été changée**

```
PS C:\Users\fayer>  ipconfig /all
```

🌞 **Vérifier que les deux machines se joignent**

```
Envoi d’une requête 'Ping'  10.10.10.90 avec 32 octets de données :

Réponse de 10.10.10.90 : octets=32 temps=5 ms TTL=128

Réponse de 10.10.10.90 : octets=32 temps=3 ms TTL=128

Réponse de 10.10.10.90 : octets=32 temps=4 ms TTL=128

Réponse de 10.10.10.90 : octets=32 temps=4 ms TTL=128
```

🌞 **Déterminer l'adresse MAC de votre correspondant**

```
PS C:\Users\fayer> arp -a

Adresse MAC: 08-8f-c3-fe-ea-ee
```

## 4. Petit chat privé

```
PS C:\Users\fayer\Desktop\netcat-win32-1.11\netcat-1.11> .\nc64.exe -l -p 8000

```

🌞 **Visualiser la connexion en cours**

```
PS C:\Users\fayer> netstat -a -n -b

   [nc64.exe]
  TCP    10.33.48.141:139       0.0.0.0:0              LISTENING 
  ```

🌞 **Pour aller un peu plus loin**

```
PS C:\Users\fayer> netstat -a -n -b
[nc64.exe]
  TCP    0.0.0.0:27036          0.0.0.0:0              LISTENING
```

```
PS C:\Users\fayer> netstat -a -n -b
 [nc64.exe]
  TCP    10.33.48.141:139       0.0.0.0:0              LISTENING
```

🌞 **Activez et configurez votre firewall**

```
PS C:\Users\lione\Downloads\netcat-win32-1.11\netcat-1.11> ping 192.168.137.1

Envoi d’une requête 'Ping'  192.168.137.1 avec 32 octets de données :
Réponse de 192.168.137.1 : octets=32 temps=8 ms TTL=128
Réponse de 192.168.137.1 : octets=32 temps=4 ms TTL=128
Réponse de 192.168.137.1 : octets=32 temps=4 ms TTL=128
Réponse de 192.168.137.1 : octets=32 temps=4 ms TTL=128
```

```
PS C:\Users\fayer\Desktop\netcat-win32-1.11\netcat-1.11>.\nc64.exe -l -p 8000
vghvghvghj
salut
```

🌞**Tester l'accès internet**

```
PS C:\Users\lione\Downloads\netcat-win32-1.11\netcat-1.11> ping 1.1.1.1

Envoi d’une requête 'Ping'  1.1.1.1 avec 32 octets de données :
Réponse de 192.168.137.2 : Impossible de joindre l’hôte de destination.
Délai d’attente de la demande dépassé.
Réponse de 192.168.137.2 : Impossible de joindre l’hôte de destination.
Réponse de 192.168.137.2 : Impossible de joindre l’hôte de destination.

Statistiques Ping pour 1.1.1.1:
    Paquets : envoyés = 4, reçus = 3, perdus = 1 (perte 25%)

 ```

🌞 **Prouver que la connexion Internet passe bien par l'autre PC**

- utiliser la commande `traceroute` ou `tracert` (suivant votre OS) pour bien voir que les requêtes passent par la passerelle choisie (l'autre le PC)

# III. Manipulations d'autres outils/protocoles côté client

## 1. DHCP

🌞**Exploration du DHCP, depuis votre PC**

```
PS C:\Users\fayer> ipconfig /all
Serveur DHCP . . . . . . . . . . . . . : 192.168.1.1
```

```
Bail obtenu. . . . . . . . . . . . . . : lundi 16 octobre 2023 17:38:28
Bail expirant. . . . . . . . . . . . . : mardi 17 octobre 2023 17:38:24
```  

## 2. DNS

🌞**Trouver l'adresse IP du serveur DNS que connaît votre ordinateur**

```
PS C:\Users\fayer> ipconfig /all
 Serveurs DNS: 10.33.10.2
```

🌞 **Utiliser, en ligne de commande l'outil `nslookup` (Windows, MacOS) ou `dig` (GNU/Linux, MacOS) pour faire des requêtes DNS à la main**

```
PS C:\Users\fayer> nslookup google.com
Réponse ne faisant pas autorité :
Nom :    google.com
Addresses:  2a00:1450:4007:810::200e
          172.217.20.206
```

```
PS C:\Users\fayer> nslookup ynov.com
Réponse ne faisant pas autorité :
Nom :    ynov.com
Addresses:  2606:4700:20::681a:ae9
          2606:4700:20::681a:be9
          2606:4700:20::ac43:4ae2
          104.26.10.233
          104.26.11.233
          172.67.74.226
```

```
Serveur :   livebox.home
Address:  2a01:cb19:941:be00:5efa:25ff:fe48:9120
```

```
PS C:\Users\fayer> nslookup 231.34.113.12
*** livebox.home ne parvient pas à trouver 231.34.113.12 : Non-existent domain
```

```
PS C:\Users\fayer> nslookup 78.34.2.17
Nom :    cable-78-34-2-17.nc.de
Address:  78.34.2.17
```

Le premier est un domaine non existant, cela nous renvoie donc une erreur car aucun domaine n'est attribué a cette adresse.

Le deuxième cas nous renvoit le nom de domaine a l'adresse 78.34.2.17 car il lui a été attribué.

# IV. Wireshark

🌞 Utilisez le pour observer les trames qui circulent entre vos deux carte Ethernet. Mettez en évidence :
