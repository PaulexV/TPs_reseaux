# TP 5 - Une "vraie" topologie ?

## I. Toplogie 1 - intro VLAN

### 1. Mise en place de la topologie

> On met en place la topologie suivante : ![](https://i.imgur.com/3e0fin6.png)

### 2. Setup Clients

> admin1 ping admin2 :
> ![](https://i.imgur.com/iF3oa2m.png)

> guest1 ping guest2 :
> ![](https://i.imgur.com/2G8Ppb7.png)

### 3. setup VLANs

> Mise en place des VLANs sur le switch1 :
> ![](https://i.imgur.com/Rw80MWd.png)

> Mise en place des VLANs sur le switch2 : 
> ![](https://i.imgur.com/xCGBXZz.png)

> Puis, on vérifie que tout le monde se ping correctement.

> admin1 ping admin2 : 
> ![](https://i.imgur.com/Z6dkk9Y.png)

> guest1 ping guest2 : 
> ![](https://i.imgur.com/jhwE8a9.png)

> Enfin, on vérifie que si un guest change d'ip, il ne peut tout de même pas joindre les admins :
> ![](https://i.imgur.com/laq1jBJ.png)

## II. Topologie 2 - VLAN, sous-interface, NAT

### 1. Mise en place de la topologie

> On met en place la topologie suivante : 
> ![](https://i.imgur.com/3poGdGV.png)

### 2. Adressage

> admin3 ping admin1 : 
> ![](https://i.imgur.com/Jblfdgg.png)

> guest3 ping guest1 : 
> ![](https://i.imgur.com/5PIVmBB.png)

### 3. VLAN

> Les VLANs du switch 1 sont inchangées.
> 
> On configure les VLANs et trunks du switch2 : 
> ![](https://i.imgur.com/1RtS4Py.png)

> On configure également le switch3, ses VLANs et trunks : 
> ![](https://i.imgur.com/PBfI1o4.png)

> Même si guest3 change d'ip, il ne peut pas joindre les admins : 
> ![](https://i.imgur.com/Xo0NHJT.png)

### 4. Sous-interfaces

> On configure les interfaces du routeur en sous-interfaces afin de pouvoir joindre nos deux réseaux.
> 
> Les admins ping leur passerelle : ![](https://i.imgur.com/FzJQDOq.png)
>
> Les guests ping leur passerelle : ![](https://i.imgur.com/UIGUJAS.png)

### 5. NAT

> A partir de là je n'arrive pas à donner l'acces à mes machines.