# TP3 - Routage, ARP, Spéléologie réseau

## Préparation de l'environnement

> Prouver que chacun des points de la préparation de l'environnement ont été respectés : 

| Point | Méthode pour prouver | Preuve |
| ----- | -------------------- | ------ |
|Carte NAT désactivée|ip a|![](https://i.imgur.com/LnbxAPC.png)|
|Server SSH fonctionnel qui écoute sur le port 777/tcp| ss -lntp |![](https://i.imgur.com/xkzrnSP.png)|
|Pare-feu activé et configuré|firewall-cmd --list-all|![](https://i.imgur.com/nj7u8q0.png)|
|Nom configuré|commande hostname|![](https://i.imgur.com/zforYOW.png)|
|Fichiers /etc/hosts de toutes les machines configurés|fichier /etc/hosts|![](https://i.imgur.com/fcwodkg.png)|
|Réseaux et adressage des machines|commandes ip a et ping|![](https://i.imgur.com/NV8DS5H.png) ![](https://i.imgur.com/jf4Bbts.png) ![](https://i.imgur.com/buebn4J.png) ![](https://i.imgur.com/TEVNebT.png)|

## I. Mise en place du routage

### 1. Configuration du routage sur router

> On active la redirection de paquets depuis router avec la commande `sudo sysctl -w net.ipv4.conf.all.forwarding=1`

### 2. Ajouter des routes statiques

> Grace à la commande `sudo ip route add 10.3.1.0/24 via 10.3.2.254 dev enp0s8` client1 peut maintenant joindre net2 : ![](https://i.imgur.com/leKeXKs.png)

> Et server1 peut joindre net1 grace à la commande `sudo ip route add 10.3.2.0/24 via 10.3.1.254 dev enp0s8` : ![](https://i.imgur.com/lWDcV8J.png)

> Puis on vérifie le bon fonctionnemment avec un traceroute : ![](https://i.imgur.com/zWQgv8c.png)

### 3. Comprendre le routage

> Après avoir fait les manipulations, voici ce que cela donne dans wireshark : ![](https://i.imgur.com/ZxxkL0Z.png)


|                                         | MAC src | MAC dst | IP src | IP dst |
| --------------------------------------- | ------- | ------- | ------ | ------ |
| Dans net1 (trame qui entre dans router) |08:00:27:63:ff:39|08:00:27:22:f3:a8|10.3.1.11|10.3.1.254|
| Dans net2 (trame qui entre dans router) |08:00:27:e9:40:26|08:00:27:5d:e0:3c|10.3.2.254|10.3.2.11|

## II. ARP

### 1. Tables ARP

> On affiche les tables ARP de chaque machine via `ip n`
>
>
> Pour le client : ![](https://i.imgur.com/nEyhakL.png)
> 
> L'hôte avec l'ip 10.3.1.254 est sur le domaine de diffusion dev enp0s8 la MAC a été obtenue via le protocole ARP et la trame en sera composée a chaque fois qu'un paquet est émis a destination de 10.3.1.254. STALE indique que la connexion a été établie mais le voisin n'est plus joignable.
> 
> L'hôte avec l'ip 10.3.1.1 est sur le domaine de diffusion dev enp0s8 la MAC a été obtenue via le protocole ARP et la trame en sera composée a chaque fois qu'un paquet est émis a destination de 10.3.1.1. DELAY indique qu'un paquet a été transmis et est en attente d'une réponse.
>
> Pour le routeur : ![](https://i.imgur.com/rFFXlPt.png)
> 
> L'hôte avec l'ip 10.3.1.11 est sur le domaine de diffusion dev enp0s8 la MAC a été obtenue via le protocole ARP et la trame en sera composée a chaque fois qu'un paquet est émis a destination de 10.3.1.11. STALE indique que la connexion a été établie mais le voisin n'est plus joignable.
> 
> L'hôte avec l'ip 10.3.2.11 est sur le domaine de diffusion dev enp0s9 la MAC a été obtenue via le protocole ARP et la trame en sera composée a chaque fois qu'un paquet est émis a destination de 10.3.2.11. STALE indique que la connexion a été établie mais le voisin n'est plus joignable.
> 
> L'hôte avec l'ip 10.3.1.1 est sur le domaine de diffusion dev enp0s8 la MAC a été obtenue via le protocole ARP et la trame en sera composée a chaque fois qu'un paquet est émis a destination de 10.3.1.1. DELAY indique qu'un paquet a été transmis et est en attente d'une réponse.
>
> Pour le serveur : ![](https://i.imgur.com/RnmPIiI.png)
>
> L’hôte avec l’ip 10.3.2.1 est sur le domaine de diffusion dev enp0s8 la MAC a été obtenue via le protocole ARP et la trame en sera composée a chaque fois qu’un paquet est émis a destination de 10.3.2.1. DELAY indique qu'un paquet a été transmis et est en attente d'une réponse.
> 
> L’hôte avec l’ip 10.3.2.254 est sur le domaine de diffusion dev enp0s8 la MAC a été obtenue via le protocole ARP et la trame en sera composée a chaque fois qu’un paquet est émis a destination de 10.3.2.254. REACHABLE indique que la connexion a été établie et que l'hôte est apparement joignable.

### 2. Requêtes ARP

#### A. Table ARP 1

> On vide les tables ARP de router et client1 via la commande `ip n flush all`
> Table ARP avant flush : 
> ![](https://i.imgur.com/CKtKr04.png)
> 
> Table ARP après flush : 
> ![](https://i.imgur.com/pvSU2NI.png)

#### B. Table ARP 2

> Table ARP avant flush : 
> ![](https://i.imgur.com/gGwQs6B.png)
>
> Table ARP après flush : 
> ![](https://i.imgur.com/5ZtOYPr.png)

#### C. tcpdump 1

> Voici le rendu du `tcpdump` sur wireshark : ![](https://i.imgur.com/ChW5P0e.png)
>
> Sur la ligne 1, 10.3.1.11 demande qui a l'adresse 10.3.1.254
> Sur la ligne 2, 10.3.1.254 réponds en donnant la MAC correspondante
> Sur la ligne 3, 10.3.1.254 demande qui a l'adresse 10.3.1.11
> Sur la ligne 4, 10.3.1.11 réponds en donnant la MAC correspondante
> 

#### D. tcpdump 2

> Voici le rendu du `tcpdump` sur wireshark : ![](https://i.imgur.com/Ig2fnqF.png)
> Sur la ligne 1, 10.3.2.11 demande qui a l'adresse 10.3.2.254
> Sur la ligne 2, 10.3.2.254 réponds en donnant la MAC correspondante
> Sur la ligne 3, 10.3.2.254 demande qui a l'adresse 10.3.2.11
> Sur la ligne 4, 10.3.2.11 réponds en donnant la MAC correspondante
> 

#### E. u okay bro ?

> 1 : Client1 demande qui as l'ip de server1
> 2 : Router demande qui as l'ip demandée par client1
> 3 : Server1 donne l'adresse de l'ip à router
> 4 : Router donnne l'adresse de l'ip à client1

### Entracte : Donner un accès internet aux VMs

> On configure client1 en lui donnant la route par défaut 10.3.1.254, en modifiant le fichier /etc/sysconfig/network comme suit : `GATEWAY=10.3.1.254`
>
> On configure le server DNS en modifiant le fichier /etc/resolv.conf et en ajoutant la ligne : `nameserver 1.1.1.1` qui ajoute le DNS de cloudfare.
>
> Envoi d'un message simple vers un serveur en ligne : ![](https://i.imgur.com/Q1Kr9pX.png)
>
> Test de la résolution de nom DNS : 
> ![](https://i.imgur.com/F15ysCE.png)
>
> Test d'installation de paquet : 
> ![](https://i.imgur.com/w6kIkg5.png)
>
> Lancement de la locomotiiive : 
> ![](https://i.imgur.com/qlsApUa.png)

## III. Plus de tcpdump

### 1. TCP et UDP

#### A. Warm-up

- en UDP : 

```
[pverrier@server1 ~]# nc -l 10.3.2.11 9999
hello world
[pverrier@server1 ~]# 
```

```
[pverrier@client1 ~]# nc 10.3.2.11 9999
hello world
^C
[pverrier@client1 ~]# 
```

#### B. Analyse de trames

🌞 TCP : 

```
[pverrier@client1 ~]# nc 10.3.2.11 9999
Salut à tous
Comment c'est cool
La team est là
^C
[pverrier@client1 ~]#
```

```
[pverrier@server1 ~]# nc -l 10.3.2.11 9999
Salut à tous
Comment c'est cool
La team est là
[pverrier@server1 ~]#
```
>
> - Mise en évidence 3-way handshake TCP
>
>![](https://i.imgur.com/2vQh3fW.png)
>
> - observez la suite des échanges (PSH, ACK, etc)
>
>![](https://i.imgur.com/ITjMiAU.png)
>
> - observez la fin de connexion
>
>![](https://i.imgur.com/KVgPlan.png)
>
---
🌞 UDP : 

```
[pverrier@server1 ~]# nc -l 10.3.2.11 -u 9999
cc
cc toi
^C
[pverrier@server1 ~]#
```

```
[pverrier@client1 ~]# nc 10.3.2.11 -u 9999
cc
cc toi
^C
[pverrier@client1 ~]#
```
>
>![](https://i.imgur.com/C75diJa.png)
>
>La différence remarquable est qu'il n'y a pas le "3-way handshake > TCP"
>
>Du coup, l'UDP est plus rapide.

### 2. SSH

>Après s'être connecté en ssh sur notre serveur depuis le client. On observe >la connection ssh dans wireshark. On remarque tout de suite que le >protocole SSH utilise le TCP puisque qu'il n'y a aucune trame UDP.

## IV. Bonus

### ARP cache poisonning. 

>On part sur Arping
>
>Sur notre client pirate on effectue cette commande : `arping -c 1 -I enp0s8 10.3.1.11 (ip victime)`
>
>On va ensuite ping notre server 1 avec le client 1, et observer ce qui ce >passe dans wireshark
>
>
>![](https://i.imgur.com/w4mNnDD.png)
>
>
>On voit un "duplicate use of 10.3.1.12 detected" ce qui veut dire que notre >ARP cache poisoning a marcher mais que wireshark l'a detecté avec un niveau >de sécutité "WARINING"

### NGINX

>Apres avoir installé nginx on fait : `systemctl start nginx && enable nginx`
> Puis : `firewall-cmd --add-port=80/tcp --permanent
firewall-cmd --reload`
>
>
>On peut mettre notre ip serveur dans firefox et admirer le résultat : 
>
>![](https://i.imgur.com/zKxp2p0.png)
>
>
# FIN DU TP3