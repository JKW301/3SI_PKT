# **Documentation de l’infrastructure**  

**[Conception de la Topologie Réseau	3](#conception-de-la-topologie-réseau)**

[LANs et plan d’adressage	3](#lans-et-plan-d’adressage)

[Type de Câblage et Équipements Utilisés	4](#type-de-câblage-et-équipements-utilisés)

[**Sécurisation	6**](#sécurisation)

[Passwords	6](#passwords)

[Authentification OSPF	7](#authentification-ospf)

[**Configuration de la Redondance HSRP (Hot Standby Router Protocol)	8**](#configuration-de-la-redondance-hsrp-\(hot-standby-router-protocol\))

[**Service DHCP pour les Hôtes :	9**](#service-dhcp-pour-les-hôtes-:)

[**Routage	11**](#routage)

[Routage Statique entre le Siège et les Agences :	11](#routage-statique-entre-le-siège-et-les-agences-:)

[Routage Dynamique OSPF :	12](#routage-dynamique-ospf-:)

[**Configuration des ACLs	13**](#configuration-des-acls)

[**Configurations VLAN et switch	17**](#configurations-vlan-et-switch)

[Sécurité	18](#sécurité)

[Un VLAN dit “partagé”	19](#un-vlan-dit-“partagé”)

[**Bornes Wi-Fi	20**](#bornes-wi-fi)

**Scénario :**

En tant qu’équipe d’experts en réseau et sécurité, notre mission est de concevoir et mettre en place un réseau sécurisé pour LAB\_Security, une société spécialisée dans les solutions d’audit et de sécurité logicielle. L'objectif est de créer une infrastructure réseau IPv4 qui supporte la configuration dynamique des hôtes et le routage OSPF. LAB\_Security compte quatre agences situées à Rennes, Strasbourg, Grenoble, et Bordeaux, toutes connectées au siège de la société à Paris, qui centralise l'accès à Internet via un FAI. L'infrastructure réseau de LAB\_Security comprend plusieurs sites interconnectés et un siège central. 

**Sites distants (Agences)** :

- 4 agences situées à Rennes, Strasbourg, Grenoble, et Bordeaux.  
- Chaque agence est connectée au siège par un routeur.  
- Les agences ne communiquent pas directement entre elles ; elles passent par le siège.  
- Le siège central héberge l'essentiel des équipements serveurs et joue le rôle de hub pour toutes les agences.  
- Il est connecté au routeur d'un FAI pour l'accès Internet.  
- Un routeur FAI pour la connexion Internet du siège.

# **Conception de la Topologie Réseau** {#conception-de-la-topologie-réseau}

Pour répondre aux besoins de LAB\_Security, nous avons structuré le réseau de la manière suivante :

**Routeurs et Commutateurs :**  
Un routeur principal pour le siège de LAB\_Security, situé à Paris.  
Quatre routeurs pour les agences à Rennes, Strasbourg, Grenoble, et Bordeaux, connectés au routeur du siège.  
Un routeur de FAI connecté au routeur du siège pour centraliser l'accès à Internet.  
Des switches L2 pour les réseaux locaux dans chaque agence et le siège.  
**Configuration WAN (Réseaux Série) :**  
Les routeurs des agences et du siège sont connectés via des liaisons série.  
Chaque routeur d'agence communique uniquement avec le routeur du siège et non avec les autres agences, permettant un routage centralisé via OSPF.  
**VLANs et Réseaux Locaux :**  
Le siège est divisé en VLANs pour chaque services, avec une configuration adaptée pour les serveurs et hôtes, permettant la segmentation et la gestion des flux de données.  
Chaque agence dispose de sous-réseaux séparés pour les différents groupes d’utilisateurs et d'équipements.

# 

## **LANs et plan d’adressage** {#lans-et-plan-d’adressage}

Chaque site est subdivisé en plusieurs LANs, chacun avec un nombre spécifique de postes et une adresse IP assignée.

| Site | Nombre d'hôtes | Adresse IP | Router Gateway |
| ----- | ----- | ----- | ----- |
| Siège (Paris) | 580 hôtes \+ 152 serveurs | 10.0.0.0/22 | \- |
| Site de Rennes \- LAN 1 | 500 | 10.4.0.0/23 | 10.4.4.1 |
| Site de Rennes \- LAN 2 | 500 | 10.4.2.0/23 | 10.4.4.1 |
| Site de Strasbourg \- LAN 1 | 120 | 172.16.0.0/25 | 172.16.0.1 |
| Site de Strasbourg \- LAN 2 | 200 | 172.16.1.0/24 | 172.16.0.1 |
| Site de Grenoble \- LAN 1 | 80 | 192.168.0.0/25 | 192.168.0.126 |
| Site de Grenoble \- LAN 2 | 80 | 192.168.0.128/25 | 192.168.0.254 |
| Site de Bordeaux \- LAN 1 | 60 | 192.168.1.0/26 | 192.168.1.1 |
| Site de Bordeaux \- LAN 2 | 27 | 192.168.1.64/27 | 192.168.1.1 |
| FAI \- LAN | 3 | 203.0.13.0/30 | \- |

# 

## **Type de Câblage et Équipements Utilisés** {#type-de-câblage-et-équipements-utilisés}

# **Équipements Réseau :** 

| Routeurs |  | Switches |  | Bornes |
| :---- | :---- | :---- | :---- | :---- |
| Cisco 2911 | x6 | Cisco 2960 | x9 | AP-PT |

**Types de Câbles Utilisés**

* **Câble Série** : Utilisé pour les connexions WAN entre les routeurs (agences, siège et FAI). Exemple : liaison entre le siège et le routeur FAI.  
* **Câble Ethernet Copper Straight-Through** : Utilisé pour connecter des équipements de types différents, tels que :  
  * **Routeur-Switch** : Assure la communication entre le routeur central et les switches des réseaux locaux.  
  * **Switch-PC** : Permet de relier les périphériques finaux (PC) aux switches pour un accès réseau.

# 

# **Étapes de Configuration dans Packet Tracer**

Tout d’abord, voici un aperçu de notre projet : 

![][image1]

**Routeurs Hostname**

| Routeurs | Switchs | Adresse IP | Masque |
| :---- | :---- | :---- | :---- |
| HSRP1 |  sw\_siege\_1 |  VLAN |  |
| HSRP2 |  |  |  |
|  |  |  |  |
| routeur\_rennes | sw\_rennes\_1 | 10.1.0.0 | 255.255.254.0 |
|  | sw\_rennes\_2 | 10.1.2.0 | 255.255.254.0 |
| routeur\_strasbourg | sw\_strasbourg\_1 | 172.16.0.0 | 255.255.255.128 |
|  | sw\_strasbourg\_2 | 172.16.1.0 | 255.255.255.0 |
| routeur\_grenoble | sw\_grenoble\_1 | 192.168.0.0 | 255.255.255.128 |
|  | sw\_grenoble\_2 | 192.168.0.128 | 255.255.255.128 |
| routeur\_bordeaux | sw\_bordeaux\_1 | 192.168.1.0 | 255.255.255.192 |
|  | sw\_bordeaux\_2 | 192.168.1.64 | 255.255.255.224 |

Configuration renommage des routeurs exemple : 

| Router\>enableRouter\#configure terminalRouter(config)\#hostname Le\_NOM\_DU\_ROUTEURMonRouteur(config)\#exitMonRouteur\#write memory |
| :---- |

# **Sécurisation** {#sécurisation}

## Passwords {#passwords}

Sur chaque routeur du réseau, nous avons appliqué un mot-de-passe en CLI. 

Ce “password” a pour but de s’assurer que seuls les techniciens autorisés et donc ayant connaissance du mot-de-passe, pourront interagir et modifier les configuration du routeur. 

Le mot de passe sur un routeur relève de la **couche 7 (Application)** du modèle OSI, car il fait partie des mécanismes d'accès et de contrôle des services administratifs fournis par le routeur.

Ci-dessous une illustration de la procédure : 

| router-rennes\#configure terminal Enter configuration commands, one per line.  End with CNTL/Z. router-rennes(config)\#hostname router-rennes router-rennes(config)\#enable secret Azerty1\! router-rennes(config)\#service password-encryption router-rennes(config)\#exit |
| :---- |

**enable secret Azerty1\!** : Cette commande définit un mot de passe chiffré pour protéger l'accès privilégié (mode enable). Contrairement à enable password, qui stocke le mot de passe en clair dans la configuration. 

**enable secret** utilise un chiffrement MD5, offrant une protection plus robuste contre l'accès non autorisé.

| Routeurs | Password |
| :---- | :---- |
| HSRP1 | Qwertz4@ |
| HSRP2 | Qwertz8$ |
| routeur\_rennes | Azerty1\! |
| routeur\_strasbourg | Azerty1\! |
| routeur\_grenoble | Azerty1\! |
| routeur\_bordeaux | Azerty1\! |

**service password-encryption** : Cette commande active le chiffrement des mots de passe stockés en clair dans la configuration du routeur (par exemple, les mots de passe pour les lignes console ou VTY). Bien que le chiffrement soit faible (type 7), il empêche un accès direct en clair à ces mots de passe dans le fichier de configuration.

## **Authentification OSPF** {#authentification-ospf}

**OSPF (Open Shortest Path First)** est un protocole de routage qui opère à la couche réseau pour échanger des informations sur les routes disponibles.

Cette configuration met en place une authentification OSPF avec une clé de type message-digest (MD5) sur l’interface réseau menant à un autre routeur du réseau.

 La commande **ip ospf authentication message-digest** active cette méthode d'authentification sécurisée, qui garantit que les échanges OSPF sur cette interface sont validés à l'aide d'une clé partagée. 

La commande **ip ospf message-digest-key 1 md5 securepassword** définit une clé d'authentification (ici *secure password*) associée à un identifiant (ici 1). 

Cela empêche les routeurs non autorisés de participer à l'échange OSPF, renforçant ainsi la sécurité du réseau en évitant l'injection de routes malveillantes.

Cette authentification adresse principalement des failles de sécurité comme  :

**Prévention des attaques par usurpation de routeur (spoofing)** : Sans authentification, un attaquant pourrait prétendre être un routeur légitime dans le réseau, injectant de fausses informations de routage.

Cela peut entraîner des interruptions de service ou rediriger le trafic vers des destinations malveillantes.

**Intégrité des données** : L'authentification garantit que les mises à jour OSPF échangées entre routeurs n'ont pas été altérées en transit. (*ces mise à jour peuvent être forcées via : ‘clear ip ospf process’).* 

Cela empêche des modifications malveillantes ou accidentelles des tables de routage.

**Protection contre les routeurs non autorisés** : L'authentification assure que seuls les routeurs disposant de la clé partagée (clé MD5 ou SHA, selon la configuration) peuvent participer au processus OSPF. 

Ainsi, un attaquant ne pourra perturber le service en insérant un routeur dans le réseau.

# **Configuration de la Redondance HSRP (Hot Standby Router Protocol)** {#configuration-de-la-redondance-hsrp-(hot-standby-router-protocol)}

La configuration de la redondance HSRP (Hot Standby Router Protocol) garantit une haute disponibilité en basculant automatiquement le trafic réseau vers un routeur de secours (HSRP2) en cas de défaillance du routeur principal (HSRP1), assurant ainsi une continuité de service optimale.

**Sur HSRP1** (routeur principal) :

| enableconfigure terminalinterface GigabitEthernet0/0ip address 10.0.3.254 255.255.255.0standby 1 ip 10.0.3.250      	standby 1 priority 110        	standby 1 preempt             	no shutdownendwrite memory |
| :---- |

**Sur HSRP 2** (routeur de secours) :

| enableconfigure terminalinterface GigabitEthernet0/0ip address 10.0.3.253 255.255.255.0standby 1 ip 10.0.3.250      \< \--- 	Même IP virtuelle partagéestandby 1 priority 90        \< \-- 	Priorité plus basse pour HSRP2standby 1 preemptno shutdownendwrite memory |
| :---- |

Les deux routeurs HSRP ont chacun leur propre adresse IP pour leur interface GigabitEthernet0/0, connectée au switch : HSRP1 utilise l'IP 10.0.3.254 et HSRP2 utilise 10.0.3.253. 

Ils partagent aussi une IP virtuelle commune, 10.0.3.250, qui sert de passerelle par défaut pour le réseau. 

Cette IP virtuelle permet de basculer automatiquement sur HSRP2 si HSRP1 tombe en panne, sans que les utilisateurs aient besoin de changer de passerelle.

**Vérification de l’état de HSRP** :

| show standby brief |
| :---- |

# **Service DHCP pour les Hôtes :**  {#service-dhcp-pour-les-hôtes-:}

Permettre une attribution d’adresses IP dynamiques aux hôtes du siège et des agences.

Avant de configurer le DHCP pour le siège, il est important de définir les VLANs pour organiser le réseau.

**Configuration des VLANs pour le Siège :** 

**Sur HSRP1** :

| enableconfigure terminalinterface GigabitEthernet0/1ip address 10.1.0.1 255.255.255.0no shutdownexitinterface GigabitEthernet0/1.1encapsulation dot1Q 1ip address 192.168.1.1 255.255.255.0no shutdowninterface GigabitEthernet0/1.2encapsulation dot1Q 2ip address 192.168.2.1 255.255.255.0no shutdowninterface GigabitEthernet0/1.3encapsulation dot1Q 3ip address 192.168.3.1 255.255.255.0no shutdown |
| :---- |

Après avoir configuré les VLANs, nous pouvons définir les pools DHCP pour chaque VLAN et pour les agences.

**Configuration DHCP sur HSRP1 pour le siège :**

| enableconfigure terminalip dhcp pool LAN\_SIEGEnetwork 10.0.0.0 255.255.252.0default-router 10.0.3.250      dns-server 10.255.0.2	ip dhcp excluded-address 10.0.3.253 10.0.3.254 endwrite memory |
| :---- |

##### 

**Configuration DHCP sur les routeurs d’agences (exemple pour Rennes) :**

| enableconfigure terminalip dhcp pool LAN\_RENNES\_1network 10.1.0.0 255.255.254.0default-router 10.1.1.254 dns-server 10.255.0.2ip dhcp excluded-address 10.1.1.254ip dhcp pool LAN\_RENNES\_2network 10.1.2.0 255.255.254.0default-router 10.1.3.254 dns-server 10.255.0.2ip dhcp excluded-address 10.1.3.254endwrite memory |
| :---- |

Dans les pools DHCP : 

- le “default-router” correspond à la passerelle par défaut, soit l’adresse IP de l’interface réseau sur lequel est branché le réseau recevant le pool  
- le “dns-server” corresponds au serveur DNS assurant la résolution inverse et la résolution d’adresse IP.

Dans le cadre de cette étude, le serveur DNS imite le service assuré par Google (8.8.8.8) ou encore Cloudflare

# 

# **Routage** {#routage}

## Routage Statique entre le Siège et les Agences :  {#routage-statique-entre-le-siège-et-les-agences-:}

Au début, nous avons utilisé des **routes statiques** pour connecter le siège et les agences, ce qui nous permettait de contrôler manuellement les chemins que le trafic devait suivre. 

Cependant, ce système devenait complexe à gérer à mesure que le réseau grandissait. Pour avoir une meilleure gestion et offrir plus de flexibilité, nous avons ensuite opté pour **OSPF**, un protocole de routage dynamique qui ajuste automatiquement les chemins en fonction des changements dans le réseau.

la connectivité entre le siège et chaque agence via des routes statiques.

**Sur HSRP1** (route principale vers Rennes) :

| enableconfigure terminalip route 10.1.0.0 255.255.254.0 10.0.4.10ip route 10.1.2.0 255.255.254.0 10.0.4.10endwrite memory |
| :---- |

**Sur HSRP2** (route de secours vers Rennes) :

| enableconfigure terminalip route 10.1.0.0 255.255.254.0 10.0.4.26ip route 10.1.2.0 255.255.254.0 10.0.4.26endwrite memory |
| :---- |

## 

## **Routage Dynamique OSPF :** {#routage-dynamique-ospf-:}

Nous avions initialement envisagé d'utiliser des routes statiques, mais pour une meilleure maintenance, le routage dynamique avec OSPF a été préféré. 

Permettre un routage dynamique entre le siège, les agences, et le FAI pour une bonne connectivité.

**Sur HSRP1** (configuration OSPF) :

| enableconfigure terminalrouter ospf 1router-id 1.1.1.1network 10.0.4.8 0.0.0.3 area 0network 10.0.4.12 0.0.0.3 area 0network 10.0.4.16 0.0.0.3 area 0network 10.0.0.0 0.0.3.255 area 0default-information originate   exitendwrite memory |
| :---- |

**Sur HSRP2** : 

| enableconfigure terminalrouter ospf 1router-id 2.2.2.2network 10.0.4.24 0.0.0.3 area 0network 10.0.4.28 0.0.0.3 area 0network 10.0.0.0 0.0.3.255 area 0default-information originateexitendwrite memory |
| :---- |

**Sur les routeurs des agences** (exemple pour Rennes) :

| enableconfigure terminalrouter ospf 1router-id 3.3.3.3network 10.0.4.8 0.0.0.3 area 0network 10.1.0.0 0.0.1.255 area 0network 10.1.2.0 0.0.1.255 area 0exitendwrite memory |
| :---- |

**Sur le routeur du FAI** :

| enableconfigure terminalrouter ospf 1router-id 7.7.7.7network 203.0.13.0 0.0.0.3 area 0exitip route 0.0.0.0 0.0.0.0 203.0.13.1endwrite memory |
| :---- |

# **Configuration des ACLs**  {#configuration-des-acls}

Dans l’objectif de sécuriser l’infrastructure réseau de LAB\_Security tout en assurant une gestion simple et efficace, nous avons défini une ACL (Access Control List) étendue sur le routeur HSRP1 et le routeur de secours HSRP2.   
Les Access Control Lists (ACL) fonctionnent principalement à la couche 3 (Réseau) et à la couche 4 (Transport) :

* Couche 3 (Réseau) : Elles filtrent les paquets IP en fonction des adresses source et destination.  
* Couche 4 (Transport) : Elles appliquent des filtres basés sur les ports et les protocoles (comme TCP, UDP, ou ICMP).

Cette configuration joue donc  un rôle clé dans le filtrage du trafic réseau, en établissant des règles qui définissent quel type de trafic est autorisé ou refusé à entrer ou sortir du réseau.

La logique derrière cette configuration repose sur deux principes fondamentaux : d’une part, faciliter les opérations internes et le dépannage rapide en permettant les flux nécessaires à l’intérieur du réseau ; et d’autre part, protéger le réseau contre les accès non autorisés ou les attaques venant de l’extérieur.

- **Autorisation des flux HTTP et HTTPS :**

L’ACL autorise explicitement le trafic web via les ports 80 (HTTP) et 443 (HTTPS). Cette règle garantit que les utilisateurs internes peuvent accéder à Internet pour naviguer sur des sites web ou utiliser des services en ligne essentiels. Ce choix reflète la nécessité pour LAB\_Security de disposer d’un accès fluide aux ressources en ligne, tout en filtrant uniquement les flux strictement nécessaires.

- **Support du DHCP :**

Dans le cadre de la gestion des adresses IP, le DHCP est indispensable. L’ACL permet au trafic DHCP de circuler librement, ce qui permet aux équipements internes de recevoir automatiquement leur configuration réseau. Cela simplifie grandement la gestion des postes de travail et garantit une connectivité rapide et fiable, même lorsque de nouveaux appareils sont ajoutés au réseau.

- **Autorisation du protocole OSPF :**

OSPF est utilisé pour le routage dynamique au sein de l’infrastructure. En autorisant OSPF, nous assurons que les routeurs peuvent échanger leurs informations de routage sans restriction. Cela est crucial pour maintenir une connectivité stable entre les différents sites de l’entreprise, particulièrement lorsque des chemins alternatifs doivent être empruntés en cas de panne.

- **Blocage des pings entrants :**

Les requêtes ICMP de type Echo (ping) provenant de l’extérieur sont bloquées. Cela réduit la visibilité du réseau interne pour les attaquants potentiels, qui utilisent souvent le ping pour détecter les équipements accessibles ou pour préparer des attaques. En effet, empêcher les pings venant d'Internet vers le réseau interne répond à plusieurs préoccupations de sécurité. Tout d'abord, cela protège contre les tentatives de balayage réseau, où les attaquants utilisent des pings pour identifier les équipements actifs dans le réseau. De plus, cela limite l'exposition aux attaques ICMP, telles que les inondations ICMP ou des abus plus complexes comme les attaques Smurf. Enfin, cette mesure réduit les informations exposées à l'extérieur, rendant plus difficile pour un attaquant de cartographier le réseau ou d'évaluer la disponibilité des ressources internes.

En bloquant les pings entrants, on renforce la confidentialité et la sécurité en minimisant les opportunités de reconnaissance et d'exploitation de la part des acteurs malveillants.

Cependant, les pings restent autorisés à l’intérieur d’un sous-réseau, permettant aux équipes techniques de tester rapidement la connectivité entre les appareils.

- **Refus de tout autre trafic :**

Enfin, toute forme de trafic non spécifiquement autorisée est bloquée par défaut. Cela empêche les flux inattendus ou malveillants de pénétrer dans le réseau, renforçant ainsi la posture de sécurité globale de LAB\_Security.

**Dans HSRP1 :** 

Les ACLs (Access Control Lists) permettent de contrôler les flux de trafic qui transitent par un routeur. Dans notre configuration, nous avons mis en place une ACL étendue 102, sur HSRP1, avec des règles spécifiques pour sécuriser le réseau tout en garantissant l'accès aux services essentiels. 

| enableconfigure terminal\! Créer l'ACL 102ip access\-list extended 102\! Autoriser le trafic HTTP et HTTPSpermit tcp any any eq wwwpermit tcp any any eq 443\! Autoriser DHCPpermit udp any eq bootps any eq bootpcpermit udp any eq bootpc any eq bootps\! Autoriser OSPFpermit ospf any any\! Bloquer les pings entrantsdeny icmp any any echo\! Bloquer tout autre traficdeny ip any anyendwrite memoryinterface GigabitEthernet0/0ip access\-group 102 inend |
| :---- |

À l’intérieur du réseau, l’ACL est volontairement permissive pour simplifier les interventions techniques. 

Par exemple, les techniciens de LAB\_Security doivent pouvoir effectuer des tests de connectivité, comme des pings ou des traceroutes, sans être bloqués par des restrictions inutiles. Ce choix reflète un équilibre entre sécurité et flexibilité opérationnelle.

Nous allons aussi présenter et détailler la configuration des groupes ACL INBOUND et OUTBOUND, en précisant leurs règles respectives ainsi que les interfaces sur lesquelles elles sont appliquées pour garantir une gestion sécurisée et efficace du trafic réseau

Les ACL INBOUND, appliquées sur les interfaces connectées aux réseaux externes, filtrent le trafic entrant pour autoriser uniquement les flux nécessaires, tels que HTTP, HTTPS, DHCP, et OSPF, tout en bloquant les pings et tout autre trafic non spécifiquement autorisé.

De leur côté, les ACL OUTBOUND, configurées sur les interfaces internes, contrôlent le trafic sortant en limitant les flux inutiles ou potentiellement malveillants, tout en permettant les services essentiels comme le DHCP, le routage dynamique (OSPF), et les communications web sécurisées.

Ces configurations assurent une sécurité optimale tout en préservant la fluidité des opérations réseau.

**ACL INBOUND :**

Configuration des règles pour filtrer le trafic entrant :

| access-list 102 permit tcp any any eq 80  	\! Autoriser HTTPaccess-list 102 permit tcp any any eq 443 	\! Autoriser HTTPSaccess-list 102 permit udp any any eq 67  	\! Autoriser DHCP (serveur)access-list 102 permit udp any any eq 68  	\! Autoriser DHCP (client)access-list 102 permit ospf any any       	\! Autoriser OSPFaccess-list 102 deny icmp any any echo    	\! Bloquer les pings entrantsaccess-list 102 deny ip any any           	\! Bloquer tout autre trafic |
| :---- |

**Application sur l'interface externe :**

| interface GigabitEthernet0/0ip access-group 102 in |
| :---- |

**ACL OUTBOUND :**

| Configuration des règles pour contrôler le trafic sortant :access-list 103 permit tcp any any eq 80  	\! Autoriser HTTPaccess-list 103 permit tcp any any eq 443 	\! Autoriser HTTPSaccess-list 103 permit udp any eq 68 any eq 67\! Autoriser DHCP (client vers serveur)access-list 103 permit ospf any any       	\! Autoriser OSPFaccess-list 103 deny icmp any any echo    	\! Bloquer les pings sortantsaccess-list 103 deny ip any any           	\! Bloquer tout autre trafic |
| :---- |

**Application sur l'interface interne :**

| interface GigabitEthernet0/1ip access-group 103 out |
| :---- |

Ces configurations permettent de :

* Le renommage des Routeurs ainsi que la sécurisation des routeurs (mot de passe et authentification).  
* Assurer la redondance avec HSRP au siège.  
* Fournir un service DHCP aux hôtes du siège et des agences pour une gestion simplifiée des adresses IP.  
* Mettre en place des routes statiques pour des chemins de secours entre le siège et les agences.  
* Activer le routage dynamique avec OSPF et un accès à Internet via le routeur du FAI.  
* Configurations des ACLs pour HSRP1 & 2

# 

# **Configurations VLAN et switch** {#configurations-vlan-et-switch}

Le siège, étant le cœur du système d’information et du réseau informatique, est composé de 152 serveurs physiques et virtuels ainsi que de 850 hôtes. 

Selon la consigne, les VLAN doivent être organisés par cœur de métier, ce qui implique une segmentation logique des services et des utilisateurs pour améliorer la gestion du trafic réseau et renforcer la sécurité.

Cependant, la consigne reste vague sur le nombre exact d’hôtes par cœur de métier ou sur la répartition des serveurs entre les VLAN. Cela ouvre deux possibilités pour la configuration du plan d’adressage IP :

- Adressage Classful : 

Chaque VLAN est configuré avec une plage IP standard telle que /24 (254 adresses disponibles). Cette approche est simple à implémenter mais cela sous-entend que tous les services ont un nombre de postes inférieur à 254\.

- Adressage Classless : 

Si les services ou les services ont des besoins différents en termes de nombre d’hôtes, une approche classless peut être pertinente. 

Par exemple :

- Un service nécessitant 400 hôtes peut recevoir un sous-réseau en /23 (510 adresses disponibles).  
- Un service plus petit, avec 50 hôtes, peut être configuré avec un sous-réseau en /26 (62 adresses disponibles).

Une approche mixte mêlant classful et classless selon les besoin de chaque service

**Isolation par les ACL**

Grâce à ces ACLs, les machines des différents VLAN peuvent accéder à Internet tout en étant complètement isolées les unes des autres. Par exemple, un serveur ou une machine virtuelle du VLAN1 sera uniquement accessible aux machines du même VLAN. En revanche, les machines du VLAN2 ne pourront ni interagir avec celles du VLAN1 ni même les détecter.

Cette configuration est particulièrement adaptée à une entreprise spécialisée dans l’audit de sécurité informatique, qui travaille fréquemment sur des laboratoires isolés. L’isolation des VLANs est essentielle pour garantir que les environnements de test ou de simulation restent totalement protégés et n’interfèrent pas avec le reste du réseau. Cela réduit les risques d’accès non autorisé et de propagation accidentelle de menaces.

## 

## **Sécurité** {#sécurité}

Avant d’appliquer des mesures de sécurité sur le switch, nous devons nous assurer que les interfaces vlan soient active (*up*) et pour cela, il suffit d’indiquer l’adresse ip de l’interface, comme ceci : 

| Switch(config)\# interface vlan 1 Switch(config-if)\# ip address 192.168.3.1 255.255.255.0 Switch(config-if)\# no shutdown Switch(config-if)\# exit |
| :---- |

| Switch\# show ip interface brief |
| :---- |

Nous jugeons qu’il s’agit d’une étape importante parce que si les interfaces sont *down*, le technicien n’aura d’autre choix que d'interagir en personne avec le switch en cas de besoin. 

Cette étape corrige cela en permettant à quiconque disposant des accès de se connecter à distance au switch pour procéder à des manipulations. Cependant, pour restreindre au mieux l’accès au switch, il est préférable de désactiver l’interface vlan.

**Mots de passe :** 

Le password apparaît à la couche 7 du modèle OSI. Les mots de passe configurés (par exemple, enable secret) empêchent les utilisateurs non autorisés d'accéder au mode privilégié (Switch\#), où des configurations critiques peuvent être modifiées. 

Un mot de passe bien configuré réduit le risque d'accès accidentel ou malveillant.  
Pour restreindre l’accès au mode privilégié, il convient d’appliquer un *secret* car les mots de passe configurés avec enable secret sont hachés en utilisant l'algorithme MD5, cd dernier étant un hachage unidirectionnel, ce qui signifie qu'il n'est pas directement réversible, offrant une meilleure sécurité.

| Switch(config)\# enable secret 5Witch6\# |
| :---- |

**Sécurisation des lignes VTY**  
Les lignes VTY de 0 à 4 permettent l’accès à distance via Telnet ou SSH; cela concerne la couche 7 Application du modèle OSI. Il est donc primordiale d’imposer une sécurité tel qu’un mot de passe comme le montre le code-block suivant : 

| Switch(config)\# line vty 0 4 Switch(config-line)\# password P@ssP0rt4Me Switch(config-line)\# login Switch(config-line)\# exit Switch(config)\# do wr |
| :---- |

En exigeant un mot de passe pour les lignes VTY, on empêche les utilisateurs non autorisés d'accéder au switch via des sessions distantes. Bien que le mot de passe puisse être attaqué par force brute, son activation ajoute une première barrière contre les attaques simples. Si le protocole SSH est utilisé, les communications entre le client et le switch sont chiffrées, empêchant les attaques de type écoute clandestine (eavesdropping). 

Pour sécuriser davantage les mots de passe dans la configuration du switch, la commande **service password-encryption** a été activée. Sans service password-encryption, les mots de passe apparaissent en texte clair dans la configuration, accessible avec show running-config ou show startup-config. 

Avec cette commande, les mots de passe sont cryptés avec un algorithme de type 7, bien que ce dernier soit faible. Cette mesure réduit la probabilité qu'un administrateur malveillant ou une personne ayant accès au fichier de configuration puisse extraire facilement les mots de passe.  
On peut également apposer un avertissement en suivant cette commande : 

| Switch(config)\# banner motd \# Warning : access restricted to authorized personnel only \! \# |
| :---- |

## 

## **Un VLAN dit “partagé”** {#un-vlan-dit-“partagé”}

Au sein du Siège, nous avons 3 VLAN dont 1 VLAN dit “partagé” contenant un serveur. Ce VLAN a pour but de partager au reste du réseau de l’entreprise des ressources telles que des machines virtuelles.

Avoir un VLAN accessible au reste du réseau de l'entreprise offre des avantages significatifs, notamment en matière de gestion des ressources, de sauvegarde locale, et de respect de la confidentialité des données. Un VLAN dédié permet de centraliser les serveurs critiques pour l'entreprise, tels que les serveurs de fichiers, les bases de données ou les solutions de sauvegarde, tout en garantissant une accessibilité contrôlée depuis les autres VLANs. 

Cette approche est idéale pour les entreprises pratiquant le **self-hosting**, car elle leur permet de conserver leurs données sensibles sur site, sans dépendre de services cloud externes, renforçant ainsi la confidentialité et la souveraineté des données.  
Ce serveur d’illustration a pour adresse IP 192.168.1.3.

# 

# **Bornes Wi-Fi** {#bornes-wi-fi}

Dans notre réseau, un AP-PT a été configuré sur le site de Strasbourg pour offrir un accès Wi-Fi sécurisé aux utilisateurs.

Le répéteur est branché directement sur un switch du réseau de Strasbourg, ce qui lui permet de tirer parti de la connectivité du réseau local (LAN) pour diffuser un signal Wi-Fi. Ce point d'accès utilise le protocole de sécurité WPA2-PSK (Wi-Fi Protected Access 2 avec clé prépartagée) pour garantir une connexion sécurisée. 

**Paramètres de l'AP-PT :**

Connexion physique : L'AP est connecté au switch via un câble Ethernet standard, ce qui correspond à la couche 1 (physique) du modèle OSI.

SSID et Sécurité : Le SSID est configuré pour émettre un réseau Wi-Fi sécurisé avec une authentification WPA2-PSK. La gestion de l'accès et du chiffrement des données se situe à la couche 2 (liaison) et à la couche 7 (application) pour les échanges d'informations.

**Mot de passe** : 

Le mot de passe configuré pour cet AP est “office\_wifi”, qui offre le minimum de sécurité et facilite l'accès pour les utilisateurs autorisés et robustesse face aux intrusions.  
Ce mot de passe  doit être communiqué uniquement aux utilisateurs autorisés pour limiter l'accès au réseau. Cela garantit une couche de sécurité au niveau de l'accès, intégrée aux couches 2 (liaison) et 7 (application).

Couverture réseau : Ce point d'accès permet une extension du réseau filaire pour les terminaux mobiles et autres équipements sans fil au sein du site, touchant principalement les couches 1 (physique) et 2 (liaison).

Nous recommandons également au moins une borne wi-fi par site dédiée aux visiteurs dont le nom serait “*guest\_wifi”*.


**Remarques :** 

Le Wi-Fi 5 (802.11ac) peut gérer jusqu'à 50 connexions simultanées, mais il est recommandé de limiter à 25-30 appareils actifs par borne pour de meilleures performances. 

Le Wi-Fi 6 (802.11ax) supporte jusqu'à 75 connexions simultanées grâce à une gestion de réseau plus efficace, mais il est conseillé de limiter à 40-50 appareils pour un réseau stable

Pour un siège de 850 hôtes, 3 bornes Wi-Fi 5 ou Wi-Fi 6 sont recommandées. Les agences moyennes (500 hôtes) nécessitent 2 bornes, tandis qu'une seule borne suffit pour les petites agences (80-120 hôtes)

En termes de sécurité, il est recommandé d'utiliser des bornes Cisco ou Aruba avec les fonctionnalités suivantes :

* Console web pour le monitoring des activités  
* Restriction d'accès aux sites suspects et contenus illicites  
* Détection et prévention des intrusions (WIDS/WIPS)  
* Segmentation des réseaux (VLANs)  
* Contrôle de la bande passante

[image1]: <data:image/png;base64,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>
