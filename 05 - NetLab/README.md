# Mise en place du réseau NetLab

## 5.1 Configuration des VMs :

- Chaque VM (Microcore, VyOS et Alpine) est configurée avec une ou plusieurs interfaces réseau, chacune étant connectée à un adaptateur réseau en mode **host-only**.

## 5.2 Configuration des équipements réseaux :

### Microcore T[n] :

  - Configuration automatique avec /opt/eth0.sh :
    
    ```
    #!/bin/sh

    pkill udhcpc
    sudo ip addr add W.X.Y.Z/NN dev eth0
    sudo ip link set up dev eth0
    sudo ip route add default via W.X.Y.R/NN dev eth0
    ```
    
  - Contenu du fichier bootsync :
    
    ```
    /usr/bin/sethostname T[n]
    /opt/bootlocal.sh &
    ```
    
  - Contenu du fichier bootlocal :
    
    ```
    #!/bin/sh
    loadkmap < /usr/share/kmap/azerty/fr-latin1.kmap
    sudo modprobe ipv6
    /opt/eth0.sh
    ```
    
### VyOS R0 :

  - Commandes :
    
    ```
    set interfaces ethernet eth0 address 10.8.5.2/24
    set interfaces ethernet eth0 address 2001:0:8:5::2/64
    set interfaces ethernet eth0 description towardR1

    set interfaces ethernet eth1 address 10.8.1.1/24
    set interfaces ethernet eth1 address 2001:0:8:1::1/64
    set interfaces ethernet eth1 description towardT1

    set interfaces ethernet eth2 address 10.8.2.1/24
    set interfaces ethernet eth2 address 2001:0:8:2::1/64
    set interfaces ethernet eth2 description towardT2

    set protocols ospf area 0 network 10.8.1.0/24
    set protocols ospf area 0 network 10.8.2.0/24
    set protocols ospf area 0 network 10.8.5.0/24
    set protocols ospf parameters router-id 1.1.1.1

    set protocols ospfv3 area 0
    set protocols ospfv3 interface eth0 area 0
    set protocols ospfv3 interface eth1 area 0
    set protocols ospfv3 interface eth2 area 0
    set protocols ospfv3 parameters router-id 1.1.1.1
    ```
    
### VyOS R1 :

  - Commandes :
    
    ```
    set interfaces ethernet eth0 address 172.16.0.8/24
    set interfaces ethernet eth0 address 2002:16:0:0::8/64
    set interfaces ethernet eth0 description towardNothing

    set interfaces ethernet eth1 address 192.168.8.5/30
    set interfaces ethernet eth1 address 3FFE:0:8:4::1/64
    set interfaces ethernet eth1 description towardR2

    set interfaces ethernet eth2 address 10.8.5.1/24
    set interfaces ethernet eth2 address 2001:0:8:5::1/64
    set interfaces ethernet eth2 description towardR0

    set protocols ospf area 0 network 172.16.0.0/24
    set protocols ospf area 0 network 192.168.8.4/30
    set protocols ospf area 0 network 10.8.5.0/24
    set protocols ospf parameters router-id 2.2.2.2

    set protocols ospfv3 area 0
    set protocols ospfv3 interface eth0 area 0
    set protocols ospfv3 interface eth1 area 0
    set protocols ospfv3 interface eth2 area 0
    set protocols ospfv3 parameters router-id 2.2.2.2
    ```
    
### VyOS R2 :

  - Commandes :
    
    ```
    set interfaces ethernet eth0 address 10.8.3.1/24
    set interfaces ethernet eth0 address 2001:0:8:3::1/64
    set interfaces ethernet eth0 description towardT3T4

    set interfaces ethernet eth1 address 192.168.8.6/30
    set interfaces ethernet eth1 address 3FFE:0:8:4::2/64
    set interfaces ethernet eth1 description towardR1

    set protocols ospf area 0 network 10.8.3.0/24
    set protocols ospf area 0 network 192.168.8.4/30
    set protocols ospf parameters router-id 3.3.3.3

    set protocols ospfv3 area 0
    set protocols ospfv3 interface eth0 area 0
    set protocols ospfv3 interface eth1 area 0
    set protocols ospfv3 parameters router-id 3.3.3.3
    ```
