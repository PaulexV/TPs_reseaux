# TP3 : systemd

Le but ici c'est d'explorer un peu systemd. 

systemd est un outil qui a été très largement adopté au sein des distributions GNU/Linux les plus répandues (Debian, RedHat, Arch, etc.). systemd occupe plusieurs fonctions :
* système d'init
* gestion de services
* embarque plusieurs applications très proche du noyau et nécessaires au bon fonctionnement du système
  * comme par exemple la gestion de la date et de l'heure, ou encore la gestion des périphériques
* PID 1

Ce TP3 a donc pour objectif d'explorer un peu ces différentes facettes. La finalité derrière tout ça est de vous faire un peu mieux appréhender comment marche un OS GNU/Linux ; mais aussi de façon plus générale vous faire mieux appréhender en quoi consiste l'application qu'on appelle "système d'exploitation" (car ui, c'est juste une application).

# I. Services systemd

## 1. Intro

🌞 Utiliser la ligne de commande pour sortir les infos suivantes :
* Afficher le nombre de *services systemd* dispos sur la machine
```bash
[vagrant@CentosTP3 ~]$ systemctl list-units --type=service | grep systemd | wc -l
13
```
* Afficher le nombre de *services systemd* actifs *("running")* sur la machine
```bash
[vagrant@CentosTP3 ~]$ systemctl list-units --type=service --state=running | grep systemd | wc -l
3
```
* Afficher le nombre de *services systemd* qui ont échoué *("failed")* ou qui sont inactifs *("exited")* sur la machine
```bash
[vagrant@CentosTP3 ~]$ systemctl list-units --type=service --state=failed,exited | grep systemd | wc -l
10
```
* Afficher la liste des *services systemd* qui démarrent automatiquement au boot *("enabled")*
```bash
[vagrant@CentosTP3 ~]$ systemctl list-unit-files --type=service | grep enabled
auditd.service                                enabled 
autovt@.service                               enabled 
chronyd.service                               enabled 
crond.service                                 enabled 
dbus-org.fedoraproject.FirewallD1.service     enabled 
dbus-org.freedesktop.nm-dispatcher.service    enabled 
firewalld.service                             enabled 
getty@.service                                enabled 
irqbalance.service                            enabled 
NetworkManager-dispatcher.service             enabled 
NetworkManager-wait-online.service            enabled 
NetworkManager.service                        enabled 
postfix.service                               enabled 
qemu-guest-agent.service                      enabled 
rhel-autorelabel-mark.service                 enabled 
rhel-autorelabel.service                      enabled 
rhel-configure.service                        enabled 
rhel-dmesg.service                            enabled 
rhel-domainname.service                       enabled 
rhel-import-state.service                     enabled 
rhel-loadmodules.service                      enabled 
rhel-readonly.service                         enabled 
rpcbind.service                               enabled 
rsyslog.service                               enabled 
sshd.service                                  enabled 
systemd-readahead-collect.service             enabled 
systemd-readahead-drop.service                enabled 
systemd-readahead-replay.service              enabled 
tuned.service                                 enabled 
vgauthd.service                               enabled 
vmtoolsd-init.service                         enabled 
vmtoolsd.service                              enabled 
```
---

## 2. Analyse d'un service

🌞 Etudier le service `nginx.service`
* déterminer le path de l'unité `nginx.service`
> la commande `systemctl cat nginx` nous affiche son chemin, il s'agit de :
> `# /usr/lib/systemd/system/nginx.service`
* afficher son contenu et expliquer les lignes qui comportent :
  * `ExecStart`
  * `ExecStartPre`
  * `PIDFile`
  * `Type`
  * `ExecReload`
  * `Description`
  * `After`
