# Développement de la commande pour lister les interfaces (ifshow) :

## Sur Microcore / Tinycore :

```
tce-load -wi compiletc
```

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <arpa/inet.h>
#include <ifaddrs.h>
#include <net/if.h>
#include <netinet/in.h>

void print_interface_info(const char *ifname) {
    struct ifaddrs *ifaddr, *ifa;
    char addr[INET6_ADDRSTRLEN];
    char netmask[INET6_ADDRSTRLEN];

    if (getifaddrs(&ifaddr) == -1) {
        perror("getifaddrs");
        exit(EXIT_FAILURE);
    }

    for (ifa = ifaddr; ifa != NULL; ifa = ifa->ifa_next) {
        if (ifa->ifa_addr == NULL)
            continue;

        if (ifname != NULL && strcmp(ifa->ifa_name, ifname) != 0)
            continue;

        if (ifa->ifa_addr->sa_family == AF_INET) {
            // IPv4
            struct sockaddr_in *in = (struct sockaddr_in *)ifa->ifa_addr;
            struct sockaddr_in *mask = (struct sockaddr_in *)ifa->ifa_netmask;

            inet_ntop(AF_INET, &in->sin_addr, addr, sizeof(addr));
            inet_ntop(AF_INET, &mask->sin_addr, netmask, sizeof(netmask));

            int prefix_length = __builtin_popcount(ntohl(mask->sin_addr.s_addr));
            printf("Interface: %s (IPv4) -> Address: %s/%d\n", ifa->ifa_name, addr, prefix_length);
        } else if (ifa->ifa_addr->sa_family == AF_INET6) {
            // IPv6
            struct sockaddr_in6 *in6 = (struct sockaddr_in6 *)ifa->ifa_addr;
            struct sockaddr_in6 *mask6 = (struct sockaddr_in6 *)ifa->ifa_netmask;

            inet_ntop(AF_INET6, &in6->sin6_addr, addr, sizeof(addr));

            // Calculer le préfixe pour IPv6
            int prefix_length = 0;
            if (mask6) {
                unsigned char *mask_bytes = (unsigned char *)&mask6->sin6_addr;
                for (int i = 0; i < 16; i++) {
                    prefix_length += __builtin_popcount(mask_bytes[i]);
                }
            }
            printf("Interface: %s (IPv6) -> Address: %s/%d\n", ifa->ifa_name, addr, prefix_length);
        }
    }

    freeifaddrs(ifaddr);
}

void print_all_interfaces_with_prefixes() {
    struct ifaddrs *ifaddr, *ifa;
    char addr[INET6_ADDRSTRLEN];
    char netmask[INET6_ADDRSTRLEN];

    if (getifaddrs(&ifaddr) == -1) {
        perror("getifaddrs");
        exit(EXIT_FAILURE);
    }

    for (ifa = ifaddr; ifa != NULL; ifa = ifa->ifa_next) {
        if (ifa->ifa_addr == NULL)
            continue;

        printf("Interface: %s\n", ifa->ifa_name);

        if (ifa->ifa_addr->sa_family == AF_INET) {
            // IPv4
            struct sockaddr_in *in = (struct sockaddr_in *)ifa->ifa_addr;
            struct sockaddr_in *mask = (struct sockaddr_in *)ifa->ifa_netmask;

            inet_ntop(AF_INET, &in->sin_addr, addr, sizeof(addr));
            inet_ntop(AF_INET, &mask->sin_addr, netmask, sizeof(netmask));

            int prefix_length = __builtin_popcount(ntohl(mask->sin_addr.s_addr));
            printf("  IPv4 -> Address: %s/%d\n", addr, prefix_length);
        } else if (ifa->ifa_addr->sa_family == AF_INET6) {
            // IPv6
            struct sockaddr_in6 *in6 = (struct sockaddr_in6 *)ifa->ifa_addr;
            struct sockaddr_in6 *mask6 = (struct sockaddr_in6 *)ifa->ifa_netmask;

            inet_ntop(AF_INET6, &in6->sin6_addr, addr, sizeof(addr));

            // Calculer le préfixe pour IPv6
            int prefix_length = 0;
            if (mask6) {
                unsigned char *mask_bytes = (unsigned char *)&mask6->sin6_addr;
                for (int i = 0; i < 16; i++) {
                    prefix_length += __builtin_popcount(mask_bytes[i]);
                }
            }
            printf("  IPv6 -> Address: %s/%d\n", addr, prefix_length);
        }
    }

    freeifaddrs(ifaddr);
}

