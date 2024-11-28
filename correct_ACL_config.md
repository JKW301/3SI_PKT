# ACL sur HSRPs
Ces commandes sont à taper **uniquement sur le routeur HSRP2**.<br>
Le HSRP1 étant déjà configuré pour ce qui est des ACL

## Identifier la config en cours
Ouvrir le CLI et taper :
- `enable`
- `show access-lists`
- `sh run`

Cela permettra d'identifier les groupes ACL qui ont cours et les interfaces qui sont conifguré avec ces groupes.

## Supprimer les groupes ACL
Appliquer les nouveaux groupes ACL sur les interfaces quand les anciens groupes sont encore appliqués créera des conflits au sein de la macronie.

Pour retirer le groupe ACL d'une interface, interface par interface : 
```
en
conf t
interface se0/0/0
no ip access-group 101 in
no ip access-group 102 out
```
Pour supprimer carrément le groupe du routeur et donc, de toutes les interfaces auxquelles elle était appliquée : 
```
en
conf t
no access-list 101
no access-list 102
```


## Comment corriger l'ACL

### Créer les nouveaux groupes ACL

**Groupe ACL pour l'interface vers le RFAI**
```
ip access-list extended OUTBOUND_TO_FAI
  permit tcp any any eq www
  permit tcp any any eq 443
  permit udp any any eq domain
  permit icmp any any unreachable
  deny icmp any 10.0.0.0 0.255.255.255
  deny icmp any 192.168.0.0 0.0.255.255
  deny icmp any 172.16.0.0 0.15.255.255
  deny icmp any 10.255.0.0 0.0.0.255
  permit ip any any

```
**Groupe ACL pour les interfaces vers le réseau interne**

```
ip access-list extended INBOUND_FROM_INTERNAL
  permit tcp any any eq www
  permit tcp any any eq 443
  permit udp any any eq domain
  permit icmp any any
  permit ip any any
```

### Appliquer les groupes ACL aux interfaces
**L'interface côté RFAI**
```
interface Serial0/0/0
 ip access-group OUTBOUND_TO_FAI in
 ip access-group OUTBOUND_TO_FAI out
```

**Les interfaces côté réseau interne**
```
interface Serial0/0/1
 ip access-group INBOUND_FROM_INTERNAL in
 ip access-group INBOUND_FROM_INTERNAL out
 
interface Serial0/1/0
 ip access-group INBOUND_FROM_INTERNAL in
 ip access-group INBOUND_FROM_INTERNAL out 

interface Serial0/1/1
 ip access-group INBOUND_FROM_INTERNAL in
 ip access-group INBOUND_FROM_INTERNAL out
 
interface Serial0/2/0
 ip access-group INBOUND_FROM_INTERNAL in
 ip access-group INBOUND_FROM_INTERNAL out
 
```
