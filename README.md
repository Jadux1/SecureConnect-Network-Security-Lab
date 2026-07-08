# SecureConnect — Réseau sécurisé d’une petite entreprise

## 1. Présentation du projet

**SecureConnect** est un projet personnel réalisé avec **Cisco Packet Tracer**.

L’objectif de ce projet est de concevoir et sécuriser le réseau d’une petite entreprise fictive.  
Le réseau est organisé en plusieurs VLAN afin de séparer les services : Direction, RH, Employés, Invités, Serveurs, DMZ et Administration.

Le projet met en pratique plusieurs notions importantes en réseau :

- VLAN
- Routage inter-VLAN
- DHCP
- DNS
- Serveur web en DMZ
- ACL de sécurité
- SSH
- Port-Security
- Désactivation des ports inutilisés

---

## 2. Objectifs du projet

Les objectifs de ce projet sont :

- créer une topologie réseau professionnelle ;
- séparer les différents services avec des VLAN ;
- permettre la communication entre les VLAN grâce au routage inter-VLAN ;
- distribuer automatiquement les adresses IP avec DHCP ;
- configurer un serveur DNS interne ;
- placer un serveur web dans une DMZ ;
- sécuriser les communications avec des ACL ;
- autoriser l’administration réseau uniquement depuis un PC Admin ;
- sécuriser les ports des switchs avec Port-Security ;
- documenter le projet pour GitHub et LinkedIn.

---

## 3. Topologie finale

La topologie finale contient :

- 1 routeur : `R1-SecureConnect`
- 2 switchs : `SW1-Core` et `SW2-Access`
- 5 PC :
  - `PC-Direction`
  - `PC-RH`
  - `PC-Employes`
  - `PC-Invite`
  - `PC-Admin`
- 1 serveur DHCP/DNS : `SRV-DHCP-DNS`
- 1 serveur web en DMZ : `SRV-WEB-DMZ`

![Topologie finale](captures/Topologie%20finale.png)

---

## 4. Plan des VLAN

| VLAN | Nom | Réseau | Rôle |
|---:|---|---|---|
| 10 | DIRECTION | 192.168.10.0/24 | Réseau de la direction |
| 20 | RH | 192.168.20.0/24 | Réseau ressources humaines |
| 30 | EMPLOYES | 192.168.30.0/24 | Réseau des employés |
| 40 | INVITES | 192.168.40.0/24 | Réseau invités |
| 50 | SERVEURS | 192.168.50.0/24 | Serveur DHCP/DNS |
| 60 | DMZ_WEB | 192.168.60.0/24 | Serveur web en DMZ |
| 99 | ADMIN | 192.168.99.0/24 | Administration réseau |
| 999 | UNUSED_PORTS | Aucun usage utilisateur | Ports inutilisés |

---

## 5. Plan d’adressage IP

| Équipement | VLAN | Adresse IP | Rôle |
|---|---:|---|---|
| R1-SecureConnect | 10 | 192.168.10.1 | Passerelle Direction |
| R1-SecureConnect | 20 | 192.168.20.1 | Passerelle RH |
| R1-SecureConnect | 30 | 192.168.30.1 | Passerelle Employés |
| R1-SecureConnect | 40 | 192.168.40.1 | Passerelle Invités |
| R1-SecureConnect | 50 | 192.168.50.1 | Passerelle Serveurs |
| R1-SecureConnect | 60 | 192.168.60.1 | Passerelle DMZ |
| R1-SecureConnect | 99 | 192.168.99.1 | Passerelle Admin |
| SW1-Core | 99 | 192.168.99.2 | Administration SSH |
| SW2-Access | 99 | 192.168.99.3 | Administration SSH |
| SRV-DHCP-DNS | 50 | 192.168.50.10 | Serveur DHCP/DNS |
| SRV-WEB-DMZ | 60 | 192.168.60.10 | Serveur web DMZ |
| PC-Admin | 99 | 192.168.99.10 | Poste d’administration |

