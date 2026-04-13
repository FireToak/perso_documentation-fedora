# Installation et configuration de Vagrant

## Informations

- **Auteur :** Louis MEDO
- **Date de création :** 13 avril 2026
- **Date de modification :** 13 avril 2026

---

## Contexte

Cette procédure a pour but de montrer l'installation et la configuration de Vagrant sur Linux Fedora en utilisant le moteur de virtualisation KVM, tout en respectant l'état de l'art. Elle inclut également la résolution du problème de communication NFS limitant le lancement des machines virtuelles.

---

## Installation de Vagrant

1. **Ajout du dépôt officiel HashiCorp et installation.** L'état de l'art recommande d'utiliser le dépôt officiel de l'éditeur pour garantir une version à jour et sécurisée de Vagrant, au lieu des paquets natifs potentiellement obsolètes.

    ```bash
    sudo dnf install -y dnf-plugins-core
    sudo dnf config-manager --add-repo https://rpm.releases.hashicorp.com/fedora/hashicorp.repo
    sudo dnf install -y vagrant
    ```

    - `dnf install -y dnf-plugins-core` : Installe le plugin fournissant la commande `config-manager`.
    - `config-manager --add-repo` : Ajoute l'URL spécifiée comme nouvelle source de paquets logiciels pour votre système.
    - `dnf install -y vagrant` : Installe le paquet principal Vagrant. Le paramètre `-y` valide automatiquement les invites de confirmation.

2. **Installation de l'hyperviseur KVM/Libvirt.** Vagrant a besoin d'un moteur de virtualisation sous-jacent. KVM (Kernel-based Virtual Machine) géré par Libvirt est le standard sous Linux.

    ```bash
    sudo dnf install -y @virtualization
    sudo systemctl enable --now libvirtd
    ```

    - `@virtualization` : Le préfixe `@` indique à `dnf` d'installer un "groupe" entier de paquets liés à la virtualisation (QEMU, Libvirt, virt-manager).
    - `systemctl enable --now libvirtd` : Active le service au démarrage du système (`enable`) et le lance immédiatement (`--now`).

3. **Installation du fournisseur (plugin) Vagrant-Libvirt.** Vagrant ne communique pas nativement avec KVM ; il nécessite un plugin spécifique et ses dépendances de compilation.

    ```bash
    sudo dnf install -y gcc libvirt-devel ruby-devel
    vagrant plugin install vagrant-libvirt
    ```

    - `gcc libvirt-devel ruby-devel` : Installe le compilateur C et les en-têtes de développement requis pour compiler le plugin en Ruby.
    - `vagrant plugin install` : Télécharge et installe le module d'extension permettant à Vagrant de piloter KVM.

---

## Configuration de Vagrant

1. **Gestion des droits utilisateur.** Pour éviter d'utiliser `sudo` avec Vagrant (ce qui est une mauvaise pratique de sécurité), l'utilisateur courant doit avoir les droits de pilotage de Libvirt.

    ```bash
    sudo usermod -aG libvirt $USER
    newgrp libvirt
    ```

    - `usermod -aG libvirt $USER` : Ajoute (`-a` pour append) l'utilisateur actuel (`$USER`) au groupe cible (`-G`) `libvirt`.
    - `newgrp libvirt` : Applique les nouveaux droits de groupe à la session en cours sans nécessiter de déconnexion.

2. **Installation et activation du serveur NFS.** La synchronisation de dossiers (`synced_folder`) sous Vagrant-Libvirt repose massivement sur NFS, qui n'est pas installé par défaut sous Fedora Desktop.

    ```bash
    sudo dnf install -y nfs-utils
    sudo systemctl enable --now nfs-server
    ```

    - `nfs-utils` : Paquet fournissant les démons serveur (`rpcbind`, `mountd`, etc.) permettant le partage de fichiers en réseau.
    - `systemctl enable --now nfs-server` : Automatise et démarre le daemon NFS en tâche de fond.

