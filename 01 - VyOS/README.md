# Installation et configuration de VyOS

## 1.1 Création de la VM VyOS

1. **Config de la machine virtuelle** :
   - **Mémoire** : 2 GB
   - **Proc** : 4
   - **Hard drive** : SCSI (4 Go)

3. **Procédure d'installation** :
   - Lancer la VM avec l'ISO de VyOS.
   - Lancer la commande suivante sur le disque `sda` :
     ```bash
     install image
     ```
     - Répondre `y` pour utiliser l'espace libre.
     - Choisir toujours les options par défaut (y compris pour l'option "1").
   - Eteindre la machine virtuelle en utilisant la commande :
     ```bash
     poweroff
     ```
   - Dans les paramètres de la VM, CD/DVD (SATA), décocher `Connected` et `Connect at power on` et cocher `Use Physical drive`.
   - Redémarrer la VM.

## 1.2 Conf automatique du clavier azerty

1. **Utiliser la commande pour la conf du clavier en azerty :**
   ```bash
   sudo dpkg-reconfigure console-data
   ```
   - Appuyer sur `[Entrer]`
   - Sélectionner `keymap from full list` et `pc azerty French latin9 standard`.

   - Dans le tty, exécuter les commandes suivantes :
     ```bash
     configure
     set system option keyboard-layout fr
     commit
     save
     ```    

## 1.3 Homogénéisation automatique des noms des interfaces

1. **Homogénéisation des noms :**

    - Créer/Ouvrer le fichier de règles `udev` :
   ```bash
   sudo vi /etc/udev/rules.d/70-persistent-net.rules
   ```

    - Ajouter la règle suivante en remplaçant `xx:xx:xx:xx:xx:xx` par l'adresse MAC de la carte réseau :
   ```bash
   SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="xx:xx:xx:xx:xx:xx", NAME="eth0"
   ```

    - Recharger les daemons udev et redémarrer la VM :
    ```bash
    sudo udevadm control --reload-rules && sudo udevadm trigger
    sudo reboot
    ```

2. **Adressage des interfaces réseaux :**

    - Config des interfaces réseaux :
      Dans le tty :

    ```bash
    configure
    set interfaces ethernet eth0 address dhcp
    commit
    save
    exit

    ```
