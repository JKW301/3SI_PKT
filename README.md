# Projet PKT - Documentation de l'Infrastructure

**Auteurs** : Julien KHALIFA & Alexandre UZAN  
**Année académique** : ESGI 3SI2, 2024-2025

---

## Table des Matières

1. [Scénario](#scénario)
2. [Conception de la Topologie Réseau](#conception-de-la-topologie-réseau)
3. [LANs et Plan d’Adressage](#lans-et-plan-dadressage)
4. [Câblage et Équipements Utilisés](#câblage-et-équipements-utilisés)
5. [Sécurisation](#sécurisation)
6. [Configuration HSRP](#configuration-hsrp)
7. [Service DHCP](#service-dhcp)
8. [Routage](#routage)
9. [Configuration des ACLs](#configuration-des-acls)
10. [Configurations VLAN et Switch](#configurations-vlan-et-switch)
11. [Bornes Wi-Fi](#bornes-wi-fi)

---

## Scénario

LAB_Security, une société spécialisée en audit et sécurité logicielle, nécessite un réseau IPv4 sécurisé interconnectant ses 5 sites (Paris, Rennes, Strasbourg, Grenoble, Bordeaux).  
- **Objectifs** :
  - Routage centralisé par OSPF.
  - Configuration dynamique des hôtes.
  - Sécurisation via ACLs, VLANs et HSRP.

---

## Conception de la Topologie Réseau

- **Équipements principaux** :
  - Routeurs : 1 principal au siège, 4 pour les agences.
  - Switches L2 pour les LANs.
  - Routeur FAI pour l'accès Internet.
- **Organisation** :
  - Routage centralisé par le siège.
  - VLANs pour segmentation des services au siège.

---

## LANs et Plan d’Adressage

| Site                | Hôtes    | Adresse IP         | Gateway         |
|---------------------|----------|--------------------|-----------------|
| Siège (Paris)       | 580+152  | `10.0.0.0/22`      | -               |
| Rennes - LAN 1      | 500      | `10.4.0.0/23`      | `10.4.4.1`      |
| Rennes - LAN 2      | 500      | `10.4.2.0/23`      | `10.4.4.1`      |
| Strasbourg - LAN 1  | 120      | `172.16.0.0/25`    | `172.16.0.1`    |
| Strasbourg - LAN 2  | 200      | `172.16.1.0/24`    | `172.16.0.1`    |

---

## Câblage et Équipements Utilisés

- **Routeurs Cisco 2911** et **Switches Cisco 2960**.
- Types de câbles :
  - Série : Connexions WAN.
  - Ethernet Straight-Through : Routeur-Switch et Switch-PC.

---

## Sécurisation

- **Passwords** :
  - Utilisation de `enable secret` pour le chiffrement MD5.
  - Commande `service password-encryption` activée.
- **Authentification OSPF** :
  - Utilisation de `ip ospf authentication message-digest`.

---

## Configuration HSRP

Redondance configurée entre deux routeurs :
- Adresse IP virtuelle : `10.0.3.250`.
- Priorités : 110 pour HSRP1 (principal), 90 pour HSRP2 (secours).

---

## Service DHCP

- **Siège** : VLANs définis avant configuration DHCP.
- **Exemple de Pool** :
```plaintext
  ip dhcp pool LAN_SIEGE
  network 10.0.0.0 255.255.252.0
  default-router 10.0.3.250
  dns-server 10.255.0.2
  end
  write memory
```
## Routage

    Statique : Initialement utilisé pour les chemins de secours.
    Dynamique avec OSPF : Adopté pour une meilleure flexibilité.

## Configuration des ACLs

Règles principales :
- Autoriser HTTP (80), HTTPS (443), DHCP, OSPF.
- Bloquer les pings entrants.
- Refuser tout autre trafic par défaut.

Par exemple : 
```
ip access-list extended 102
permit tcp any any eq www
permit tcp any any eq 443
deny icmp any any echo
deny ip any any
```

## Configurations VLAN et Switch

 ### Organisation :
- VLANs par cœur de métier.
- Isolation via ACLs.

### Adresses IP par VLAN :
 
 VLAN partagé pour serveurs : 192.168.1.3.

## Bornes Wi-Fi

Configuration :
        WPA2-PSK avec mot de passe sécurisé.
        Réseau invité : guest_wifi.
    Recommandations :
        Wi-Fi 6 (802.11ax) pour 40-50 connexions simultanées.
