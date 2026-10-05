# Simple Manageable Linux Machine Parc (Vagrant + Ansible)

Comment déployer et gérer rapidement un parc de machines virtuelles sécurisées avec des configurations partagées ?
Ce projet répond à ce besoin en automatisant la création et la configuration d'une infrastructure locale avec Vagrant et Ansible.

## 1. À propos du projet

### Cas d'usage

Imaginez devoir tester un environnement réseau comprenant un serveur et plusieurs postes clients. Le faire manuellement demande de configurer les adresses IP, d'installer les outils de base, de gérer les clés SSH, de créer les utilisateurs et de configurer les pare-feux pour chaque machine.

Au lieu de tout configurer à la main, ce projet définit **une infrastructure sous forme de code** (IaC) et gère :

* **L'approvisionnement** : Création automatique d'un serveur et de plusieurs clients avec leurs IP dédiées.

* **La sécurité et les permissions** : Création d'utilisateurs restreints (sans droits root) et isolation par pare-feu.

* **Le partage de données** : Mise en place d'un dossier commun accessible par toutes les machines via NFS.

### Architecture et sources

Le parc fonctionne avec le fournisseur `vmware_desktop` et utilise la distribution `bento/ubuntu-26.04` en architecture `arm64`.

| Machine | Rôle | Adresse IP | Ressources | 
| ----- | ----- | ----- | ----- | 
| `serveur-central` | Serveur de partage et de gestion | 192.168.56.10 | 1 vCPU, 512 Mo RAM | 
| `client-User1` | Poste client pour l'utilisateur 1 | 192.168.56.11 | 1 vCPU, 512 Mo RAM | 
| `client-User2` | Poste client pour l'utilisateur 2 | 192.168.56.12 | 1 vCPU, 512 Mo RAM | 

*Note: Toutes les informations d'architecture proviennent du fichier de configuration Vagrant.*

### Outils

* **Vagrant** (Orchestration des VMs)

* **VMware** (Hyperviseur avec interface graphique activée via `v.gui = true`)

* **Ansible** (Provisionnement, paquets, utilisateurs, NFS, UFW)

### Fichiers de configuration Ansible

| Ordre | Fichier | Objectif et composants gérés | 
| ----- | ----- | ----- | 
| 1 | `playbook.yml` | Installation des outils de base (`curl`, `wget`, `nano`, `net-tools`) et création des utilisateurs. | 
| 2 | `nfs.yml` | Configuration d'un dossier partagé pour avoir la main sur un répertoire commun. | 
| 3 | `firewall.yml` | Sécurisation des ports de communication avec UFW. | 

## 2. Analyse des configurations (Ansible)

### 2.1 Utilisateurs et permissions (Zero-Trust basique)

**Méthode :** Le provisionnement Ansible utilise `become: true` pour exécuter les tâches avec les droits administrateur, mais crée des environnements utilisateurs restreints.

* L'utilisateur `user1` est créé exclusivement sur `client-User1`, et `user2` sur `client-User2`.

* Les fichiers de règles spécifiques dans `/etc/sudoers.d/` sont explicitement supprimés s'ils existent.

* **Résultat :** Lors d'une connexion SSH (ex: `vagrant ssh client-User1` suivi de `sudo su - user1`), l'utilisateur ne possède aucun privilège `sudo`. Toute tentative de mise à jour (ex: `sudo apt update`) est rejetée.

### 2.2 Dossier partagé NFS

**Méthode :** Un serveur NFS expose un dossier à destination d'un sous-réseau privé.

* **Côté Serveur (`serveur-central`) :** Installation de `nfs-kernel-server` et création du dossier `/var/nfs/partage_commun`. Les permissions sont réglées sur `0777` avec le propriétaire `nobody:nogroup` pour simplifier les tests. Le dossier est exporté vers le réseau `192.168.56.0/24`.

* **Côté Clients :** Installation de `nfs-common`, création du dossier `/mnt/serveur_central`, et montage direct de l'adresse `192.168.56.10:/var/nfs/partage_commun`.

### 2.3 Règles de Pare-feu (UFW)

**Méthode :** Définition d'une politique restrictive de base combinée à des exceptions granulaires.

* **Politique globale :** Tous les paquets entrants sont refusés et tous les sortants sont autorisés.

* **Accès de base :** Le port SSH (22 / TCP) est autorisé sur toutes les machines pour ne pas se bloquer dehors.

* **Serveur central :** Une règle spécifique ouvre le port 2049 (NFS) au protocole TCP, mais uniquement pour les requêtes provenant du sous-réseau privé `192.168.56.0/24`.

## 3. Dépannage et limitations

