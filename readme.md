---
author: Valentin
date: 14/02/2024
---

# Flamenco

<!-- TOC -->
- [Installation Samba](#installation-samba)
    - [Création groupe utilisateur](#création-groupe-utilisateur)
- [Installation alternative poste openmediavault](#installation-alternative-poste-openmediavault)
- [Installation Flamenco](#installation-flamenco)
    - [Mise en place du poste manager](#mise-en-place-du-poste-manager)
- [Configuration addon flamenco](#configuration-addon-flamenco)
<!-- /TOC -->


## Installation Samba


Mise à jour des packages

`sudo apt update && sudo apt upgrade -y`

Installation Samba sur Debian 

`sudo apt install samba -y`

Installation client Samba Debian

`sudo apt install samba smbclient cifs-utils`


Ouvrir la feuille de configuration samba 

`sudo nano /etc/samba/smb.conf`

On vient changer la ligne 36 

`;interfaces = 192.168.0.134/24 eth0`

vers 

`interfaces = ton-ip eth0`

Ensuite on se dirige tout au bout du fichier conf et on va venir créer un premier dossier de partage privé

```
[private]
path = /private
writable = yes
guest ok = no 
valid users = @smbshare
force create mode = 770
force directory mode = 770 
inherit permission = yes 
```
En cas de dossier créer sur un autre disque sur le système pensez à donner toutes les permissions nécessaires et à rajouter la ligne suivante au paragraphe ci-dessus 

`force user = your-user-name`

Dans une autre fenêtre de terminal on vient créer le dossier en question 

`sudo mkdir /private`

### Création groupe utilisateur 

Donc on va venir créer le groupe avec la commande suivante

`sudo groupadd smbshare`

Puis on ajoute les permissions nécessaires 

`sudo chgrp -R smbshare /private/`

`sudo chmod 2770 /private/`

On vient ensuite créer un no login user pour l'accès local 

`sudo useradd -M -s /sbin/nologin sambauser`

On ajoute cet utilisateur au groupe créer précédemment 

`sudo usermod -aG smbshare sambauser`

On vient définir un mot de passe 

`sudo smbpasswd -a sambauser`

Et pour finir on active l'utilisateur 

`sudo smbpasswd -e sambauser`

Afin de vérifier la configuration 

`sudo testparm`

On redémarre samba 

`sudo systemctl restart nmbd`

Si présence de firewall pensez à la commande 

`sudo ufw allow from ton-ip to any app Samba`

Ensuite pour se connecter depuis un ordinateur client pensez à installer les clients Samba 

`sudo apt install samba smbclient cifs-utils`

Ensuite pour se connecter depuis linux, il suffit d'aller dans Files > Other locations en bas à droite on à connect to server 

On utilise la formule ci-dessous 

`smb://servername/Share_name`

## Installation alternative poste openmediavault

Pour une raison où pour une autre vous pouvez choisir également de sacrifier un poste qui fera office de NAS en lui installant dessus openmediavault, qui est une interface basé sur debian optimisé pour la création d'un NAS

La première étape consiste à télécharger l'image ISO contenant la distribution Linux OMV. Pour cela, il faut se rendre sur le site officiel :

https://www.openmediavault.org/download.html

Télécharger la dernière version **Stable**

On va venir récupérer un ISO avec lequel on peut créer une clé USB Bootable à l'aide d'un logiciel tiers. 

-Pour Linux : https://unetbootin.github.io/linux_download.html

-Pour Windows : https://rufus.ie/fr/

-Pour MacOS : https://etcher.balena.io/

Donc on va se munir d'une clé USB d'au moins 8go qu'on va venir monter sur notre ordinateur afin de flasher l'ISO. 

Une fois la clé prête on peut la retirer et l'insérer dans l'ordinateur ou l'on souhaite installer openmediavault.

**Pensez à régler dans le bios la boot sequence avec le lecteur USB en premier**

On se retrouve ensuite sur le processus d'installation, qu'on suit tout le long.

**Attention au mot de passe superutilisateur (root) vous devez vous en rappeler pour vous connecter à la console**

A la fin de l'installatin l'ordinateur va redémarrer et s'ouvrir sur la console de openmediavault. On s'y connecte en utilisant le login *root* et le mot de passe défini préalablement 

Afin de récupérer l'adresse ip de l'ordinateur on peut utiliser cette commande sur la console : 

`ip a`

On récupère l'adresse qui commence par 192. Et on vient saisir l'adresse ip complète dans le navigateur de votre choix sur un poste connecté au même réseau. **Attention à utiliser une fenêtre de navigation privée pour éviter les problèmes de caches.**

On arrive ensuite sur la page d'acceuil de l'interface, le login de base est : *admin* et le mot de passe de base : *openmediavault*  

Pour le setup complet d'un dossier partagé je joins le lien d'une doc compléte et bien expliquée.

Documentation complète : 

https://sparwan.com/articles/post/tutoriel-dinstallation-open-media-vault-pour-serveur-nas-avec-disque-dur.html

Openmediavault fonctionne très bien en plus de disposer d'une interface graphique modulable et facilement compréhensible. Tout en permettant d'être modifié/géré à distance par n'importe quel poste connecté au réseau.

## Installation Flamenco 

Schéma de fonctionnement Flamenco

<a href="https://ibb.co/2FYHxS6"><img src="https://i.ibb.co/FhqC2XW/flamencoorga.png" alt="flamencoorga" border="0"></a>

Pour commencer, on vient télécharger Blender, en sélectionnant la version correspondante à son système d'exploitation https://blender.org

Puis on vient télécharger Flamenco, en sélectionnant la version correspondante à son système d'exploitation https://flamenco.blender.org/download/

On vient extraire le dossier dans le chemin de son choix (celui devra être le même pour tout les postes worker !)

### Mise en place du poste manager 

Lancer flamenco manager, vous allez arriver sur une page qui va s'ouvrir dans votre navigateur 

De là vous devez indiquer le chemin du dossier partagé **attention le chemin doit être le même pour tout les postes**

Ensuite le chemin de Blender 
**attention le chemin doit être le même pour tout les postes**

La configuration de base de flamenco est désormais terminée et vous devriez normalement arriver sur l'interface de flamenco.

Normalement dans le dossier flamenco un fichier flamencomanager.yaml a été créer à la suite de la configuration, si vous utilisez samba ou openmediavault et non un NAS il va falloir l'ouvrir et désactiver Shaman. Pour cela on change la ligne 

`Shaman : true` 

en

`Shaman : false`

Dans le même document, si vous comptez utiliser flamenco avec plusieurs plateformes, il est important de rajouter un paragraphe dans les variables (en changeant les chemins pour les votres)

```variables:
  my_storage:
    is_twoway: true
    values:
    - platform: linux
      value: /media/shared/flamenco
    - platform: windows
      value: F:\flamenco
    - platform: darwin
      value: /Volumes/shared/flamenco
```

En cas de besoin, plus de doc ici : https://flamenco.blender.org/usage/variables/multi-platform/

## Configuration addon flamenco

Maintenant vous allez pouvoir installer l'addon flamenco sur votre poste personnel, peu importe votre système d'exploitation le procédé est le même. 

Vous venez installer l'addon grâce au zip télécharger (installation classique d'addon blender)

<a href="https://ibb.co/q7vrRT2"><img src="https://i.ibb.co/7jq1Gs0/Capture-d-e-cran-2024-02-14-a-16-14-48.png"
alt="Capture-d-e-cran-2024-02-14-a-16-14-48" border="0"></a>

Ensuite il va falloir le configurer. Donc on vient rentrer l'adresse ip du manager et on vient refresh. Si tout se passe bien on devrait voir apparaître la phrase *Flamenco version 3.4 found*

Cela suffit pour la configuration on peut sauvegarder les préférences et fermer le menu. 

Maintenant on peut préparer sa scène, on vient bien enregistrer son fichier Blender dans le dossier partagé, dans le dossier **Jobs**.

On procéde a tout les réglages de rendu, on oublie pas de sauvegarder avant de lancer le render. Une fois tout enregistrer on peut se rendre dans l'onglet Flamenco 3.

<a href="https://ibb.co/k4DCLMR"><img src="https://i.ibb.co/1rXWCbD/Capture-d-e-cran-2024-02-14-a-16-18-14.png" alt="Capture-d-e-cran-2024-02-14-a-16-18-14" border="0"></a>

On peut venir ici définir ici le nom de notre job et sa priorité. 

**Important** pour le render output root il faut penser à le placer dans le dossier partagé accessible par tout les postes (de base le render si non défini va se faire sur votre poste)

Et après on peut lancer le rendu en appuyant sur **Submit to Flamenco**

