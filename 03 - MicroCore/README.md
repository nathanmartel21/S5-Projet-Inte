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
   <br>

   > Source : [Lien YouTube](https://www.youtube.com/watch?v=_oNzsIcvfbM)

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

   > Source : [Lien YouTube](https://www.eldin.net/meteo/tinycore.php)

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

   > Source : [Lien YouTube](https://www.youtube.com/watch?v=9fVkWLM0sCg)

## 3.4 Intégration des commandes Linux
   - L'intégration d'iPv6 dans Microcore se fait par la bibliothèque iptables qui contient elle-même le package ipv6-netfilter. Dans un tty, saisir :
   ```bash
   tce-load -wi tcpdump.tcz
   tce-load -wi iproute2.tcz
   tce-load -wi iptables
   sudo modprobe ipv6
   ```
   - Ajout de la configuration pour qu'elle persiste au démarrage :
   ```bash
   sudo –s
   echo "sudo modprobe ipv6" >> /opt/bootlocal.sh
   ```
   - Enfin, ne pas oublier de sauvegarder les modifications :
   ```bash
   filetool.sh -b
   ```

   > Source : [Lien externe](https://brezular.com/2011/01/26/linux-core-as-network-host/)

## 3.4bis Adressage des machines Microcore :
   - Chaque machine Microcore possède une @IP différente et fixe. Cf tableau adressage.
     Pour rendre l'adressage persistent :
   ```bash
   touch /opt/eth0.sh
   ```
   - Inclure les lignes suivantes dans le script pour la config de l'interface de la machine :
   ```bash
   #!/bin/sh
   
   pkill udhcpc # Arrêter le processus UDHCPC
sudo ip addr add [@IP]/[MASK] dev [INTERFACE] #Ajoute une @IP à la machine
   sudo ip route add default via [GATEWAY] dev [INTERFACE] #Ajoute une default gateway
   sudo nameserver [DNSSERVEUR] #Ajoute le serveur DNS pour les résolutions
   ```
   - Ajout de la configuration pour qu'elle persiste au démarrage :
   ```bash
   sudo –s
   echo "sudo /opt/eth0.sh" >> /opt/bootlocal.sh
   ```
   - Enfin, ne pas oublier de sauvegarder les modifications :
   ```bash
   filetool.sh -b
   ```   
   A chaque redémarrage, Microcore excutera le script et recevra l'adresse statique indiquée dans le script.

   > Source : [Lien externe](https://forum.tinycorelinux.net/index.php/topic,13781.0.html)




