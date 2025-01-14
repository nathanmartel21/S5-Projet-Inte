# Développement de la commande  pour lister les interfaces réseaux d’une machine distante (ifnetshow) :

## Agent :

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <ifaddrs.h>
#include <net/if.h>
#include <netinet/in.h>

#define PORT 8080
#define BUFFER_SIZE 4096

void print_interface_info(const char *ifname, char *output) {
    struct ifaddrs *ifaddr, *ifa;
    char addr[INET6_ADDRSTRLEN];
    char buffer[512];

    if (getifaddrs(&ifaddr) == -1) {
        perror("getifaddrs");
        exit(EXIT_FAILURE);
    }

    output[0] = '\0'; // Clear the output buffer
    for (ifa = ifaddr; ifa != NULL; ifa = ifa->ifa_next) {
        if (ifa->ifa_addr == NULL) continue;
        if (ifname != NULL && strcmp(ifa->ifa_name, ifname) != 0) continue;

        if (ifa->ifa_addr->sa_family == AF_INET) {
            // IPv4
            struct sockaddr_in *in = (struct sockaddr_in *)ifa->ifa_addr;
            struct sockaddr_in *mask = (struct sockaddr_in *)ifa->ifa_netmask;
            inet_ntop(AF_INET, &in->sin_addr, addr, sizeof(addr));
            int prefix_length = __builtin_popcount(ntohl(mask->sin_addr.s_addr));
            snprintf(buffer, sizeof(buffer), "Interface : %s (IPv4) -> Adresse : %s/%d\n",
                     ifa->ifa_name, addr, prefix_length);
            strcat(output, buffer);
        } else if (ifa->ifa_addr->sa_family == AF_INET6) {
            // IPv6
            struct sockaddr_in6 *in6 = (struct sockaddr_in6 *)ifa->ifa_addr;
            struct sockaddr_in6 *mask6 = (struct sockaddr_in6 *)ifa->ifa_netmask;
            inet_ntop(AF_INET6, &in6->sin6_addr, addr, sizeof(addr));

            // Calculer la longueur du préfixe pour IPv6
            int prefix_length = 0;
            if (mask6) {
                unsigned char *mask_bytes = (unsigned char *)&mask6->sin6_addr;
                for (int i = 0; i < 16; i++) {
                    prefix_length += __builtin_popcount(mask_bytes[i]);
                }
            }
            snprintf(buffer, sizeof(buffer), "Interface : %s (IPv6) -> Adresse : %s/%d\n",
                     ifa->ifa_name, addr, prefix_length);
            strcat(output, buffer);
        }
    }
    freeifaddrs(ifaddr);
}

void handle_client(int client_socket) {
    char buffer[BUFFER_SIZE];
    char response[BUFFER_SIZE];
    int bytes_received = recv(client_socket, buffer, BUFFER_SIZE, 0);
    if (bytes_received < 0) {
        perror("recv");
        close(client_socket);
        return;
    }
    buffer[bytes_received] = '\0';

    char ifname[128];
    if (strncmp(buffer, "LIST", 4) == 0) {
        print_interface_info(NULL, response);
    } else if (sscanf(buffer, "IF %s", ifname) == 1) {
        print_interface_info(ifname, response);
    } else {
        snprintf(response, sizeof(response), "Invalid command\n");
    }

    send(client_socket, response, strlen(response), 0);
    close(client_socket);
}

int main() {
    int server_socket, client_socket;
    struct sockaddr_in server_addr, client_addr;
    socklen_t client_addr_len = sizeof(client_addr);

    server_socket = socket(AF_INET, SOCK_STREAM, 0);
    if (server_socket < 0) {
        perror("socket");
        exit(EXIT_FAILURE);
    }

    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = INADDR_ANY;
    server_addr.sin_port = htons(PORT);

    if (bind(server_socket, (struct sockaddr *)&server_addr, sizeof(server_addr)) < 0) {
        perror("bind");
        close(server_socket);
        exit(EXIT_FAILURE);
    }

    if (listen(server_socket, 5) < 0) {
        perror("listen");
        close(server_socket);
        exit(EXIT_FAILURE);
    }

    printf("Agent ecoute sur le port %d...\n", PORT);

    while (1) {
        client_socket = accept(server_socket, (struct sockaddr *)&client_addr, &client_addr_len);
        if (client_socket < 0) {
            perror("accept");
            continue;
        }
        handle_client(client_socket);
    }

    close(server_socket);
    return 0;
}
```

## Client :

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>

#define PORT 8080
#define BUFFER_SIZE 4096

void send_request(const char *server_ip, const char *command) {
    int sock;
    struct sockaddr_in server_addr;
    char buffer[BUFFER_SIZE];

    sock = socket(AF_INET, SOCK_STREAM, 0);
    if (sock < 0) {
        perror("socket");
        exit(EXIT_FAILURE);
    }

    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(PORT);
    if (inet_pton(AF_INET, server_ip, &server_addr.sin_addr) <= 0) {
        perror("inet_pton");
        close(sock);
        exit(EXIT_FAILURE);
    }

    if (connect(sock, (struct sockaddr *)&server_addr, sizeof(server_addr)) < 0) {
        perror("connect");
        close(sock);
        exit(EXIT_FAILURE);
    }

    send(sock, command, strlen(command), 0);
    int bytes_received = recv(sock, buffer, BUFFER_SIZE - 1, 0);
    if (bytes_received > 0) {
        buffer[bytes_received] = '\0';
        printf("%s", buffer);
    } else {
        perror("recv");
    }

    close(sock);
}

int main(int argc, char *argv[]) {
    if (argc < 3) {
        fprintf(stderr, "Usage :\n");
        fprintf(stderr, " %s -n <addr_distante> -a\n", argv[0]);
        fprintf(stderr, " %s -n <addr_distante> -i <ifname>\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    const char *server_ip = argv[2];
    char command[BUFFER_SIZE];

    if (strcmp(argv[3], "-a") == 0) {
        strcpy(command, "LIST");
    } else if (strcmp(argv[3], "-i") == 0 && argc == 5) {
        snprintf(command, sizeof(command), "IF %s", argv[4]);
    } else {
        fprintf(stderr, "Argument(s) invalide(s)\n");
        exit(EXIT_FAILURE);
    }

    send_request(server_ip, command);
    return 0;
}
```

## Usage : 

- Sur la machine cible (machine dont on va lister les interfaces), ouvrir une socket en écoute :

```
./ifnetshow_agent
```

- Sur la machine qui émet la requête :

```
ifnetshow_client -n W.X.Y.Z -a
ifnetshow_client -n W.X.Y.Z -i eth0
```