```bash
[Unit]
# Décrit le nom et l'utilité de l'unité
Description=The nginx HTTP and reverse proxy server
# L'unité démarrera après que les unités listées ici aient étées démarrées
After=network.target remote-fs.target nss-lookup.target

[Service]
# Catégorise le type du service en fonction de leur comportement
Type=forking
# Si le type de service est "forking", il permet de préciser le process enfant à surveiller
PIDFile=/run/nginx.pid
# Nginx will fail to start if /run/nginx.pid already exists but has the wrong
# SELinux context. This might happen when running `nginx -t` from the cmdline.
# https://bugzilla.redhat.com/show_bug.cgi?id=1268621

# Permet de donner des commandes supplémentaires à executer avant le main process
ExecStartPre=/usr/bin/rm -f /run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t
# Précise le PATH de l'unité à démarrer
ExecStart=/usr/sbin/nginx
# Indique la commande à utiliser pour relancer la configuration
ExecReload=/bin/kill -s HUP $MAINPID
KillSignal=SIGQUIT
TimeoutStopSec=5
KillMode=process
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```
🌞 Lister tous les services qui contiennent la ligne `WantedBy=multi-user.target`

```bash
[vagrant@CentosTP3 system]$ grep WantedBy=multi-user.target /etc/systemd/system/*/* | grep service| cut -d '/' -f6 | cut -d ':' -f1
auditd.service
chronyd.service
crond.service
firewalld.service
irqbalance.service
NetworkManager.service
postfix.service
rhel-configure.service
rpcbind.service
rsyslog.service
sshd.service
tuned.service
vmtoolsd.service
```
## 3. Création d'un service

### A. Serveur web

🌞 Créer une unité de service qui lance un serveur web
```bash
[Unit]
Description=Start a webserver

[Service]
User=web
Environment="PORT=8080"
Type=simple
ExecStartPre=/usr/bin/sudo /usr/bin/firewall-cmd --add-port=${PORT}/tcp
ExecStart=/usr/bin/python2 -m SimpleHTTPServer ${PORT}
ExecStopPost=/usr/bin/sudo /usr/bin/firewall-cmd --remove-port=${PORT}/tcp

[Install]
WantedBy=multi-user.target
```

🌞 Lancer le service
* prouver qu'il est en cours de fonctionnement pour systemd
```bash
[vagrant@CentosTP3 ~]$ sudo systemctl status webserver.service
● webserver.service - Start a webserver
   Loaded: loaded (/usr/lib/systemd/system/webserver.service; disabled; vendor preset: disabled)
   Active: active (running) since Wed 2020-10-07 07:44:32 UTC; 36s ago
  Process: 3041 ExecStartPre=/usr/bin/sudo /usr/bin/firewall-cmd --add-port=${PORT}/tcp (code=exited, status=0/SUCCESS)
 Main PID: 3048 (python2)
   CGroup: /system.slice/webserver.service
           └─3048 /usr/bin/python2 -m SimpleHTTPServer 8080
```
* prouver que le serveur web est bien fonctionnel
```bash
[vagrant@CentosTP3 ~]$ curl 0.0.0.0:8080
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 3.2 Final//EN"><html>
<title>Directory listing for /</title>
<body>
<h2>Directory listing for /</h2>
<hr>
<ul>
<li><a href="bin/">bin@</a>
<li><a href="boot/">boot/</a>
<li><a href="dev/">dev/</a>
<li><a href="etc/">etc/</a>
<li><a href="home/">home/</a>
<li><a href="lib/">lib@</a>
<li><a href="lib64/">lib64@</a>
<li><a href="media/">media/</a>
<li><a href="mnt/">mnt/</a>
<li><a href="opt/">opt/</a>
<li><a href="proc/">proc/</a>
<li><a href="root/">root/</a>
<li><a href="run/">run/</a>
<li><a href="sbin/">sbin@</a>
<li><a href="srv/">srv/</a>
<li><a href="swapfile">swapfile</a>
<li><a href="sys/">sys/</a>
<li><a href="tmp/">tmp/</a>
<li><a href="usr/">usr/</a>
<li><a href="var/">var/</a>
</ul>
<hr>
</body>
</html>
```

