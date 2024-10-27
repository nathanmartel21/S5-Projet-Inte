# Installation et configuration de MicroCore

## 3.1 Création de la VM TinyCore

1. **Config de la machine virtuelle** :
   - **Mémoire** : 2 GB
   - **Proc** : 4
   - **Hard drive** : SATA (4 GB)

2. **Installation de MicroCore** :

   - Démarrer la VM avec l'ISO de MicroCore.
   - Faire [clic droit] sur le bureau et cliquer sur **Applications** > **tc-install**.
   - Cocher **Frugal** et **Whole Disk** en choisissant le disque `sda` (select disk for core).
   - Laisser les paramètres par défaut pour les autres pages et cliquer à la fin sur **Proceed**.
   - À la fin de l'installation, si tout est ok, on a le message suivant : `Setting up core image on /mnt/sda1`.
   - Redémarrer la VM

## 3.2 Conf automatique du clavier azerty
   - Installation du package de disposition de clavier :
   ```bash
   sudo tce-load -wi kmaps.tcz 
   ```
   - Chargement du clavier en azerty :
   ```bash
   sudo loadkmap < /usr/share/kmap/azerty/fr-latin1.kmap
   ```
   - Ajout de la configuration de clavier pour qu'elle persiste au démarrage :
   ```bash
   sudo –s
   echo "loadkmap < /usr/share/kmap/azerty/fr-latin1.kmap" >> /opt/bootlocal.sh
   ```
   - Sauvegarde des modifications :
   ```bash
   filetool.sh -b
   ```

## 3.3 Conf des dossiers utilisateurs et logiciels afin qu'ils soient persistant
   - Lancer la VM et dans RunProgram, lancer editor en super utilisateur
   - Ouvrir le fichier `/mnt/sda1/tce/boot/extlinux/extlinux.conf`
   - Après `quiet`, ajouter :
   ```bash
   opt=sda1 home=sda1
   ```
   - Sauvegarder le fichier, quitter
   - Dans le ControlPanel > Backup/Restore > Included for Backup, supprimer les items `opt` et `home`
   - Redémarrer la VM avec l'option None

## 3.4 Intégration des commandes Linux
   Dans un tty, saisir :
   ```bash
   tce-load -wi tcpdump.tcz
   tce-load -wi iproute2.tcz
   # IPv6 ?? https://forum.tinycorelinux.net/index.php/topic,23340.0.html
   ```  