3. **Configuration du pare-feu (Firewalld) pour NFS.** L'erreur de lancement vient souvent du fait que la machine virtuelle ne peut pas contacter le serveur NFS de l'hôte à cause du pare-feu local. Il faut ouvrir les flux sur la zone réseau dédiée à Libvirt.

    ```bash
    sudo firewall-cmd --zone=libvirt --add-service=nfs --add-service=rpc-bind --add-service=mountd --permanent
    sudo firewall-cmd --reload
    ```

    - `firewall-cmd --zone=libvirt` : Cible l'interface réseau virtuelle générée par KVM (généralement `virbr0`).
    - `--add-service=...` : Autorise les protocoles standards requis par NFS à traverser le pare-feu.
    - `--permanent` : Enregistre la règle pour qu'elle survive aux redémarrages.
    - `firewall-cmd --reload` : Recharge la configuration du pare-feu pour appliquer immédiatement les règles persistantes.

4. **Configuration du fournisseur par défaut.** Pour éviter de devoir spécifier `--provider libvirt` à chaque commande, configurez Vagrant pour qu'il utilise KVM par défaut. Créez ou éditez le fichier `~/.vagrant.d/Vagrantfile`.

    ```ruby
    Vagrant.configure("2") do |config|
      config.vagrant.plugins = ["vagrant-libvirt"]
    end

    ENV['VAGRANT_DEFAULT_PROVIDER'] = 'libvirt'
Voici la procédure mise à jour selon l'état de l'art pour Fedora, intégrant la résolution spécifique de votre problème NFS.

plain text
# Installation et configuration de Vagrant

## Informations

- **Auteur :** Louis MEDO
- **Date de création :** 13 avril 2026
- **Date de modification :** 13 avril 2026

---

## Contexte

Cette procédure a pour but de montrer l'installation et la configuration de Vagrant sur Linux Fedora en utilisant le moteur de virtualisation KVM (via le plugin `vagrant-libvirt`). Elle inclut la résolution des problèmes de montage des dossiers partagés NFS, souvent dus à l'absence des services serveur NFS sur la machine hôte.

---

## Installation de Vagrant

1. **Installation des paquets de base et du plugin Libvirt.** Sur Fedora, Vagrant est disponible dans les dépôts officiels, mais le plugin pour KVM (`vagrant-libvirt`) doit être installé séparément. Il est recommandé d'installer également `gcc` et `libvirt-devel` pour compiler les dépendances natives du plugin.

    ```bash
    sudo dnf install -y vagrant libvirt-devel gcc ruby-libvirt
    vagrant plugin install vagrant-libvirt
    ```

2. **Configuration des droits utilisateur.** Pour éviter d'utiliser `sudo` à chaque commande Vagrant, ajoutez votre utilisateur au groupe `libvirt` et assurez-vous que le socket libvirt est accessible.

    ```bash
    sudo usermod -aG libvirt $(whoami)
    # Déconnectez-vous et reconnectez-vous pour appliquer le changement de groupe
    ```

---

## Configuration de Vagrant

1. **Installation et activation des services NFS (Résolution du blocage).** Le problème de montage NFS survient car Vagrant agit comme serveur NFS pour partager les dossiers de l'hôte vers la VM. Par défaut, `nfs-utils` (serveur) n'est pas toujours installé ou activé sur les postes de travail Fedora. Installez le paquet et activez le service.

    ```bash
    sudo dnf install -y nfs-utils
    sudo systemctl enable --now nfs-server
    sudo firewall-cmd --permanent --add-service=nfs
    sudo firewall-cmd --permanent --add-service=mountd
    sudo firewall-cmd --permanent --add-service=rpc-bind
    sudo firewall-cmd --reload
    ```

2. **Configuration du fournisseur par défaut.** Pour éviter de devoir spécifier `--provider libvirt` à chaque commande, configurez Vagrant pour qu'il utilise KVM par défaut. Créez ou éditez le fichier `~/.vagrant.d/Vagrantfile`.

    ```ruby
    Vagrant.configure("2") do |config|
      config.vagrant.plugins = ["vagrant-libvirt"]
    end

    ENV['VAGRANT_DEFAULT_PROVIDER'] = 'libvirt'
    ```

---

## Ressources

- [Documentation officielle Vagrant](https://developer.hashicorp.com/vagrant/docs)
- [Dépôt du plugin Vagrant-Libvirt](https://github.com/vagrant-libvirt/vagrant-libvirt)
- [Wiki Fedora - Virtualisation](https://fedoraproject.org/wiki/Virtualization)