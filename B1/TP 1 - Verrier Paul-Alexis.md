# TP 1 - Verrier Paul-Alexis

## I. Exploratoin locale en Solo

### 1. Affichage d'informations sur la pile TCP/IP locale

* Affichage des infos des cartes réseau du PC :
>    `commande : ip -a`
    
* Resultat : 
```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
       
2: wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 64:5d:86:6b:22:45 brd ff:ff:ff:ff:ff:ff
    inet 10.33.3.222/22 brd 10.33.3.255 scope global dynamic noprefixroute wlan0
       valid_lft 3301sec preferred_lft 3301sec
    inet6 fe80::7136:6209:8126:e5ce/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```
>Nom: Wlan0
>Adresse MAC: 64:5d:86:6b:22:45
>Adresse IP: 10.33.3.222
>
>Pas de carte Ethernet.

* Affichage du Gateway : 
>`commande : netstat -nr`

* Resultat : 
```
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
0.0.0.0         10.33.3.253     0.0.0.0         UG        0 0          0 wlan0
10.33.0.0       0.0.0.0         255.255.252.0   U         0 0          0 wlan0
```

>Gateway: 10.33.3.253

* En Grahique : (GUI : Graphical User Interface)

>System > Info Center > Network Informations > Network Interfaces

![](https://i.imgur.com/yk68NmN.png)

>Question : à quoi sert la gateway dans le réseau d'YNOV ?
>
>Réponse : Elle sert à faire la lisaison entre deux réseaux

### 2. Modification des Informations

#### A. Modification d'adresse IP (part 1)

> L'utilisation de Parrot Os ne permet pas la configuration graphique de l'IP
> La perte de la connexion internet s'explique par le fait qu'il s'applique alors un changement de sous réseau, le gateway ne voit alors plus l'ordiangeur et le connexion est perdue.

#### B. nmap

>Utilisez nmap pour scanner le réseau de votre carte WiFi et trouver une adresse IP libre

> Commande: `nmap -sP 10.33.0.0/22`
> 
> Resultat: https://pastebin.com/T5d5hiAE

#### C. Modification d'adresse IP (part 2)

> Après avoir fait le scan nmap, l'adresse 10.33.3.165 s'est avérée libre (car elle n'apparait pas), je l'ai donc utilisée pour faire le nouveau ping.

> Puis, on fait un nouveau ping:

> Commande: `nmap -sP 10.33.0.0/22`
> 
> Resultat: Le resultat est le même qu'auparavant.

### II. Exploration locale en duo

#### 1. Prérequis 
>aquis
#### 2. Câblage 
>cablage effectué via adaptateur usb to ethernet 

#### 3. Modification d'adresse IP

>modification de l'addresse IPv4 en /30


#### 4. Utilisation d'un des deux comme gateway
>- ma configuration :
>![](https://i.imgur.com/T1aqyC9.png)
>- connexion du 2ème PC:
>Carte Ethernet Ethernet 5 :
>
>   Suffixe DNS propre à la connexion. . . :
>   Adresse IPv6 de liaison locale. . . . .: fe80::b929:bcb6:9384:13aa%78
>   Adresse IPv4. . . . . . . . . . . . . .: 192.168.137.1
>   Masque de sous-réseau. . . . . . . . . : 255.255.255.0
>   Passerelle par défaut. . . . . . . . . :
   

#### 5. Petit chat privé
>commande utilisé : 
>netcat -l -p 3000

>résultat obtenue 
>![](https://i.imgur.com/EPnOjIW.png)


#### 6. Wireshark
>-message netcat reçu : 
>![](https://i.imgur.com/2ZvP2oG.png)
>-ping reponse : 
>![](https://i.imgur.com/f5oDPmo.png)


#### 7. Firewall
>- firewall activé autorisation port 3000
>netcat  192.168.137.1 3000
>jlvhsdbkl
>adad
>dvdvdvdvdvdvdvdv


>- protocole ping bloqué : 
>PING 3000 (0.0.11.184) 56(124) bytes of data.
>^C
>--- 3000 ping statistics ---
>5 packets transmitted, 0 received, 100% packet loss, time 4072ms


>- bloquer protocole ICMPv4 (le ping)
>ping 192.168.137.1
>PING 192.168.137.1 (192.168.137.1) 56(84) bytes of data.
>^C
>--- 192.168.137.1 ping statistics ---
>11 packets transmitted, 0 received, 100% packet loss, time 10244ms

## III. Manipulations d'autres outils/protocoles côté client

### 1. DHCP

>-> Afficher l'adresse IP du serveur DHCP du réseau WiFi YNOV
>
>Commandes
>```
>sudo dhclient -v wlan0
>```
>Résultats
>```
>Listening on LPF/wlan0/04:ea:56:21:65:4c
>Sending on   LPF/wlan0/04:ea:56:21:65:4c
>Sending on   Socket/fallback
>DHCPREQUEST for 10.33.2.86 on wlan0 to 255.255.255.255 port 67
>DHCPACK of 10.33.2.86 from 10.33.3.254
>RTNETLINK answers: File exists
>bound to 10.33.2.86 -- renewal in 1360 seconds.
>```
>La ligne qui nous intéresse :
>```
>DHCPACK of 10.3cat dhclient-f6160ce6-9132-4c76-b7b2-9c6e9cbb88f1->wlan0.lease3.2.86 from 10.33.3.254
>```
>Cette commande permet de demander une nouvelle adresse ip au serveur dhcp.
>Ici, on remarque qu'il nous donne l'adresse ip : "10.33.2.86" depuis le serveur  dhcp : "10.33.3.254"
>
>-> BAIL DHCP
>Pour exemple, avec NetworkManager, on peut faire cette commande :
>```
>cat dhclient-f6160ce6-9132-4c76-b7b2-9c6e9cbb88f1-wlan0.lease
>```
>Ce qui donne
>```
>lease {
>  interface "wlan0";
>  fixed-address 192.168.20.116;
>  option subnet-mask 255.255.255.0;
>  option wpad 22:a:22;
>  option dhcp-lease-time 86400;
>  option routers 192.168.20.1;
>  option dhcp-message-type 5;
>  option dhcp-server-identifier 192.168.20.1;
>  option domain-name-servers 192.168.20.1;
>  option dhcp-renewal-time 43200;
> option broadcast-address 192.168.20.255;
>  option host-name "parrot";
>  renew 4 2019/11/14 05:55:29;
>  rebind 4 2019/11/14 17:19:08;
>  expire 4 2019/11/14 20:19:08;
>}
>lease {
>  interface "wlan0";
>  fixed-address 192.168.20.116;
>  option subnet-mask 255.255.255.0;
>  option routers 192.168.20.1;
>  option dhcp-lease-time 86400;
>  option dhcp-message-type 5;
>  option domain-name-servers 192.168.20.1;
>  option dhcp-server-identifier 192.168.20.1;
>  option dhcp-renewal-time 43200;
>  option broadcast-address 192.168.20.255;
>  option dhcp-rebinding-time 75600;
>  option host-name "parrot";
>  renew 4 2019/11/14 08:38:29;
>  rebind 4 2019/11/14 18:09:41;
>  expire 4 2019/11/14 21:09:41;
>
>```
>Les lignes qui nous intéressent : 
>```
>renew 4 2019/11/14 08:38:29;
>rebind 4 2019/11/14 18:09:41;
>expire 4 2019/11/14 21:09:41;
>```
>
>-> Demandez une nouvelle adresse IP
>
>Commandes
>```
>sudo dhclient -v wlan0
>```
>Le résultat est un peu plus haut.
> --> DHCPACK of 10.33.2.86 from 10.33.3.254
>
### 2. DNS
>
>-> Trouver l'ip serveur dns que notre ordi connait : 
>
>Commande
>```
>ip r | grep default
>```
>Résultat
>```
>default via 10.33.3.253 dev wlan0
>```
>Notre ordinateur connait 10.33.3.253.
>
#### lookup
>
>-> lookup google.com
>Commande
>```
>dig google.com
>```
>Résultat
>```
> <<>> DiG 9.11.5-P4-5.1+b1-Debian <<>> google.com
> global options: +cmd
> Got answer:
> ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 51547
> flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1
>
> OPT PSEUDOSECTION:
> EDNS: version: 0, flags:; udp: 4096
> QUESTION SECTION:
>google.com.			IN	A
>
> ANSWER SECTION:
>google.com.		245	IN	A	216.58.201.238
>
> Query time: 4 msec
> SERVER: 10.33.10.148#53(10.33.10.148)
> WHEN: mer. janv. 22 10:05:44 CET 2020
> MSG SIZE  rcvd: 55
>```
>
>Dns google : 216.58.201.238
>
>->Lookup ynov.com
>Commande
>```
>dig ynov.com
>```
>Résultat
>```
> <<>> DiG 9.11.5-P4-5.1+b1-Debian <<>> ynov.com
> global options: +cmd
> Got answer:
> ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 58069
> flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1
>
> OPT PSEUDOSECTION:
> EDNS: version: 0, flags:; udp: 4096
> QUESTION SECTION:
>ynov.com.			IN	A
>
> ANSWER SECTION:
>ynov.com.		2329	IN	A	217.70.184.38
>
> Query time: 5 msec
> SERVER: 10.33.10.148#53(10.33.10.148)
> WHEN: mer. janv. 22 10:15:34 CET 2020
> MSG SIZE  rcvd: 53
>
>```
>
>Dns Ynov : 217.70.184.38
>
>On remarque que les résultats sont différents car notre "odinateur" ne >passe pas par le meme chemin pour aller sur google.com et sur Ynov.com
>
#### Reverse LOOKUP
>
>-> reverse lookup 78.74.21.21
>Commande
>```
>dig -x 78.74.21.21
>```
>Résultat
>```
> <<>> DiG 9.11.5-P4-5.1+b1-Debian <<>> -x 78.74.21.21
> global options: +cmd
> Got answer:
> ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 45166
> flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1
>
> OPT PSEUDOSECTION:
> EDNS: version: 0, flags:; udp: 4096
> QUESTION SECTION:
>21.21.74.78.in-addr.arpa.	IN	PTR
>
> ANSWER SECTION:
>21.21.74.78.in-addr.arpa. 3490	IN	PTR	host-78-74-21-21.homerun.telia.com.
>
> Query time: 5 msec
> SERVER: 10.33.10.148#53(10.33.10.148)
> WHEN: mer. janv. 22 10:20:48 CET 2020
> MSG SIZE  rcvd: 101
>```
>
>Serveur : host-78-74-21-21.homerun.telia.com.
>
>-> reverse Lookup 92.146.54.88
>Commande
>```
>dig -x 92.146.54.88
>```
>Résultat
>```
> <<>> DiG 9.11.5-P4-5.1+b1-Debian <<>> -x 92.146.54.88
> global options: +cmd
> Got answer:
> ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 19911
> flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1
>
> OPT PSEUDOSECTION:
> EDNS: version: 0, flags:; udp: 4096
> QUESTION SECTION:
>88.54.146.92.in-addr.arpa.	IN	PTR
>
> ANSWER SECTION:
>88.54.146.92.in-addr.arpa. 3425	IN	PTR	>apoitiers654-1-167-88.w92-146.abo.wanadoo.fr.
>
> Query time: 6 msec
> SERVER: 10.33.10.148#53(10.33.10.148)
> WHEN: mer. janv. 22 10:22:12 CET 2020
> MSG SIZE  rcvd: 113
>```
>
>Serveur : apoitiers654-1-167-88.w92-146.abo.wanadoo.fr.
>
>Grace  à la commande dig -x (qui permet de faire un reverse lookup), on peut obtenir facilement le serveur derriere une adresse ip.