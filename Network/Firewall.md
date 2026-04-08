- **Firewall** : système qui **filtre le trafic réseau** (in/out) selon un **ensemble de règles**.
- Tout trafic **passe d’abord par le firewall**, qui **autorise** ou **bloque** en fonction des règles.
## **Types of Firewalls**

Il existe **plusieurs types**, avec des rôles différents selon les **couches OSI** où ils opèrent.

![[Firewalls types.png]]

| Type                            | Couches OSI | Principe                                                                   | Points forts                                                                                             | Limites                                                                                    |
| ------------------------------- | ----------: | -------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| **Stateless**                   |         3–4 | Filtre **chaque paquet** selon des règles, **sans mémoire** des connexions | Très **rapide** (adapté aux réseaux à haut débit)                                                        | Pas de décisions “intelligentes” basées sur l’historique (politiques complexes difficiles) |
| **Stateful**                    |         3–4 | Filtrage + **suivi des connexions** via une **state table**                | Plus **sécurisé** : décisions basées sur l’état (connexion déjà acceptée/refusée)                        | Plus “lourd” que stateless (gestion d’état)                                                |
| **Proxy (Application gateway)** |           7 | Intermédiaire : inspecte **le contenu** (applicatif), relaie les requêtes  | **Inspection L7**, **filtrage de contenu**, **contrôle applicatif**, masque l’IP interne (anonymisation) | Ajoute de la latence/complexité (car intermédiaire)                                        |
| **NGFW**                        |      3 –> 7 | Firewall avancé : **deep packet inspection** + fonctions de sécu intégrées | **IPS**, analyse **heuristique/anomalies**, **déchiffrement SSL/TLS**, corrélation avec **threat intel** | Plus coûteux/complexe à déployer et à gérer                                                |

### À retenir (ultra synthèse)

- **Stateless = rapide mais “sans contexte”** (paquet par paquet).
- **Stateful = contexte via state table** (connexion reconnue).
- **Proxy = inspection applicative (L7) + contenu + IP masking**.
- **NGFW = “tout-en-un” (L3→L7) + DPI + IPS + TLS decrypt + threat intel**.

## **Rules in Firewalls**

### **Composants d'un règle firewall**

|Champ|Rôle|
|---|---|
|**Source address**|IP qui **initie** le trafic|
|**Destination address**|IP qui **reçoit**|
|**Port**|Port concerné (ex: 80, 22, 25)|
|**Protocol**|TCP / UDP / etc.|
|**Action**|Que faire (Allow / Deny / Forward)|
|**Direction**|Sens : Inbound / Outbound / Forward|

---
### **Actions principales**

|Action|Effet|Exemple typique|
|---|---|---|
|**Allow**|Autorise le trafic|Autoriser HTTP sortant (TCP/80)|
|**Deny**|Bloque le trafic|Bloquer SSH entrant (TCP/22)|
|**Forward**|Redirige le trafic vers un autre segment/hôte (firewall = gateway/routing)|Forward HTTP entrant vers serveur web interne|

---
### **Direction des règles**

|Type|S’applique à…|Exemple|
|---|---|---|
|**Inbound**|Trafic **entrant**|Autoriser HTTP entrant vers un webserver|
|**Outbound**|Trafic **sortant**|Bloquer SMTP sortant (25) sauf mail server|
|**Forward**|Trafic **transité/relayé** (routé)|Rediriger HTTP entrant vers 192.168.1.8|

## **Linux iptables Firewall**

- Linux a un firewall natif via **Netfilter** (framework noyau) :
    - **packet filtering**
    - **NAT**
    - **connection tracking**
- Plusieurs outils “front-end” pilotent Netfilter : **iptables**, **nftables**, **firewalld**, **ufw**.

### **Outils basés sur Netfilter**

|Outil|Idée clé|
|---|---|
|**iptables**|classique, très répandu|
|**nftables**|successeur d’iptables (plus moderne)|
|**firewalld**|règles prédéfinies + notion de **zones**|
|**ufw**|interface **simplifiée** (débutant-friendly) qui configure derrière (iptables/nft selon système)|

### **UFW : commandes essentielles**

|Objectif|Commande|Résultat attendu|
|---|---|---|
|Voir statut|`sudo ufw status`|ex: `Status: inactive`|
|Activer|`sudo ufw enable`|actif + au démarrage|
|Désactiver|`sudo ufw disable`|firewall off|
|Politique par défaut sortante|`sudo ufw default allow outgoing`|autorise tout sortant sauf règle contraire|
|Bloquer SSH entrant|`sudo ufw deny 22/tcp`|ajoute règle (IPv4 + IPv6)|
|Lister règles numérotées|`sudo ufw status numbered`|affiche règles avec numéros|
|Supprimer une règle|`sudo ufw delete <num>`|supprime la règle ciblée|

### **note importante**

- Pour gérer l’entrant vs sortant : logique “par défaut” + exceptions.
- Choix de l’outil = dépend de tes besoins + habitude (simplicité vs contrôle fin).