Les autres PC reçoivent leur adresse IP automatiquement grâce au DHCP.

---

## 6. Services configurés

### DHCP

Le serveur `SRV-DHCP-DNS` distribue automatiquement les adresses IP aux VLAN suivants :

| VLAN | Pool DHCP | Adresse de départ |
|---:|---|---|
| 10 | DIRECTION | 192.168.10.100 |
| 20 | RH | 192.168.20.100 |
| 30 | EMPLOYES | 192.168.30.100 |
| 40 | INVITES | 192.168.40.100 |

Le routeur utilise la commande `ip helper-address` pour transférer les requêtes DHCP vers le serveur `192.168.50.10`.

### DNS

Le serveur DNS interne permet de résoudre le nom suivant :

| Nom DNS | Adresse IP |
|---|---:|
| intranet.secureconnect.local | 192.168.60.10 |

### HTTP

Le serveur web est placé dans la DMZ.

| Serveur | Adresse IP | Service |
|---|---:|---|
| SRV-WEB-DMZ | 192.168.60.10 | HTTP |

![Page web DMZ depuis PC-Invite](captures/pc%20invite%20web-dmz-ok.png)

---

## 7. Règles de sécurité ACL

Des ACL ont été configurées sur le routeur pour contrôler les communications entre les VLAN.

### VLAN Invités

Le réseau Invités est fortement limité.

Les invités peuvent :

- utiliser le DNS interne ;
- accéder au serveur web en DMZ avec HTTP.

Les invités ne peuvent pas :

- accéder au réseau Direction ;
- accéder au réseau RH ;
- accéder au réseau Employés ;
- accéder directement au serveur DHCP/DNS ;
- accéder au VLAN Administration ;
- faire un ping vers le serveur web DMZ.

| Test depuis PC-Invite | Résultat |
|---|---|
| Ping vers Direction | Bloqué |
| Ping vers RH | Bloqué |
| Ping vers Employés | Bloqué |
| Ping vers serveur DHCP/DNS | Bloqué |
| Ping vers serveur web DMZ | Bloqué |
| Ping vers VLAN Admin | Bloqué |
| Accès HTTP à `intranet.secureconnect.local` | Autorisé |

![Tests PC-Invite - ACL bloquées 1](captures/pc%20invite%20acl%20bloque%201.png)

![Tests PC-Invite - ACL bloquées 2](captures/pc%20invite%20acl%20bloque%202.png)

![Page web DMZ depuis PC-Invite](captures/pc%20invite%20web-dmz-ok.png)

### VLAN Employés

Le réseau Employés a des droits plus larges que le réseau Invités, mais il reste limité.

Les employés peuvent :

- accéder au serveur DHCP/DNS ;
- accéder au serveur web en DMZ ;
- accéder au réseau Direction.

Les employés ne peuvent pas :

- accéder au réseau RH ;
- accéder au VLAN Administration.

| Test depuis PC-Employes | Résultat |
|---|---|
| Ping vers RH | Bloqué |
| Ping vers VLAN Admin | Bloqué |
| Ping vers serveur DHCP/DNS | Autorisé |
| Ping vers serveur web DMZ | Autorisé |
| Ping vers Direction | Autorisé |

![Tests PC-Employes 1](captures/pc%20employes%20tests%201.png)

![Tests PC-Employes 2](captures/pc%20employes%20tests%202.png)

---

## 8. Administration sécurisée avec SSH

L’administration des équipements réseau se fait avec **SSH**.

Seul `PC-Admin`, situé dans le VLAN 99, peut administrer les équipements réseau.

| Équipement | Adresse IP d’administration |
|---|---:|
| R1-SecureConnect | 192.168.99.1 |
| SW1-Core | 192.168.99.2 |
| SW2-Access | 192.168.99.3 |

La sécurité SSH repose sur :

- un utilisateur local `admin` ;
- un mot de passe configuré uniquement pour le lab ;
- SSH version 2 ;
- une ACL qui autorise uniquement `PC-Admin`.

