# Projet Intégrateur - Systèmes et réseaux virtuels

## Description
Ce projet est un ensemble de travaux pratiques en virtualisation, réseaux, et programmation en C, s'inscrivant dans le cadre de mes études à l'[IMT Mines Alès](https://www.imt-mines-ales.fr/). Il vise à développer mes compétences dans la configuration de machines virtuelles, la gestion de réseaux, et la création de commandes réseau personnalisées sous Linux. Le projet se déroule dans un environnement virtualisé utilisant l'hyperviseur VMware et s'appuie sur des outils de versionnement avec Git.

## Objectifs
- **Appliquer les concepts de virtualisation et de gestion de réseaux** : Création de VM sous VMware, configuration de réseaux en IPv4 et IPv6, et routage.
- **Développement en langage C** : Programmation de commandes réseau pour la gestion et la surveillance des interfaces réseau.
- **Optimisation des ressources** : Réduction de l'empreinte disque et mémoire des VM.
- **Maîtrise des outils de versionnement avec Git** : Organisation du projet en plusieurs étapes et sauvegarde des versions du code.

## Adressage

| Réseaux | Adresses réseaux             |
|---------|------------------------------|
| N0      | 172.16.8.0/24                |
|         | 2002:16:0:0::/64             |
| N2      | 10.8.2.0/24                  |
|         | 2001:0:8:2::/64              |
| N4      | 192.168.8.4/30               |
|         | 3FFE:0:8:4::/64              |

| Réseaux | Adresses réseaux             |
|---------|------------------------------|
| N1      | 10.8.1.0/24                  |
|         | 2001:0:8:1::/64              |
| N3      | 10.8.3.0/24                  |
|         | 2001:0:8:3::/64              |
| N5      | 10.8.5.0/24                  |
|         | 2001:0:8:5::/64              |

*Tableau 1 : Adresses des réseaux de NetLab*

| Interfaces | Adresses interfaces        |
|------------|----------------------------|
| R0 - N1    | 10.8.1.1/24                |
|            | 2001:0:8:1::1/64           |
| R0 - N2    | 10.8.2.1/24                |
|            | 2001:0:8:2::1/64           |
| R0 - N5    | 10.8.5.2/24                |
|            | 2001:8:x:5::2/64           |
| R2 - N3    | 10.8.3.1/24                |
|            | 2001:0:8:3::1/64           |
| R2 - N4    | 192.168.8.6/30             |
|            | 3FFE:0:8:4::2/64           |
| T1 - N1    | 10.8.1.101/24              |
| T2 - N2    | 10.8.2.102/24              |

| Interfaces | Adresses interfaces        |
|------------|----------------------------|
| R1 - N0    | 172.16.0.8/24              |
|            | 2002:16:0:0::8/64            |
| R1 - N4    | 192.168.8.5/30             |
|            | 3FFE:0:8:4::1/64           |
| R1 - N5    | 10.8.5.1/24                |
|            | 2001:0:8:5::1/64           |

| Interfaces | Adresses interfaces        |
|------------|----------------------------|
| T3 - N5    | 10.8.3.103/24              |
| T4 - N3    | 10.8.3.104/24              |

*Tableau 2: Adresses des interfaces réseaux des équipements de NetLab*

## Auteur
@uthor : Nathan Martel
<br>

<p style="color: red; font-weight: bold;">
If you have access to this GitHub repository in any way, you are strictly prohibited from using, reproducing, or distributing any of the resources, code, or research contained within this project. Access to this repository does not grant permission to utilize or share any part of its content without explicit authorization.
</p>
