# Frugal's installation + Microcore on ttySO

Cher lecteur, ou lectrice, aujourd’hui tu vas apprendre des trucs cool sur Microcore (oui je sais, dit comme ça c’est pas trop cool mais t’inquiète tu vas voir).

## Au menu :

1. **Rendre Frugal ta MicroCore avec le tty et donc sans la GUI** (en ayant installé l’image Core de 17MB)
2. **Redirection de toute la sortie de démarrage de Tiny Core vers le port série COM1**

---

### 1. Pour installer Tinycore sur le disque local en Frugal :

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
  - Choisisser l’option `1` pour utiliser le disque en entier
  - Sélectionner le **disque local sda** (option `2`)
  - **Installer un chargeur de démarrage local** : taper `y`
  - **Installer les extensions à partir de ce répertoire TCD/CDE** : taper `n`
  - Sélectionner l'**option de formatage pour sda** : `3` (ext 4)
  - Sur l’écran suivant, ajouter la résolution d’affichage et le répertoire de restauration TCE en saisissant :
  ```bash
  vga=788 tce=hda1
  ```
  - Dernière chance de sortir avant de détruire toutes les données sur sda : taper `y`
- À la fin, si l'installation est bien faite la sortie indique que l’installation est terminée [et pas de message d’erreur].
- Dernière étape :
  ```bash
  sudo reboot
  ```

### 2. Redirection stdout vers ttyS0 :

Tinycore prend en charge tous les codes de démarrage supplémentaires. Dans cette longue liste, il y a l’option `console=`. Dans mon cas, c'était :
  ```bash
  console=ttyS0,9600n8
  ```
- Dans Tinycore, le fichier `extlinux.conf` configure les options de démarrage du système (passées au noyau Linux au démarrage).
- Normalement, après l’installation de TinyCore en Frugal (cf. partie 1), le fichier `extlinux.conf` se trouve dans le disque SCSI avec le chemin suivant :
  ```bash
  /mnt/sda1/tce/boot/extlinux/extlinux.conf
  ```
- Modifier ce fichier de conf en ajoutant à la fin de la ligne APPEND l’option suivante :
  ```bash
  console=ttyS0,9600n8
  ```
- Si la VM se redémarre maintenant, seule la sortie de la phase de démarrage sera affichée sur la console série. Pour que Tinycore prenne en charge la carte vidéo comme périphérique de sortie (cf fichier /etc/inittab), il faut également éditer un autre fichier.
- Éditer le fichier /opt/bootsync.sh (script exécuté automatiquement au démarrage du système) pour y ajouter la ligne suivante entre les deux lignes existantes :
  ```bash
  /sbin/getty 9600 ttyS0
  ```
  Cette commande active le port série.
- Enregistrer toutes les modifications avec :
  ```bash
  sudo filetool.sh –b
  ```
- Un dernier redémarrage avec :
  ```bash
  sudo reboot
  ```
Et le tour est joué (ne pas oublier de passer en telnet).

> Tips : `sudo reboot` / `sudo poweroff` est remplacé par :
  ```bash
  exit
  ```
> Source : [Documentation du noyau Linux sur la console série](https://www.kernel.org/doc/html/v4.15/admin-guide/serial-console.html)

### 3. Auteur :

@uthor : Nathan Martel

**If you have access to this GitHub repository in any way, you are strictly prohibited from using, reproducing, or distributing any of the resources, code, or research contained within this project. Access to this repository does not grant permission to utilize or share any part of its content without explicit authorization.**
