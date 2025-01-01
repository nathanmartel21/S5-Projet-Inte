# Installation et configuration de Alpine Linux

## 2.1 Création de la VM Alpine Linux

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

1. Installer OpenBox :

```bash
setup-xorg-base
apk add openbox xterm font-terminus
rc-update add acpid
mkdir ~/.config
cp -r /etc/xdg/openbox ~/.config
```

2. Configurer OpenBox pour le lancement :

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

1. Installer DWM :

```bash
setup-xorg-base
apk add dwm dmenu st
```

2. Configurer OpenBox pour le lancement :

- En utilisateur nathan :

```bash
cd ~
touch .profile
+ sudo startx
touch .xinitrc
+ exec dwm
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

3. Configurer Grub :
















