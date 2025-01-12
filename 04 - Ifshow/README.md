# Développement de la commande pour lister les interfaces réseaux de la machine (ifshow) :

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

## Sur Alpine Linux :

```
empty
```

## Sur VyOS :  

```
empty
```

## Sur machine hôte (Win64) :  

```
#include <stdio.h>
#include <stdlib.h>
#include <winsock2.h>
#include <ws2tcpip.h>
#include <iphlpapi.h>

#pragma comment(lib, "ws2_32.lib")
#pragma comment(lib, "iphlpapi.lib")

void print_interface_info(const char *ifname) {
    DWORD dwSize = 0;
    PIP_ADAPTER_ADDRESSES pAddresses = NULL, pCurrAddress = NULL;

    // Récupération des informations des interfaces
    if (GetAdaptersAddresses(AF_UNSPEC, GAA_FLAG_INCLUDE_PREFIX, NULL, pAddresses, &dwSize) == ERROR_BUFFER_OVERFLOW) {
        pAddresses = (PIP_ADAPTER_ADDRESSES)malloc(dwSize);
    }

    if (GetAdaptersAddresses(AF_UNSPEC, GAA_FLAG_INCLUDE_PREFIX, NULL, pAddresses, &dwSize) == NO_ERROR) {
        for (pCurrAddress = pAddresses; pCurrAddress != NULL; pCurrAddress = pCurrAddress->Next) {
            // Comparer les noms d'interfaces si spécifié
            if (ifname && strcmp(pCurrAddress->AdapterName, ifname) != 0) {
                continue;
            }

            printf("Interface: %s\n", pCurrAddress->AdapterName);

            // Parcourir les adresses associées
            for (IP_ADAPTER_UNICAST_ADDRESS *pUnicast = pCurrAddress->FirstUnicastAddress; pUnicast != NULL; pUnicast = pUnicast->Next) {
                char addressBuffer[INET6_ADDRSTRLEN];
                DWORD prefixLength = 0;

                if (pUnicast->Address.lpSockaddr->sa_family == AF_INET) {
                    // IPv4
                    struct sockaddr_in *sa_in = (struct sockaddr_in *)pUnicast->Address.lpSockaddr;
                    inet_ntop(AF_INET, &(sa_in->sin_addr), addressBuffer, sizeof(addressBuffer));
                    prefixLength = pUnicast->OnLinkPrefixLength;
                    printf("  IPv4 -> Address: %s/%lu\n", addressBuffer, prefixLength);
                } else if (pUnicast->Address.lpSockaddr->sa_family == AF_INET6) {
                    // IPv6
                    struct sockaddr_in6 *sa_in6 = (struct sockaddr_in6 *)pUnicast->Address.lpSockaddr;
                    inet_ntop(AF_INET6, &(sa_in6->sin6_addr), addressBuffer, sizeof(addressBuffer));
                    prefixLength = pUnicast->OnLinkPrefixLength;
                    printf("  IPv6 -> Address: %s/%lu\n", addressBuffer, prefixLength);
                }
            }
        }
    }

    free(pAddresses);
}

void print_all_interfaces_with_prefixes() {
    DWORD dwSize = 0;
    PIP_ADAPTER_ADDRESSES pAddresses = NULL, pCurrAddress = NULL;

    // Récupération des informations des interfaces
    if (GetAdaptersAddresses(AF_UNSPEC, GAA_FLAG_INCLUDE_PREFIX, NULL, pAddresses, &dwSize) == ERROR_BUFFER_OVERFLOW) {
        pAddresses = (PIP_ADAPTER_ADDRESSES)malloc(dwSize);
    }

    if (GetAdaptersAddresses(AF_UNSPEC, GAA_FLAG_INCLUDE_PREFIX, NULL, pAddresses, &dwSize) == NO_ERROR) {
        for (pCurrAddress = pAddresses; pCurrAddress != NULL; pCurrAddress = pCurrAddress->Next) {
            printf("Interface: %s\n", pCurrAddress->AdapterName);

            // Parcourir les adresses associées
            for (IP_ADAPTER_UNICAST_ADDRESS *pUnicast = pCurrAddress->FirstUnicastAddress; pUnicast != NULL; pUnicast = pUnicast->Next) {
                char addressBuffer[INET6_ADDRSTRLEN];
                DWORD prefixLength = 0;

                if (pUnicast->Address.lpSockaddr->sa_family == AF_INET) {
                    // IPv4
                    struct sockaddr_in *sa_in = (struct sockaddr_in *)pUnicast->Address.lpSockaddr;
                    inet_ntop(AF_INET, &(sa_in->sin_addr), addressBuffer, sizeof(addressBuffer));
                    prefixLength = pUnicast->OnLinkPrefixLength;
                    printf("  IPv4 -> Address: %s/%lu\n", addressBuffer, prefixLength);
                } else if (pUnicast->Address.lpSockaddr->sa_family == AF_INET6) {
                    // IPv6
                    struct sockaddr_in6 *sa_in6 = (struct sockaddr_in6 *)pUnicast->Address.lpSockaddr;
                    inet_ntop(AF_INET6, &(sa_in6->sin6_addr), addressBuffer, sizeof(addressBuffer));
                    prefixLength = pUnicast->OnLinkPrefixLength;
                    printf("  IPv6 -> Address: %s/%lu\n", addressBuffer, prefixLength);
                }
            }
        }
    }

    free(pAddresses);
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





