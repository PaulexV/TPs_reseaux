# TP 4 - Cisco, Routage, DHCP

## 2. Mise en place

### B. Définition d'IPs statiques : 
### admin1 : 
> désativation SELINUX
>![](https://i.imgur.com/FSRGLZX.png)

>définition d'une IP statique
>![](https://i.imgur.com/eEVMkcU.png)

>définition d'un nom d'hôte
>![](https://i.imgur.com/zHv6Fuw.png)


### routeur 1 : 
>définition d'une IP statique
>![](https://i.imgur.com/ecrWDNE.png)

>définition d'un nom
>![](https://i.imgur.com/tIxYWb4.png)


### guest1 : 
>définition d'une IP statique
>![](https://i.imgur.com/M3dFb52.png)


>définition d'un nom (depuis l'interface de GNS3)
>![](https://i.imgur.com/DtOyHkD.png)

### Vérification : 
> * guest1 peut joindre le routeur : ![](https://i.imgur.com/BVLoPyI.png)


> * admin1 peut joindre le routeur : ![](https://i.imgur.com/RRagTHI.png)



> * router1 peut joindre les deux autres machines
> 
>    * routeur vers ADMIN :![](https://i.imgur.com/x7L43JC.png)
>    * routeur vers GUEST :![](https://i.imgur.com/2nk5fvC.png)

> * table ARP
>     * ![](https://i.imgur.com/PGWwS3L.png)

### C. Routage : 

> ``show ip route``
> * résultat de la commande : 
> ![](https://i.imgur.com/gAS6TUl.png)


> * ADMIN vers GUEST : 
> ![](https://i.imgur.com/1yKTLNN.png)

> * GUEST vers ADMIN: 
> ![](https://i.imgur.com/NF3n8RB.png)


> * vérification : 
>     * traceroute ADMIN vers GUEST : ![](https://i.imgur.com/DvBmSJy.png)
>     * traceroute GUEST vers ADMIN : ![](https://i.imgur.com/S3rd6f9.png)

## II. Topologie 2 : dumb switches 

### 2. Mise en place : 

#### C. Vérification : 

> * GUEST vers ADMIN :![](https://i.imgur.com/ZVaNgRQ.png)

> * ADMIN vers GUEST :![](https://i.imgur.com/duWQibL.png)

## III. Topologie 3 : adding nodes and NAT

### 2. Mise en place : 

> * GUEST2 vers ADMIN :![](https://i.imgur.com/KbP63iY.png)
> * GUEST3 vers ADMIN :![](https://i.imgur.com/4xSfHWo.png)

#### Configurer le routeur :
> * récupérer une IP en DHCP : ![](https://i.imgur.com/dOqHW0C.png)
> * configurer un NAT simple sur le routeur : ![](https://i.imgur.com/n7AN6va.png)

#### Configurer les clients : 
> * DNS pour les GUESTS :
>     * ![](https://i.imgur.com/fmFJx4V.png)
>     * ![](https://i.imgur.com/XmWRzJk.png)
>     * ![](https://i.imgur.com/T9d5lSy.png)

> * DNS pour ADMIN : 
>     * ![](https://i.imgur.com/jp52ShV.png)

#### Vérification : 
> * le routeur a un accès WAN : 
>     * ![](https://i.imgur.com/Mjmu3FJ.png)
    
> * tous les guest et admin ont un réseau WAN : 
>     * guest1: ![](https://i.imgur.com/tYOcixF.png)
>     * guest2: ![](https://i.imgur.com/paoEYpt.png)
>     * guest3: ![](https://i.imgur.com/GGQU1yo.png)
>     * admin: ![](https://i.imgur.com/fQidv3S.png)

> * tous les clients du réseau ont de la résolution de noms grâce au serveur > DNS configuré : ![](https://i.imgur.com/TB19jRJ.png)

## IV. Topologie 4 : home-made DHCP

> * vous avez des ports UDP en écoute : ![](https://i.imgur.com/96wRMc9.png)


> * attribuez une IP en DHCP à l'un des membres du réseau guests:![](https://i.imgur.com/UjhJdG3.png)

> * observer et mettre en évidence à l'aide d'un tcpdump sur la machine dhcp les échanges DHCP : ![](https://i.imgur.com/MbVldpw.png)

# FIN