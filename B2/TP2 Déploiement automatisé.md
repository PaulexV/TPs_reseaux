# TP2 : Déploiement automatisé

Le but de ce TP est d'effectuer le même déploiement que lors du TP1 mais en automatisant le déploiement de la machine virtuelle, sa configuration basique, ainsi que l'install et la conf des services.


# I. Déploiement simple

🌞 Créer un `Vagrantfile` qui :
* utilise la box `centos/7`
* crée une seule VM
  * 1Go RAM
  * ajout d'une IP statique `192.168.2.11/24`
  * définition d'un nom (interne à Vagrant)
  * définition d'un hostname 
  * ajout d'un disque de 5Go

> Voici ce que donne le Vagrantfile fini : 
```
Vagrant.configure("2")do|config|
  config.vm.box="centos/7"

  # Ajoutez cette ligne afin d'accélérer le démarrage de la VM (si une erreur 'vbguest' est levée, voir la note un peu plus bas)
  config.vbguest.auto_update = false

  # Désactive les updates auto qui peuvent ralentir le lancement de la machine
  config.vm.box_check_update = false 

  # La ligne suivante permet de désactiver le montage d'un dossier partagé (ne marche pas tout le temps directement suivant vos OS, versions d'OS, etc.)
  config.vm.synced_folder ".", "/vagrant", disabled: true

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  config.vm.network "private_network", ip: "192.168.2.11"

  config.vm.define "Centos7"

  config.vm.hostname = "CentosVagrant"

  config.vm.provider "virtualbox" do |vb|
  # Customize the amount of memory on the VM:
     vb.memory = "1024"
     vb.name = "CentosVagrant"
     end

  # Exécution d'un script au démarrage de la VM
  config.vm.provision :shell, path: "script.sh", run: 'always'

  file_to_disk = 'VagrantDisk.vdi'

  config.vm.provider "virtualbox" do |vb|
    vb.customize ['createhd', '--filename', file_to_disk, '--size', 5 * 1024]
    vb.customize ['storageattach', :id, '--storagectl', 'IDE', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', file_to_disk]
  end
end

```

# III. Multi-node deployment

Il est possible de déployer et gérer plusieurs VMs en un seul `Vagrantfile`.


🌞 Créer un `Vagrantfile` qui lance deux machines virtuelles, **les VMs DOIVENT utiliser votre box repackagée comme base** :

| x           | `node1.tp2.b2` | `node2.tp2.b2` |
|-------------|----------------|----------------|
| IP locale   | `192.168.2.21` | `192.168.2.22` |
| Hostname    | `node1.tp2.b2` | `node2.tp2.b2` |
| Nom Vagrant | `node1`        | `node2`        |
| RAM         | 1Go            | 512Mo          |

> Voici donc le Vagrantfile permetant de lancer 2 VMs en meme temps : 

```
Vagrant.configure("2") do |config|
  # Configuration commune à toutes les machines
  config.vm.box = "../centos_patron/centos7-custom.box"

  # Ajoutez cette ligne afin d'accélérer le démarrage de la VM (si une erreur 'vbguest' est levée, voir la note un peu plus bas)
  config.vbguest.auto_update = false

  # Désactive les updates auto qui peuvent ralentir le lancement de la machine
  config.vm.box_check_update = false 

  # La ligne suivante permet de désactiver le montage d'un dossier partagé (ne marche pas tout le temps directement suivant vos OS, versions d'OS, etc.)
  config.vm.synced_folder ".", "/vagrant", disabled: true

  # Config une première VM "node1"
  config.vm.define "node1" do |node1|
    # remarquez l'utilisation de 'node1.' défini sur la ligne au dessus
    node1.vm.network "private_network", ip: "192.168.2.21"
    node1.vm.hostname = "node1.tp2.b2"
    node1.vm.provider "node1" do |n1|
  # Customize the amount of memory on the VM:
      n1.vm.memory = "1024"
    end
  end

  # Config une première VM "node2"
  config.vm.define "node2" do |node2|
    # remarquez l'utilisation de 'node2.' défini sur la ligne au dessus
    node2.vm.network "private_network", ip: "192.168.2.22"
    node2.vm.hostname = "node2.tp2.b2"
    node2.vm.provider "node2" do |n2|
  # Customize the amount of memory on the VM:
      n2.vm.memory = "512"
    end
  end
end

```

# IV. Automation here we (slowly) come

Cette dernière étape vise à automatiser la résolution du TP1 à l'aide de Vagrant et d'un peu de scripting.

🌞 Créer un `Vagrantfile` qui automatise la résolution du TP1

Voici le Vagrantfile automatisant la résolution du TP1, globalement, il paramètre la VM et execute un script.

```
Vagrant.configure("2") do |config|
  # Configuration commune à toutes les machines
  config.vm.box = "../centos_patron/centos7-custom.box"

  # Ajoutez cette ligne afin d'accélérer le démarrage de la VM (si une erreur 'vbguest' est levée, voir la note un peu plus bas)
  config.vbguest.auto_update = false

  # Désactive les updates auto qui peuvent ralentir le lancement de la machine
  config.vm.box_check_update = false 

  # La ligne suivante permet de désactiver le montage d'un dossier partagé (ne marche pas tout le temps directement suivant vos OS, versions d'OS, etc.)
  config.vm.synced_folder ".", "/vagrant", disabled: true

  # Config une première VM "node1"
  config.vm.define "node1" do |node1|
    # remarquez l'utilisation de 'node1.' défini sur la ligne au dessus
    node1.vm.network "private_network", ip: "192.168.2.21"
    node1.vm.hostname = "node1.tp2.b2"
    node1.vm.provider "node1" do |n1|
  # Customize the amount of memory on the VM:
      n1.vm.memory = "1024"
    end
  end

  # Config une première VM "node2"
  config.vm.define "node2" do |node2|
    # remarquez l'utilisation de 'node2.' défini sur la ligne au dessus
    node2.vm.network "private_network", ip: "192.168.2.22"
    node2.vm.hostname = "node2.tp2.b2"
    node2.vm.provider "node2" do |n2|
  # Customize the amount of memory on the VM:
      n2.vm.memory = "512"
    end
  end
    # Exécution d'un script au démarrage de la VM
  config.vm.provision :shell, path: "script.sh", run: 'always'
end

```