![SSH depuis PC-Admin](captures/pc%20admin%20ssh%20ok.png)

---

## 9. Sécurisation des switchs

Les switchs ont été sécurisés avec deux mesures principales.

### Port-Security

Les ports utilisés par les PC et les serveurs acceptent une seule adresse MAC.

Le mode `sticky` permet au switch d’apprendre automatiquement l’adresse MAC de l’équipement connecté.

### Ports inutilisés

Les ports inutilisés sont :

- placés dans le VLAN 999 `UNUSED_PORTS` ;
- désactivés avec la commande `shutdown`.

Cela permet de réduire le risque qu’un appareil non autorisé soit branché sur le réseau.

![Port Security SW1](captures/sw1%20port%20security.png)

![Port Security SW2](captures/sw2%20port%20security.png)

---

## 10. Vérifications de configuration

### VLAN sur SW1-Core

![VLAN SW1](captures/SW1%20vlan%20brief.png)

### VLAN sur SW2-Access

![VLAN SW2](captures/SW2%20vlan%20brief.png)

### ACL sur le routeur

![ACL sur R1](captures/R1%20access%20lists.png)

Les commandes suivantes ont été utilisées pour vérifier le bon fonctionnement du projet :

```bash
show vlan brief
show interfaces trunk
show ip interface brief
show access-lists
show port-security
show port-security address
```

Ces commandes permettent de vérifier :

- la création des VLAN ;
- l’affectation des ports aux VLAN ;
- le fonctionnement des trunks ;
- les interfaces du routeur ;
- les règles ACL ;
- les adresses MAC apprises par Port-Security.

---

## 11. Résultats des tests

Les tests réalisés montrent que :

- les PC reçoivent bien leur adresse IP avec DHCP ;
- le DNS résout correctement `intranet.secureconnect.local` ;
- le serveur web DMZ est accessible en HTTP ;
- le réseau Invités est isolé des réseaux internes ;
- le réseau Employés ne peut pas accéder au réseau RH ni au VLAN Admin ;
- le PC Admin peut administrer le routeur et les switchs avec SSH ;
- les ports inutilisés sont désactivés ;
- Port-Security fonctionne sur les ports utilisés.

---

## 12. Compétences développées

Ce projet m’a permis de pratiquer :

- la conception d’un réseau d’entreprise ;
- l’adressage IPv4 ;
- la configuration de VLAN ;
- la configuration de trunks ;
- le routage inter-VLAN ;
- DHCP ;
- DNS ;
- la mise en place d’une DMZ ;
- les ACL ;
- SSH ;
- Port-Security ;
- les tests réseau ;
- la documentation technique.

---

## 13. Structure du projet

```text
SecureConnect/
├── packet-tracer/
│   └── SecureConnect_Final.pkt
├── documentation/
│   └── README.md
└── captures/
    ├── Topologie finale.png
    ├── pc invite acl bloque 1.png
    ├── pc invite acl bloque 2.png
    ├── pc invite web-dmz-ok.png
    ├── pc employes tests 1.png
    ├── pc employes tests 2.png
    ├── pc admin ssh ok.png
    ├── R1 access lists.png
    ├── SW1 vlan brief.png
    ├── SW2 vlan brief.png
    ├── sw1 port security.png
    └── sw2 port security.png
```

---

## 14. Conclusion

Ce projet m’a permis de comprendre comment concevoir et sécuriser un réseau d’entreprise avec Cisco Packet Tracer.

La segmentation avec VLAN permet de séparer les différents services.  
Les ACL permettent de contrôler les communications entre les réseaux.  
La DMZ permet de publier un serveur web sans exposer directement les réseaux internes.  
SSH permet une administration plus sécurisée des équipements réseau.  
Port-Security et la désactivation des ports inutilisés renforcent la sécurité au niveau des switchs.

Ce projet constitue une base solide que je pourrai améliorer plus tard avec d’autres fonctionnalités comme NAT, pare-feu, redondance, supervision ou analyse de trafic.
