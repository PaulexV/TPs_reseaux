# TP1 : Déploiement classique

Le but de ce TP est d'effectuer le déploiement de services réseau assez classiques, ainsi que réaliser une configuration système élémentaire (réseau, stockage, utilisateurs, etc.).

# 0. Prérequis

🌞 **Setup de deux machines CentOS7 configurées de façon basique.**
* partitionnement
  * ajouter un deuxième disque de 5Go à la machine
  * partitionner le nouveau disque avec LVM
    * deux partitions, une de 2Go, une de 3Go
    * la partition de 2Go sera montée sur `/srv/site1`
    * la partition de 3Go sera montée sur `/srv/site2`
```
lvdisplay
  --- Logical volume ---
  LV Path                /dev/srv/site1
  LV Name                data1
  VG Name                srv
  LV UUID                BzRbSz-es9Y-VHDH-gQ1w-e2zZ-QCbg-JmUYZU
  LV Write Access        read/write
  LV Creation host, time node1.tp1.b2, 2020-09-30 12:30:18 +0200
  LV Status              available
  # open                 0
  LV Size                2,00 GiB
  Current LE             512
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:2
   
  --- Logical volume ---
  LV Path                /dev/srv/site2
  LV Name                data2
  VG Name                srv
  LV UUID                X6zlQ8-9yKQ-1bzl-VJzY-h5SK-GX1x-ydG6Kj
  LV Write Access        read/write
  LV Creation host, time node1.tp1.b2, 2020-09-30 12:32:26 +0200
  LV Status              available
  # open                 0
  LV Size                3,00 GiB
  Current LE             768
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  ```
  > les partitions doivent être modifiées automatiquement, on modifie donc le fstab : 
  
  ```
# /etc/fstab
# Created by anaconda on Wed Sep 30 22:04:55 2020
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
UUID=1c419d6c-5064-4a2b-953c-05b2c67edb15 /                       xfs     defaults        0 0
/dev/srv/site1 /mnt/site1 ext4 defaults 0 0
/dev/srv/site2 /mnt/site2 ext4 defaults 0 0
  ```
  
  > Ensuite, on crée un système de fichier sur ces partitions, sinon le boot fail.
  
* un accès internet
  * carte réseau dédiée
  * route par défaut
 ```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.1.11 node1.tp1.b2
```
* un accès à un réseau local (les deux machines peuvent se `ping`) 
  * carte réseau dédiée 
  * route locale
* les machines doivent avoir un nom 
* les machines doivent pouvoir se joindre par leurs noms respectifs 
```
[paulex@node1 /]$ ping node2.tp1.b2
PING node2.tp1.b2 (192.168.1.12) 56(84) bytes of data.
64 bytes from node2.tp1.b2 (192.168.1.12): icmp_seq=1 ttl=64 time=2.59 ms
64 bytes from node2.tp1.b2 (192.168.1.12): icmp_seq=2 ttl=64 time=1.40 ms
64 bytes from node2.tp1.b2 (192.168.1.12): icmp_seq=3 ttl=64 time=1.21 ms
```

* le pare-feu est configuré pour bloquer toutes les connexions exceptées celles qui sont nécessaires

Pour bloquer toutes les communications, on utilise cette commande : 
``sudo firewall-cmd --set-default-zone block``

Puis, indépendemment, on autorise le port 22, utilisé pour SSH à communiquer : 
`firewall-cmd --zone=public --add-port=22/tcp --permanent`

On ajoute les deux nodes dans la listes des appareils autorisés à communiquer en les rajoutant dans les zones "trusted" des firewall avec la commande `firewall-cmd --permanent --zone=trusted --add-source=192.168.1.1X`.

Puis on reload le firewall.
```
sudo firewall-cmd --list-all
block (active)
  target: %%REJECT%%
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources: 
  services: 
  ports: 
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 

```
* un utilisateur administrateur est créé sur les deux machines (il peut exécuter des commandes `sudo` en tant que `root`)
```
root    ALL=(ALL)       ALL
user1 ALL=(ALL) ALL
```
* désactiver SELinux
Ça on à l'habitude, on utilise `sudo setenforce 0`, puis dans le fichier de config, on le passe de `enforcing` à `permmissive`.


# I. Setup serveur Web

🌞 Installer le serveur web NGINX sur `node1.tp1.b2`

🌞 Faire en sorte que :
* NGINX servent deux sites web, chacun possède un fichier unique `index.html`

```
[paulex@node1 ~]$ curl -L node1.tp1.b2/site1
site1
[paulex@node1 ~]$ curl -L node1.tp1.b2/site2
site2
[paulex@node1 ~]$
```
* les sites web doivent se trouver dans `/srv/site1` et `/srv/site2`
  * les permissions sur ces dossiers doivent être le plus restrictif possible
  * ces dossiers doivent appartenir à un utilisateur et un groupe spécifique
```
[paulex@node1 ~]$ ls -la /srv/
total 0
drwxrwxrwx.  4 tp  tp   32 30 sept. 14:56 .
dr-xr-xr-x. 17 root root 242 29 sept. 14:26 ..
drwx------.  2 tp  tp   24 30 sept. 14:46 site1
drwx------.  2 tp  tp   24 30 sept. 14:46 site2
[paulex@node1 ~]$
```
* NGINX doit utiliser un utilisateur dédié que vous avez créé à cet effet
```
[paulex@node1 ~]$ sudo cat /etc/nginx/nginx.conf | grep user
user tp;
```
---

# II. Script de sauvegarde


🌞 Ecrire un script qui :
* s'appelle `tp1_backup.sh`
* sauvegarde un dossier donné
* les noms des archives doivent contenir le nom du site sauvegardé ainsi que la date et heure de la sauvegarde
* garde que 7 exemplaires sauvegardes
* le script ne sauvegarde qu'un dossier à la fois, le chemin vers ce dossier est passé en argument du script

```bash
#!/bin/bash
# Script pour le TP backup de site1 et site2

if [ -z $1 ]
then
        echo "Which site do you want to save"
        exit 7
elif [ $1 != "site1" -a $1 != "site2" ]
then
        echo "error : please indicate a site"
        exit 7
elif [ "$1" == "site1" ] 
then
        tar -czf site1_$(date +"%Y%m%d_%H%M%S").tar.gz /srv/data1
        if [[ $(ls -1 /home/backup/site1 | wc -l) == 8 ]]
        then
        rm "$(ls -t | tail -1)"
        echo "Too many backups, erased oldest one" 
        fi
        mv /home/backup/*.gz /home/backup/site1
elif [ "$1" == "site2" ]
then
        tar -czf site2_$(date +"%Y%m%d_%H%M%S").tar.gz /srv/data2
        if [[ $(ls -1 /home/backup/site2 | wc -l) == 8 ]]
        then
        rm "$(ls -t | tail -1)"
        echo "Too many backups, erased oldest one"
        fi
        mv /home/backup/*.gz /home/backup/site2
fi
```

🌞 Utiliser la `crontab` pour que le script s'exécute automatiquement toutes les heures.

```bash
00 */1 * * * /home/backup/tp1_backup.sh site1

00 */1 * * * /home/backup/tp1_backup.sh site2
```
# III. Monitoring, alerting

🌞 Mettre en place l'outil Netdata

On installe Netdata avec la commande suivante : 
```bash
bash <(curl -Ss https://my-netdata.io/kickstart.sh)
```
