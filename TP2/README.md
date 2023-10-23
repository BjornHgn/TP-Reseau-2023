# TP2 : Ethernet, IP, et ARP

# I. Setup IP

🌞 **Mettez en place une configuration réseau fonctionnelle entre les deux machines**
```
PS C:\Users\fayer> netsh interface ipv4 set address name="Ethernet" static 10.10.10.185 255.255.255.252
```
Mon Ip : 10.10.10.185

Ip machine 2 : 10.10.10.190

Mask : 255.255.255.252

Net address : 10.10.10.184

Broadcast address : 10.10.10.187

🌞 **Prouvez que la connexion est fonctionnelle entre les deux machines**
```
PS C:\Users\fayer> ping 10.10.10.186

Envoi d’une requête 'Ping'  10.10.10.186 avec 32 octets de données :
Réponse de 10.10.10.186 : octets=32 temps=4 ms TTL=128
Réponse de 10.10.10.186 : octets=32 temps=4 ms TTL=128
Réponse de 10.10.10.186 : octets=32 temps=4 ms TTL=128
Réponse de 10.10.10.186 : octets=32 temps=5 ms TTL=128
```
🌞 **Wireshark it**

🦈 **PCAP qui contient les paquets ICMP qui vous ont permis d'identifier les types ICMP** (juste quelques-uns)

# II. ARP my bro

🌞 **Check the ARP table**
```
PS C:\Users\fayer> arp -a
```
```
10.10.10.186          08-8f-c3-fe-ea-ee     dynamique
```
Adresse gateway
```
 10.33.51.254          7c-5a-1c-cb-fd-a4     dynamique
 ```

🌞 **Manipuler la table ARP**

```
PS C:\Users\fayer> arp -d
```
```
PS C:\Users\fayer> arp -a

Interface : 192.168.199.1 --- 0x4
  Adresse Internet      Adresse physique      Type
  192.168.199.255       ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique

Interface : 10.33.48.141 --- 0x5
  Adresse Internet      Adresse physique      Type
  10.33.51.254          7c-5a-1c-cb-fd-a4     dynamique
  224.0.0.22            01-00-5e-00-00-16     statique

Interface : 10.10.10.185 --- 0xe
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique

Interface : 192.168.109.1 --- 0x12
  Adresse Internet      Adresse physique      Type
  192.168.109.255       ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique
```
```
PS C:\Users\fayer> arp -a

Interface : 192.168.199.1 --- 0x4
  Adresse Internet      Adresse physique      Type
  192.168.199.254       00-50-56-ef-d7-62     dynamique
  192.168.199.255       ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.251           01-00-5e-00-00-fb     statique
  224.0.0.252           01-00-5e-00-00-fc     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique

Interface : 10.33.48.141 --- 0x5
  Adresse Internet      Adresse physique      Type
  10.33.51.254          7c-5a-1c-cb-fd-a4     dynamique
  10.33.51.255          ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.251           01-00-5e-00-00-fb     statique
  224.0.0.252           01-00-5e-00-00-fc     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique
  255.255.255.255       ff-ff-ff-ff-ff-ff     statique

Interface : 10.10.10.185 --- 0xe
  Adresse Internet      Adresse physique      Type
  10.10.10.186          08-8f-c3-fe-ea-ee     dynamique
  10.10.10.187          ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.251           01-00-5e-00-00-fb     statique
  224.0.0.252           01-00-5e-00-00-fc     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique

Interface : 192.168.109.1 --- 0x12
  Adresse Internet      Adresse physique      Type
  192.168.109.254       00-50-56-fa-63-23     dynamique
  192.168.109.255       ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.251           01-00-5e-00-00-fb     statique
  224.0.0.252           01-00-5e-00-00-fc     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique
  ```

🌞 **Wireshark it**

La première adresse source est ```CompalIn_10:d5:74 (40:c2:ba:10:d5:74)```. L'adresse de destination est ```Broadcast (ff:ff:ff:ff:ff:ff)```.

La deuxième adresse source est ```CompalIn_fe:ea:ee (08:8f:c3:fe:ea:ee)```. L'adresse de destination est ```CompalIn_10:d5:74 (40:c2:ba:10:d5:74)```.
  - déterminez à quoi correspond chacune de ces adresses

🦈 **PCAP qui contient les DEUX trames ARP**

# III. DHCP
Pour forcer un échange DHCP sur Windows, j'ai utilisé ```ipconfig /release``` puis ```ipconfig /renew```.

🌞 **Wireshark it**

DHCP Discover : adresse source ```0.0.0.0```, adresse de destination ```255.255.255.255```.
```
Option: (50) Requested IP Adress (192.168.1.16)
```

DHCP Offer : adresse source ```192.168.1.1```, adresse de destination ```192.168.1.16```.

DHCP Request : adresse source ```0.0.0.0```, adresse de destination ```255.255.255.255```.
```
Option: (54) DHCP Server Identifier (192.168.1.1)
```
DHCP ACK : adresse source ````192.168.1.1````, adresse de destination ```192.168.1.16```.
```
Option: (3) Router
Router: 192.168.1.1
```