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
```
```
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

## II. Exploration locale en duo

## III. Manipulations d'autres outils/protocoles côté client