### B. Sauvegarde

🌞 Créez une unité de service qui déclenche une sauvegarde avec votre script
```bash
[Unit]  
Description=Service used to create backups  

[Service]
User=web
PIDFile=/var/run/backup.pid
ExecStartPre=/usr/bin/sudo /usr/bin/sh /home/vagrant/scripts/pre.sh
ExecStart=/usr/bin/sudo /usr/bin/sh /home/vagrant/scripts/backup.sh
ExecStartPost=/usr/bin/sudo /usr/bin/sh /home/vagrant/scripts/post.sh

[Install]
WantedBy=multi-user.target
```

🌞 Ecrire un fichier .timer systemd

```bash
[Unit]
Description=Times when the backup needs to be done
Requires=backup.service

[Timer]
Unit=backup.service
OnCalendar=0/1:00:00

[Install]
WantedBy=timers.target
```
# II. Autres features

## 1. Gestion de boot

🌞 Utilisez `systemd-analyze plot` pour récupérer une diagramme du boot, au format SVG
* il est possible de rediriger l'output de cette commande pour créer un fichier `.svg`
  * un `.svg` ça peut se lire avec un navigateur
* déterminer les 3 **services** les plus lents à démarrer

Après avoir scp le plot sur la machine host, on se retrouve avec ça : 

![](https://i.imgur.com/wqo9ZfJ.png)

On remarque donc que les services prenant le plus de temps à se lancer sont: 
* sssd.service
* sshd-keygen@rsa.service
* tuned.service

## 2. Gestion de l'heure

🌞 Utilisez la commande `timedatectl`
```bash
[vagrant@localhost ~]$ timedatectl
               Local time: Fri 2020-10-09 14:24:22 UTC
           Universal time: Fri 2020-10-09 14:24:22 UTC
                 RTC time: Fri 2020-10-09 14:24:21
                Time zone: UTC (UTC, +0000)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```
* déterminer votre fuseau horaire
UTC indique que noous sommes dans la timezone d'europe de l'ouest.
* déterminer si vous êtes synchronisés avec un serveur NTP
Nous sommes synchronisés au serveur NTP comme l'indique ``NTP service: active``
* changer le fuseau horaire
```bash
[vagrant@localhost ~]$ sudo timedatectl set-timezone Africa/Abidjan
[vagrant@localhost ~]$ timedatectl
               Local time: Fri 2020-10-09 14:31:39 GMT
           Universal time: Fri 2020-10-09 14:31:39 UTC
                 RTC time: Fri 2020-10-09 14:31:38
                Time zone: Africa/Abidjan (GMT, +0000)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```

## 3. Gestion des noms et de la résolution de noms

🌞 Utilisez `hostnamectl`

```bash
[vagrant@localhost ~]$ hostnamectl
   Static hostname: localhost.localdomain
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 4816289300e4462097bd27f527172474
           Boot ID: 6162b794a8684397bf685c4c9460e95a
    Virtualization: oracle
  Operating System: CentOS Linux 8 (Core)
       CPE OS Name: cpe:/o:centos:centos:8
            Kernel: Linux 4.18.0-80.el8.x86_64
      Architecture: x86-64
```
* déterminer votre hostname actuel
Le hostname est `localhost.localdomain` comme indiqué par `Static hostname`
* changer votre hostname
```bash
[vagrant@localhost ~]$ sudo hostnamectl set-hostname centos8
[vagrant@localhost ~]$ hostnamectl
   Static hostname: centos8
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 4816289300e4462097bd27f527172474
           Boot ID: 6162b794a8684397bf685c4c9460e95a
    Virtualization: oracle
  Operating System: CentOS Linux 8 (Core)
       CPE OS Name: cpe:/o:centos:centos:8
            Kernel: Linux 4.18.0-80.el8.x86_64
      Architecture: x86-64
```