* **Blocage réseau VMware :** Si Vagrant reste bloqué sur "Waiting for the VM to receive an address...", il est conseillé de choisir un OS adapté pour simplifier la configuration IP, ou d'autoriser l'hyperviseur à accéder au réseau local. Il est également possible d'utiliser `vmnet-cli` pour arrêter, configurer et redémarrer le réseau VMware.

* **Comptes par défaut :** En cas de besoin de maintenance, le login et le mot de passe par défaut de la box sont tous deux définis sur `vagrant`.

* **Architecture CPU :** Le Vagrantfile actuel impose l'architecture `arm64`. Pour d'autres systèmes, des alternatives comme `debian/bookworm64`, `ubuntu/focal64`, ou TinyCoreLinux peuvent être utilisées.

## 4. Comment lancer

1. **Installer Vagrant :**

   * Sur macOS : `brew tap hashicorp/tap` puis `brew install hashicorp/tap/hashicorp-vagrant`.

   * Sur Linux (Ubuntu/Debian) : Télécharger la clé GPG d'HashiCorp, l'ajouter aux sources APT, puis lancer `sudo apt update && sudo apt install vagrant`.

2. **Démarrer l'infrastructure :** Placez-vous dans le dossier contenant le fichier `Vagrantfile` et exécutez la commande `vagrant up`. Les machines démarreront une par une (avec l'interface graphique VMware visible) et appliqueront les trois scripts Ansible définis.

3. **Vérifier les droits :** Connectez-vous avec `vagrant ssh client-User1`, basculez sur l'utilisateur (`sudo su - user1`), et lancez `whoami` ou `sudo -l` pour confirmer l'absence de privilèges.

---

## 5. Notes pour moi-même (Mémo technique)

*This is my guidance for making a simple manageable linux machine Parc.*

### Documentation Officielle
- [Vagrant Documentation (Vagrantfile)](https://developer.hashicorp.com/vagrant/docs/vagrantfile)
- [Vagrant & Ansible Provisioning](https://developer.hashicorp.com/vagrant/docs/provisioning/ansible_intro)

### Installation de Vagrant

**macOS :**
```bash
brew tap hashicorp/tap
brew install hashicorp/tap/hashicorp-vagrant
```

**Linux Ubuntu/Debian :**
```bash
wget -O - https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(grep -oP '(?<=UBUNTU_CODENAME=).*' /etc/os-release || lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install vagrant
```

### Options d'OS (Box Vagrant)

- **Debian :** `config.vm.box = "debian/bookworm64"`
- **TinyCoreLinux :** [Download Tiny OS](http://tinycorelinux.net/downloads.html) (Not really used)
- **Note pour architecture arm64 :** Utiliser `config.vm.box = "ubuntu/focal64"` ou similaire.

### Troubleshooting (Dépannage)

- Allow the hypervisor to access my local network.
- `v.gui = true` permet de forcer les permissions (prompts) dans les options VMware.
- **Identifiants par défaut :** login et password = `vagrant`
- Choisir la [bonne box OS](https://portal.cloud.hashicorp.com/vagrant/discover/bento/ubuntu-26.04) pour simplifier la configuration des adresses IP et éviter l'erreur :
  > `==> serveur-central: Starting the VMware VM...`  
  > `==> serveur-central: Waiting for the VM to receive an address...`
- **Sinon, redémarrer le réseau VMware Fusion (macOS) :**
  ```bash
  sudo /Applications/VMware\ Fusion.app/Contents/Library/vmnet-cli --stop
  sudo /Applications/VMware\ Fusion.app/Contents/Library/vmnet-cli --configure
  sudo /Applications/VMware\ Fusion.app/Contents/Library/vmnet-cli --start 
  ```

### Note — Utilisateurs et permissions avec Ansible

`become: true` donne les privilèges nécessaires à **Ansible pendant l'exécution du playbook**. Cela ne donne pas automatiquement les privilèges root à l'utilisateur créé.

**Schéma de fonctionnement :**
```text
Vagrant
   ↓
VM client-User1
   ↓
Ansible (become: true)
   ↓
création de user1
   ↓
user1 = utilisateur normal
   ↓
pas de privilèges sudo
```

**Exemple : utilisateur sans privilèges root (Bloc exécuté)**
```bash
vagrant@client-User1:~$ whoami
vagrant
vagrant@client-User1:~$ sudo su - user1
user1@client-User1:~$ whoami
user1
user1@client-User1:~$ sudo apt update
sudo: I'm sorry user1. I'm afraid I can't do that
user1@client-User1:~$ exit
```

**Commandes de vérification utiles :**
```bash
whoami
id
groups
sudo -l
cat /etc/sudoers
ls /etc/sudoers.d/
```

### Next Step
Un serveur SQL.