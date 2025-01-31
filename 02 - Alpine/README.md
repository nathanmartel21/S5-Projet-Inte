# Installation et configuration de Alpine Linux

## 2.1 Création de la VM Alpine Linux :

1. **Config de la machine virtuelle** :
   - **Mémoire** : 1024 MB
   - **SCSI Controller** : LSI Logic
   - **Proc** : 2
   - **Disk type** : SCSI (8 GB)
2. **Installer Grub** :
     ```bash
     setup-alpine
     ...
     reboot
     apk del syslinux
     apk add grub grub-bios
     grub-install /dev/sda
     grub-mkconfig -o /boot/grub/grub.cfg
     reboot
     ```

## 2.2 Configuration Grub pour un dual interface utilisateur

### 2.2.1 OpenBox :

**1. Installer OpenBox** :

```bash
setup-xorg-base
apk add openbox xterm font-terminus
rc-update add acpid
mkdir ~/.config
cp -r /etc/xdg/openbox ~/.config
```

**2. Configurer OpenBox pour le lancement / GRUB** :

- En utilisateur nathan :

```bash
cd ~
touch .profile
+ sudo startx
touch .xinitrc
+ exec openbox-session
apk add sudo
```

- En utilisateur root :

```bash
useradd nathan wheel
visudo
+ nathan ALL=(ALL:ALL) ALL
+ nathan ALL=(ALL) NOPASSWD: /usr/bin/startx
apk update
reboot
```

### 2.2.2 DWM :

**1. Installer DWM** :

```bash
setup-xorg-base
apk add dwm dmenu st
```

**2. Configurer DWM pour le lancement / GRUB** :

- En utilisateur nathan :

```bash
cd ~
touch .profile
+ sudo startx
touch .xinitrc
+ exec dwm
apk add sudo

OU

doas vi /etc/doas.d/doas.conf
+ permit nopass nathan as root cmd startx
+ permit nopass root as root cmd startx
```

- En utilisateur root :

```bash
useradd nathan wheel
visudo
+ nathan ALL=(ALL:ALL) ALL
+ nathan ALL=(ALL) NOPASSWD: /usr/bin/startx
apk update
reboot
```

**2.3 Intégration des outils** :

```bash
apk add openssh && rc-update add sshd && rc-service sshd start && rc-update add sshd boot
apk add filezilla
apk add tcpdump
apk add wireshark
apk add putty
```

Pour le navigateur WEB :

```
apk update
echo "http://dl-cdn.alpinelinux.org/alpine/edge/testing" >> /etc/apk/repositories
sudo apk add netsurf
startx
```

**2.4 Installation des services et modules VMware tools** :

- Ancien :

```bash
apk add open-vm-tools open-vm-tools-guestinfo open-vm-tools-deploypkg
rc-service open-vm-tools start
rc-update add open-vm-tools boot
```

- Nouveau : Il faut savoir que DWM inclu nativement le curseur de la souris ce qui permet de ne pas ce soucier du tools mouse de vmware. Il suffit alors, pour la résolution d'écran de faire :

```
apk add open-vm-tools-gtk
rc-update add open-vm-tools
```

**WARNING sur le deuxième sudo startx, risque de ne pas fonctionner, ne pas faire ce qui suit :**

Jouer ensuite sur :

```
apk info | grep vm

apk del open-vm-tools-openrc
apk del open-vm-tools-hgfs
```

--> fonctionne pour le premier sudo startx mais pour le second non et les suivants aussi.