int main(int argc, char *argv[]) {
    if (argc == 3 && strcmp(argv[1], "-i") == 0) {
        print_interface_info(argv[2]);
    } else if (argc == 2 && strcmp(argv[1], "-a") == 0) {
        print_all_interfaces_with_prefixes();
    } else {
        fprintf(stderr, "Usage:\n");
        fprintf(stderr, "  %s -i <ifname> : Affiche les adresses IPv4 et IPv6 de l'interface spécifiée (avec préfixes)\n", argv[0]);
        fprintf(stderr, "  %s -a         : Affiche la liste des interfaces réseau et leurs préfixes d'adresses IPv4/IPv6\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    return 0;
}
```

Pour transférer les scripts, sur client :

```
nc -l -p 9999 > /opt/ifshow.c
```

Sur le serveur :

```
cat /opt/ifshow.c | nc [IP CLIENT] 9999
```

## Sur Alpine Linux :

```
apk add gcc
apk add musl-dev
```

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <arpa/inet.h>
#include <ifaddrs.h>
#include <net/if.h>
#include <netinet/in.h>

void print_interface_info(const char *ifname) {
    struct ifaddrs *ifaddr, *ifa;
    char addr[INET6_ADDRSTRLEN];
    char netmask[INET6_ADDRSTRLEN];

    if (getifaddrs(&ifaddr) == -1) {
        perror("getifaddrs");
        exit(EXIT_FAILURE);
    }

    for (ifa = ifaddr; ifa != NULL; ifa = ifa->ifa_next) {
        if (ifa->ifa_addr == NULL)
            continue;

        if (ifname != NULL && strcmp(ifa->ifa_name, ifname) != 0)
            continue;

        if (ifa->ifa_addr->sa_family == AF_INET) {
            // IPv4
            struct sockaddr_in *in = (struct sockaddr_in *)ifa->ifa_addr;
            struct sockaddr_in *mask = (struct sockaddr_in *)ifa->ifa_netmask;

            inet_ntop(AF_INET, &in->sin_addr, addr, sizeof(addr));
            inet_ntop(AF_INET, &mask->sin_addr, netmask, sizeof(netmask));

            int prefix_length = __builtin_popcount(ntohl(mask->sin_addr.s_addr));
            printf("Interface: %s (IPv4) -> Address: %s/%d\n", ifa->ifa_name, addr, prefix_length);
        } else if (ifa->ifa_addr->sa_family == AF_INET6) {
            // IPv6
            struct sockaddr_in6 *in6 = (struct sockaddr_in6 *)ifa->ifa_addr;
            struct sockaddr_in6 *mask6 = (struct sockaddr_in6 *)ifa->ifa_netmask;

            inet_ntop(AF_INET6, &in6->sin6_addr, addr, sizeof(addr));

            // Calculer le préfixe pour IPv6
            int prefix_length = 0;
            if (mask6) {
                unsigned char *mask_bytes = (unsigned char *)&mask6->sin6_addr;
                for (int i = 0; i < 16; i++) {
                    prefix_length += __builtin_popcount(mask_bytes[i]);
                }
            }
            printf("Interface: %s (IPv6) -> Address: %s/%d\n", ifa->ifa_name, addr, prefix_length);
        }
    }

    freeifaddrs(ifaddr);
}

void print_all_interfaces_with_prefixes() {
    struct ifaddrs *ifaddr, *ifa;
    char addr[INET6_ADDRSTRLEN];
    char netmask[INET6_ADDRSTRLEN];

    if (getifaddrs(&ifaddr) == -1) {
        perror("getifaddrs");
        exit(EXIT_FAILURE);
    }

    for (ifa = ifaddr; ifa != NULL; ifa = ifa->ifa_next) {
        if (ifa->ifa_addr == NULL)
            continue;

        printf("Interface: %s\n", ifa->ifa_name);

        if (ifa->ifa_addr->sa_family == AF_INET) {
            // IPv4
            struct sockaddr_in *in = (struct sockaddr_in *)ifa->ifa_addr;
            struct sockaddr_in *mask = (struct sockaddr_in *)ifa->ifa_netmask;

            inet_ntop(AF_INET, &in->sin_addr, addr, sizeof(addr));
            inet_ntop(AF_INET, &mask->sin_addr, netmask, sizeof(netmask));

            int prefix_length = __builtin_popcount(ntohl(mask->sin_addr.s_addr));
            printf("  IPv4 -> Address: %s/%d\n", addr, prefix_length);
        } else if (ifa->ifa_addr->sa_family == AF_INET6) {
            // IPv6
            struct sockaddr_in6 *in6 = (struct sockaddr_in6 *)ifa->ifa_addr;
            struct sockaddr_in6 *mask6 = (struct sockaddr_in6 *)ifa->ifa_netmask;

            inet_ntop(AF_INET6, &in6->sin6_addr, addr, sizeof(addr));

            // Calculer le préfixe pour IPv6
            int prefix_length = 0;
            if (mask6) {
                unsigned char *mask_bytes = (unsigned char *)&mask6->sin6_addr;
                for (int i = 0; i < 16; i++) {
                    prefix_length += __builtin_popcount(mask_bytes[i]);
                }
            }
            printf("  IPv6 -> Address: %s/%d\n", addr, prefix_length);
        }
    }

    freeifaddrs(ifaddr);
}

int main(int argc, char *argv[]) {
    if (argc == 3 && strcmp(argv[1], "-i") == 0) {
        print_interface_info(argv[2]);
    } else if (argc == 2 && strcmp(argv[1], "-a") == 0) {
        print_all_interfaces_with_prefixes();
    } else {
        fprintf(stderr, "Usage:\n");
        fprintf(stderr, "  %s -i <ifname> : Affiche les adresses IPv4 et IPv6 de l'interface spécifiée (avec préfixes)\n", argv[0]);
        fprintf(stderr, "  %s -a         : Affiche la liste des interfaces réseau et leurs préfixes d'adresses IPv4/IPv6\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    return 0;
}
```

```
apk del gcc
apk del musl-dev
```

## Sur VyOS :  

Compiler en amont sur la debian buster le script C. Mettre la debian sur le meme réseau que le routeur VyOS.
Sur la machine debian, lancer un serveur WEB :

```
sudo -s
cd ~
python -m http.server 8080
```

Sur le routeur VyOS, récupérer le fichier compilé :

```
sudo bash
cd /home/vyos
wget http://192.168.8.137:8080/ifshow
sudo chmod 777 ifshow
```

Faire ensuite un wget entre les routeurs.

## Sur machine hôte (Win64) sur WSL Ubuntu :  

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <arpa/inet.h>
#include <ifaddrs.h>
#include <net/if.h>
#include <netinet/in.h>

void print_interface_info(const char *ifname) {
    struct ifaddrs *ifaddr, *ifa;
    char addr[INET6_ADDRSTRLEN];
    char netmask[INET6_ADDRSTRLEN];

    if (getifaddrs(&ifaddr) == -1) {
        perror("getifaddrs");
        exit(EXIT_FAILURE);
    }

    for (ifa = ifaddr; ifa != NULL; ifa = ifa->ifa_next) {
        if (ifa->ifa_addr == NULL)
            continue;

        if (ifname != NULL && strcmp(ifa->ifa_name, ifname) != 0)
            continue;

        if (ifa->ifa_addr->sa_family == AF_INET) {
            // IPv4
            struct sockaddr_in *in = (struct sockaddr_in *)ifa->ifa_addr;
            struct sockaddr_in *mask = (struct sockaddr_in *)ifa->ifa_netmask;

            inet_ntop(AF_INET, &in->sin_addr, addr, sizeof(addr));
            inet_ntop(AF_INET, &mask->sin_addr, netmask, sizeof(netmask));

            int prefix_length = __builtin_popcount(ntohl(mask->sin_addr.s_addr));
            printf("Interface: %s (IPv4) -> Address: %s/%d\n", ifa->ifa_name, addr, prefix_length);
        } else if (ifa->ifa_addr->sa_family == AF_INET6) {
            // IPv6
            struct sockaddr_in6 *in6 = (struct sockaddr_in6 *)ifa->ifa_addr;
            struct sockaddr_in6 *mask6 = (struct sockaddr_in6 *)ifa->ifa_netmask;

            inet_ntop(AF_INET6, &in6->sin6_addr, addr, sizeof(addr));

            // Calculer le préfixe pour IPv6
            int prefix_length = 0;
            if (mask6) {
                unsigned char *mask_bytes = (unsigned char *)&mask6->sin6_addr;
                for (int i = 0; i < 16; i++) {
                    prefix_length += __builtin_popcount(mask_bytes[i]);
                }
            }
            printf("Interface: %s (IPv6) -> Address: %s/%d\n", ifa->ifa_name, addr, prefix_length);
        }
    }

    freeifaddrs(ifaddr);
}

void print_all_interfaces_with_prefixes() {
    struct ifaddrs *ifaddr, *ifa;
    char addr[INET6_ADDRSTRLEN];
    char netmask[INET6_ADDRSTRLEN];

    if (getifaddrs(&ifaddr) == -1) {
        perror("getifaddrs");
        exit(EXIT_FAILURE);
    }

    for (ifa = ifaddr; ifa != NULL; ifa = ifa->ifa_next) {
        if (ifa->ifa_addr == NULL)
            continue;

        printf("Interface: %s\n", ifa->ifa_name);

        if (ifa->ifa_addr->sa_family == AF_INET) {
            // IPv4
            struct sockaddr_in *in = (struct sockaddr_in *)ifa->ifa_addr;
            struct sockaddr_in *mask = (struct sockaddr_in *)ifa->ifa_netmask;

            inet_ntop(AF_INET, &in->sin_addr, addr, sizeof(addr));
            inet_ntop(AF_INET, &mask->sin_addr, netmask, sizeof(netmask));

            int prefix_length = __builtin_popcount(ntohl(mask->sin_addr.s_addr));
            printf("  IPv4 -> Address: %s/%d\n", addr, prefix_length);
        } else if (ifa->ifa_addr->sa_family == AF_INET6) {
            // IPv6
            struct sockaddr_in6 *in6 = (struct sockaddr_in6 *)ifa->ifa_addr;
            struct sockaddr_in6 *mask6 = (struct sockaddr_in6 *)ifa->ifa_netmask;

            inet_ntop(AF_INET6, &in6->sin6_addr, addr, sizeof(addr));

            // Calculer le préfixe pour IPv6
            int prefix_length = 0;
            if (mask6) {
                unsigned char *mask_bytes = (unsigned char *)&mask6->sin6_addr;
                for (int i = 0; i < 16; i++) {
                    prefix_length += __builtin_popcount(mask_bytes[i]);
                }
            }
            printf("  IPv6 -> Address: %s/%d\n", addr, prefix_length);
        }
    }

    freeifaddrs(ifaddr);
}

int main(int argc, char *argv[]) {
    if (argc == 3 && strcmp(argv[1], "-i") == 0) {
        print_interface_info(argv[2]);
    } else if (argc == 2 && strcmp(argv[1], "-a") == 0) {
        print_all_interfaces_with_prefixes();
    } else {
        fprintf(stderr, "Usage:\n");
        fprintf(stderr, "  %s -i <ifname> : Affiche les adresses IPv4 et IPv6 de l'interface spécifiée (avec préfixes)\n", argv[0]);
        fprintf(stderr, "  %s -a         : Affiche la liste des interfaces réseau et leurs préfixes d'adresses IPv4/IPv6\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    return 0;
}
```





