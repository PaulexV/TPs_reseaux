# TP2 - Machine virtuelle, réseau, serveurs, routage simple

## I. Création et utilisation simples d'une VM CentOS

### 4. Configuration réseau d'une machine CentOS

| Name | IP | MAC | Fonction |
| --- | --- | --- | --- |
|`lo`|`127.0.0.1`|`00:00:00:00:00:00`|Adresse locale loopback|
|`enp0s3` | `10.0.2.15` | `08:00:27:79:3f:e4` | Carte NAT Adresse dynamique (accès internet) |
|`enp0s8` | `10.2.1.2` | `08:00:27:22:f3:a8` | Adresse Fixe (accès internet)|

---

> Avec la commande `sudo vim /etc/sysconfig/network-scripts/ifcfg-enp0s8`
> Je peux modifier l'adresse IP de ma carte enp0s8 en `10.2.1.25`, la manipulation prend effet après un redémarrage du réseau via `systemctl restart network`.

> Via la commande `ssh -l pverrier 10.2.1.2` je me connecte à ma VM et utilise `ping 10.2.1.2` pour ping ma VM depuis mon OS.
> 

### 5. Appréhension de quelques commandes

> Avec la commande `nmap -sP 10.2.1.0/24` on peut scanner le reseau actuel : 
> 
> ![](https://i.imgur.com/RpzLFtx.png)
> 
> On obtient alors une liste des IPs connectées au reseau.
> 
>Via la commande `ss -ltunp` on obtient : 
>![](https://i.imgur.com/6Istz4L.png)

>Lors d'un ping vers le PC hôte, l'utilisation de la commande `sudo tcpdump -iv 10.2.1.2` donne : ![](https://i.imgur.com/tBREurF.png)

## II. Notion de ports

### 1. SSH

> Via la commande `ss -ltunp` de tout à l'heure, on peut voir que le serveur ssh écoute actcuellement toutes les IPs sur le port 22.
> 
> Je me connecte à ma VM en ssh via la commande `ssh -l pverrier 10.2.1.2`
> 
> Puis, via netstat je vérifie bien que la connection en SSH est active ![](https://i.imgur.com/U2Qb3sJ.png)
> On voit ici que la connexion est bien "ESTABLISHED"

### 2. Firewall

#### A. SSH

> Dans le sshd_config, on change le Port 22 en Port 1025 : 
> ![](https://i.imgur.com/jOBnggR.png)

> On relance le service SSH via `sudo systemctl restart sshd` puis on vérifie la connexion via `ss -tlunp`, on voit bien que le port est 1025.
> ![](https://i.imgur.com/gCLP0tg.png)
>
> Puis on s'y connecte via `ssh -l pverrier 10.2.1.2 -p 1025`, mais la connexion est bloquée : 
> ![](https://i.imgur.com/YKLDMvF.png)
>
> On ouvre le port 1025 via la commande `sudo firewall-cmd --add-port=80/tcp --permanent`, on applique les changements avec `sudo firewall-cmd --reload` et on rééssaie la connexion :
> ![](https://i.imgur.com/bfHBj2q.png)
>
> Cette fois la connexion fonctionne.

#### B. Netcat

> Je me connecte et créé un chat sur le port 1025 ouvert précédement via la commande `nc -lp 1025`
> 
> Puis via netstat je vérifie que le connexion est bien établie : 
> ![](https://i.imgur.com/5Sxp7vp.png)

> En faisant la même manipulation via la commande `nc -lp 8080` sur la machine hôte et `nc 10.2.1.1 8080` je peux me servir de la VM comme client et de l'hôte comme serveur.

### 3. Wireshark

> Dans le premier cas on utilise la VM comme serveur : 
> ![](https://i.imgur.com/ssFJAzt.png)
> Le message envoyé est bien visible en clair ici : 
> ![](https://i.imgur.com/ydwxeIx.png)

> Puis on utilise la machine hôte comme serveur : 
> ![](https://i.imgur.com/FaVbgx8.png)
>
> Et encore une fois le message est visible en clair ici : 
> ![](https://i.imgur.com/SzvEejl.png)

> Ici, on voit bien l'échange de Syn, SynAck, Ack entre les deux machines : 
> ![](https://i.imgur.com/EGy81sF.png)

## III. Routage Statique

### 1. Préparation des hôtes

#### Check

> - PC1 et PC2 se ping en utilisant le réseau 12 : ![](https://i.imgur.com/vLcWAwb.png)

> - VM1 et PC1 se ping en utilisant le réseau 1 : ![](https://i.imgur.com/0mNMHi0.png)

> - VM2 et PC2 se ping en utilisant le réseau 2 : ![](https://i.imgur.com/cQdKkwt.png)

#### Activation du routage sur les PCs

> On active le routage via la commande :`sudo sysctl -w net.ipv4.conf.all.forwarding=1`

### 2. Configuration du routage

#### A. PC1

> PC1 accède déjà aux réseaux 1 et 12, il faut juste lui dire comment accéder au réseau 2 via la commande `route add 10.2.2.0/24 mask 255.255.255.0 10.2.12.2`

> Maintenant, PC1 ping 10.2.2.1 (l'adresse de PC2 dans 2) : ![](https://i.imgur.com/Ux2wUvK.png)

#### B. PC2

> On fait l'opération inverse, et maintenant, PC2 ping 10.2.1.1 (l'adresse de PC1 dans 1) : ![](https://i.imgur.com/1w2IzM5.png)

#### C. VM1

> Il faut dire à VM1 qu'elle peut joindre le réseau 2 en utilisant le lien qui l'unit avec PC1, on fait ceci via la commande `sudo ip route add 10.2.2.0/24 via 10.2.1.1 dev enp0s8`
>
> Sur Windows on désactive le firewall pour que tout fonctionne.
> 
> Maintenant, VM1 devrait pouvoir ping 10.2.2.1, l'adresse de PC2 dans le réseau 2 : ![](https://i.imgur.com/Ux3UKik.png)

> Traceroute : ![](https://i.imgur.com/0G6RReV.png)


#### D. VM2

> Il faut dire à VM2 qu'elle peut joindre le réseau 1 en utilisant le lien qui l'unit avec PC2, on fait ceci via la commande `sudo ip route add 10.2.1.0/24 via 10.2.2.1 dev enp0s8`
>
> Maintenant, VM2 devrait pouvoir ping 10.2.1.1, l'adresse de PC1 dans le réseau 1 : ![](https://i.imgur.com/FKcphBY.png)

> Traceroute : ![](https://i.imgur.com/pyC7XpS.png)


#### E. El gran final

> VM1 et VM2 se ping : ![](https://i.imgur.com/xzlpEnL.png)

### 3. Configuration des noms de domaine

> En modifiant les fichiers /etc/hosts des PCs et des VMs on peut déssormais ping entre les VMs via les noms de domaine : ![](https://i.imgur.com/eusocLx.png)

> Netcat : ![](https://i.imgur.com/AGzpOxm.png)

> Traceroute : ![](https://i.imgur.com/IJGZnJq.png)

> Et c'est fini !