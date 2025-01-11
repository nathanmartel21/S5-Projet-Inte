# Installation et configuration de MicroCore

## 3.1 Création de la VM TinyCore :

1. **Config de la machine virtuelle** :
   - **Mémoire** : 1024 MB
   - **Proc** : 2
   - **Hard drive** : SATA (4 GB)

2. **Installation de MicroCore en Frugal** :

   - Démarrer la VM avec l'ISO de MicroCore.
   - Dans le tty de Tinycore : Télécharger le module « tc -install » avec :
     ```bash
     tce-load –wi tc-install
     ```
   - Commencer l’installation avec :
     ```bash
      sudo tc-install.sh
     ```
   - Plusieurs options d’installation vont t’être demandées :
     - Sélectionner le **CD Rom comme source d’installation** : taper la lettre `c`
     - Sélectionner **Frugal** pour installer Tinycore en Frugal sur le disque local : taper `f`
     - Choisir l’option `1` pour utiliser le disque en entier
     - Sélectionner le **disque local sda** (option `2`)
     - **Installer un chargeur de démarrage local** : taper `y`
     - **Installer les extensions à partir de ce répertoire TCD/CDE** : taper `n`
     - Sélectionner l'**option de formatage pour sda** : `3` (ext 4)
     - Sur l’écran suivant, ajouter la résolution d’affichage et le répertoire de restauration TCE en saisissant :
     ```bash
     vga=788 tce=hda1 #je pourrai mettre aussi ici azerty ou même les persitents folders
     ```
     - Dernière chance de sortir avant de détruire toutes les données sur sda : taper `y`
   - À la fin, si l'installation est bien faite la sortie indique que l’installation est terminée [et pas de message d’erreur].
   - Dernière étape :
     ```bash
     sudo reboot
     ```

## 3.2 Conf automatique du clavier azerty :
   - Installation du package de disposition de clavier :
   ```bash
   tce-load -wi kmaps.tcz 
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

   > Source : [Lien externe](https://www.eldin.net/meteo/tinycore.php)

## 3.3 Conf des dossiers utilisateurs et logiciels afin qu'ils soient persistant :
   - Lancer la VM et dans RunProgram, lancer editor en super utilisateur
   - Ouvrir le fichier `/mnt/sda1/tce/boot/extlinux/extlinux.conf`
   - Après `quiet`, ajouter :
   ```bash
   opt=sda1 home=sda1
   ```
   - Sauvegarder le fichier, quitter

   > Source : [Lien YouTube](https://www.youtube.com/watch?v=9fVkWLM0sCg)

## 3.4 Intégration des commandes Linux :
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
   touch /opt/eth0.sh && sudo chmod 755 /opt/eth0.sh
   ```
   - Inclure les lignes suivantes dans le script pour la config de l'interface de la machine :
   ```bash
   #!/bin/sh
   
   pkill udhcpc # Arrêter le processus UDHCPC
sudo ip addr add [@IP]/[MASK] dev [INTERFACE] #Ajoute une @IP à la machine
   sudo ip route add default via [GATEWAY] dev [INTERFACE] #Ajoute une default gateway
   sudo echo "nameserver [DNSSERVEUR]" > /etc/resolv.conf #Ajoute le serveur DNS pour les résolutions
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
