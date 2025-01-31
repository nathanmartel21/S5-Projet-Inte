Pour alpine, bien ajouter :

```
#include <sys/select.h>
#include <unistd.h>

#include <sys/time.h>
```

Agent HOP : 

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <netdb.h>
#include <ifaddrs.h>

#define PORT 8888
#define BUF_SIZE 1024

// Fonction pour récupérer l'IP distante commençant par "10."
char* get_dist_ip() {
    struct ifaddrs *ifaddr, *ifa;
    int family, s;
    char *ip = NULL;

    if (getifaddrs(&ifaddr) == -1) {
        perror("getifaddrs error");
        return NULL;
    }

    // Parcours des interfaces réseau
    for (ifa = ifaddr; ifa != NULL; ifa = ifa->ifa_next) {
        if (ifa->ifa_addr == NULL)
            continue;

        family = ifa->ifa_addr->sa_family;

        if (family == AF_INET) {
            // Vérification si l'adresse commence par "10."
            struct sockaddr_in *sa = (struct sockaddr_in *) ifa->ifa_addr;
            ip = inet_ntoa(sa->sin_addr);

            if (strncmp(ip, "10.", 3) == 0) {
                break;
            }
        }
    }

    freeifaddrs(ifaddr);
    return ip;
}

int main() {
    int sockfd;
    struct sockaddr_in server_addr, client_addr;
    char buffer[BUF_SIZE];
    char hostname[BUF_SIZE];
    socklen_t addr_len;

    // Création du socket
    if ((sockfd = socket(AF_INET, SOCK_DGRAM, 0)) < 0) {
        perror("Socket error");
        exit(EXIT_FAILURE);
    }

    // Configuration de l'adresse du serveur
    memset(&server_addr, 0, sizeof(server_addr));
    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = htonl(INADDR_ANY);
    server_addr.sin_port = htons(PORT);

    // Liaison du socket
    if (bind(sockfd, (struct sockaddr *)&server_addr, sizeof(server_addr)) < 0) {
        perror("Bind error");
        close(sockfd);
        exit(EXIT_FAILURE);
    }

    printf("Agent en écoute sur le port %d...\n", PORT);

    while (1) {
        addr_len = sizeof(client_addr);
        // Réception de la requête
        int recv_len = recvfrom(sockfd, buffer, BUF_SIZE, 0,
                                (struct sockaddr *)&client_addr, &addr_len);
        if (recv_len < 0) {
            perror("Receive error");
            continue;
        }

        buffer[recv_len] = '\0';
        printf("Requête reçue de la part de %s : %d\n",
               inet_ntoa(client_addr.sin_addr), ntohs(client_addr.sin_port));

        // Récupération du nom de l'hôte
        if (gethostname(hostname, sizeof(hostname)) < 0) {
            perror("Gethostname error");
            close(sockfd);
            exit(EXIT_FAILURE);
        }

        // Récupération de l'IP distante commençant par "10."
        char *ip = get_dist_ip();
        if (ip == NULL) {
            printf("Aucune adresse IP distante valide trouvée.\n");
            continue;
        }

        // Envoi de la réponse : hostname + IP distante
        snprintf(buffer, BUF_SIZE, "%s", ip);
        if (sendto(sockfd, buffer, strlen(buffer), 0,
                   (struct sockaddr *)&client_addr, addr_len) < 0) {
            perror("Sendto error");
        }
    }

    close(sockfd);
    return 0;
}
```

Client HOP : 

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <netdb.h>
#include <sys/socket.h>

#define PORT 8888
#define BUF_SIZE 1024
#define BROADCAST_IP "255.255.255.255"

int ping_ip(const char *ip) {
    char command[BUF_SIZE];
    snprintf(command, sizeof(command), "ping -c 1 %s | grep ttl | awk '{print $6}' | cut -d'=' -f2", ip);

    FILE *fp = popen(command, "r");
    if (fp == NULL) {
        perror("Error opening pipe for ping");
        return -1;
    }

    char result[BUF_SIZE];
    if (fgets(result, sizeof(result), fp) == NULL) {
        fclose(fp);
        return -1;
    }

    fclose(fp);

    int ttl_value = atoi(result);
    return ttl_value;
}

const char *map_ip_to_hostname(const char *ip) {
        if (strcmp(ip, "10.8.1.101") == 0) return "T1";
        if (strcmp(ip, "10.8.2.102") == 0) return "T2";
        if (strcmp(ip, "10.8.3.103") == 0) return "T3";
        if (strcmp(ip, "10.8.3.104") == 0) return "T4";
        if (strcmp(ip, "10.8.1.1") == 0) return "R0";
        if (strcmp(ip, "10.8.2.1") == 0) return "R0";
        if (strcmp(ip, "10.8.5.2") == 0) return "R0";
        if (strcmp(ip, "172.16.0.8") == 0) return "R1";
        if (strcmp(ip, "192.168.8.5") == 0) return "R1";
        if (strcmp(ip, "10.8.5.1") == 0) return "R1";
        if (strcmp(ip, "10.8.3.1") == 0) return "R2";
        if (strcmp(ip, "192.168.8.6") == 0) return "R2";
        if (strcmp(ip, "10.134.34.184") == 0) return "NMARTEL";
        return ip;
}



int main(int argc, char *argv[]) {

    if (argc != 3) {
        printf("Usage : %s -hop [n]\n", argv[0]);
        return 1;
    }

    int n = atoi(argv[2]);

    int sockfd;
    struct sockaddr_in broadcast_addr, recv_addr;
    char buffer[BUF_SIZE];
    socklen_t addr_len;
    fd_set readfds;
    struct timeval timeout;
    char *unique_hosts[100];
    int unique_count = 0;

    // Création du socket
    if ((sockfd = socket(AF_INET, SOCK_DGRAM, 0)) < 0) {
        perror("Socket error");
        exit(EXIT_FAILURE);
    }

    // Configuration pour le broadcast
    int broadcast_enable = 1;
    if (setsockopt(sockfd, SOL_SOCKET, SO_BROADCAST, &broadcast_enable, sizeof(broadcast_enable)) < 0) {
        perror("Setsockopt error");
        close(sockfd);
        exit(EXIT_FAILURE);
    }

    // Configuration de l'adresse de diffusion
    memset(&broadcast_addr, 0, sizeof(broadcast_addr));
    broadcast_addr.sin_family = AF_INET;
    broadcast_addr.sin_addr.s_addr = inet_addr(BROADCAST_IP);
    broadcast_addr.sin_port = htons(PORT);

    // Envoi de la requête en broadcast
    const char *message = "WHO_IS_THERE";
    if (sendto(sockfd, message, strlen(message), 0,
               (struct sockaddr *)&broadcast_addr, sizeof(broadcast_addr)) < 0) {
        perror("Sendto error");
        close(sockfd);
        exit(EXIT_FAILURE);
    }

    printf("Requête envoyée, récup des réponses...\n");

    // Réception des réponses
    addr_len = sizeof(recv_addr);
    FD_ZERO(&readfds);
    FD_SET(sockfd, &readfds);
    timeout.tv_sec = 5;  // Timeout de 5 secondes
    timeout.tv_usec = 0;

    while (select(sockfd + 1, &readfds, NULL, NULL, &timeout) > 0) {
        int recv_len = recvfrom(sockfd, buffer, BUF_SIZE, 0,
                                (struct sockaddr *)&recv_addr, &addr_len);
        if (recv_len < 0) {
            perror("Receive error");
            continue;
        }

        buffer[recv_len] = '\0';
        // printf("Host identifié (réponse reçue) : %s\n", buffer);

        // Vérification des doublons
        int is_unique = 1;
        for (int i = 0; i < unique_count; i++) {
            if (strcmp(unique_hosts[i], buffer) == 0) {
                is_unique = 0;
                break;
            }
        }

        if (is_unique && unique_count < 100) {
            unique_hosts[unique_count] = strdup(buffer);
            unique_count++;
        }
    }

    // Affichage des résultats et ping des machines
    printf("\nMachines voisines détectées :\n");
    for (int i = 0; i < unique_count; i++) {
        // printf("- %s\n", unique_hosts[i]);
        // printf("Ping de %s...\n", unique_hosts[i]);
        int testttl = ping_ip(unique_hosts[i]);
        int somme = n + testttl;

        if (somme == 129){ // exeptionnellement, c'est 128 la
            const char *hostname = map_ip_to_hostname(unique_hosts[i]);
            printf("  - %s (%s)\n", unique_hosts[i], hostname);
        }

        free(unique_hosts[i]);
    }

    close(sockfd);
    return 0;
}
```
