**Fonctionnement IDS (résumé)**
- Se place en “observateur” sur le réseau.
- Détection via :
    - **Signature-based** : reconnaît des motifs connus (attaques connues).
    - **Anomaly-based** : repère des écarts par rapport au comportement normal.
- À chaque détection : **génère une alerte** pour les admins.
- Point important : **l’IDS ne bloque pas** (il **notifie** seulement).

**Analogie**
- **Firewall = garde / portier** (filtre qui entre/sort).
- **IDS = caméras de surveillance** (observe à l’intérieur, repère comportements anormaux).
## **Firewall vs IDS**

|Aspect|Firewall|IDS|
|---|---|---|
|Position|frontière / périmètre réseau|à l’intérieur du réseau|
|Rôle|filtrer autoriser/refuser connexions|surveiller & détecter activités suspectes|
|Moment|avant / pendant l’établissement du trafic|après (trafic déjà passé / activité interne)|
|Action|bloque/autorise|**alerte uniquement** (pas d’action directe)|
|Méthode|règles|signatures + anomalies|

## **Types of IDS**

Deux axes principaux :
1. **Mode de déploiement** (où il est placé)
2. **Mode de détection** (comment il détecte)

### **1) Deployment Modes (déploiement)

|Type|Où ?|Surveille quoi ?|Avantages|Limites|
|---|---|---|---|---|
|**HIDS** (Host-based IDS)|sur **chaque machine**|activité d’un **hôte spécifique**|visibilité **très détaillée** sur le host|lourd en ressources + **difficile à gérer** à grande échelle (install/maintenance partout)|
|**NIDS** (Network-based IDS)|sur le **réseau** (point de monitoring)|trafic de **tous les hôtes**|vue **centralisée** des détections réseau|moins “profond” qu’un HIDS sur un host précis (dépend du point d’écoute)|

![[IDS - NIDS & HIDS.png]]

### **2) Detection Modes (détection)**

|Mode|Principe|Points forts|Points faibles|Zero-day ?|
|---|---|---|---|---|
|**Signature-based**|compare le trafic à des **signatures** (patterns connus)|rapide + efficace sur menaces **connues** (si bonne base)|ne voit pas les attaques **jamais vues**|❌|
|**Anomaly-based**|apprend un **baseline** (comportement normal) et alerte sur **déviation**|peut repérer des **zero-day**|beaucoup de **faux positifs** (normal ≠ toujours stable)|✅|
|**Hybrid**|combine **signatures + anomalies**|équilibre : connu rapide + nouveau détectable|complexité/overhead potentiels|✅|

## **Snort**

- **IDS open-source**
- Détection via **règles** (rule files) : signatures/patterns d’attaques.
- Livré avec des **règles intégrées** (built-in) → couvre beaucoup de trafic malveillant.
- Personnalisation :
    - créer des **règles custom** pour du trafic non couvert
    - **désactiver** des règles built-in si elles ne sont pas pertinentes / génèrent du bruit

|Mode|Ce que Snort fait|Sortie|Quand l’utiliser|
|---|---|---|---|
|**Packet Sniffer**|lit + **affiche** les paquets, **sans analyse IDS**|console / fichier|monitoring simple, troubleshooting perf/flux|
|**Packet Logging**|détection en temps réel + **log** du trafic|**PCAP** (capture complète + détections)|enquête forensique, RCA après incident|
|**NIDS mode**|surveille en temps réel + applique les **règles IDS** (signatures)|**alertes** si match|usage IDS principal : détection proactive|
RCA = root cause analysis

### **Snort usage**

- À l’installation Snort, tu fournis **interface réseau** + **range** (plage IP).
- Par défaut, Snort capture surtout le trafic **destiné à l’hôte**.
- Pour surveiller **tout le réseau**, il faut activer le **promiscuous mode** sur l’interface

---

**fichiers importants**

- Répertoire principal : **`/etc/snort`**
- Fichier clé : **`/etc/snort/snort.conf`**
    - définit le **réseau à monitorer** (ex: `$HOME_NET`)
    - active/désactive des **rule files**
    - autres réglages
- Règles : **`/etc/snort/rules/`** (dont **`local.rules`** pour tes règles custom)

---

**format d'une règle snort**

règle = en-tête + metadonnées

![[IDS - format règle snort.png]]

| Champ                     | Rôle                            | Exemple dans le texte                          |
| ------------------------- | ------------------------------- | ---------------------------------------------- |
| **Action**                | quoi faire si match             | `alert`                                        |
| **Protocol**              | protocole visé                  | `icmp`                                         |
| **Source IP / Port**      | origine du trafic               | `any any`                                      |
| **Direction**             | sens du flux                    | `->`                                           |
| **Destination IP / Port** | cible du trafic                 | `$HOME_NET any` ou `127.0.0.1 any`             |
| **msg**                   | message de l’alerte             | `"Ping Detected"` / `"Loopback Ping Detected"` |
| **sid**                   | identifiant unique de signature | `10003` (doit être unique)                     |
| **rev**                   | version de la règle             | `1` (incrémenter si modif)                     |
**Exemple de règle (loopback ping)**

- Détecte des **ICMP** vers **127.0.0.1** → alerte “Loopback Ping Detected”.

---

### **création d'un règle custom (workflow)**

1. Éditer le fichier :
- `sudo nano /etc/snort/rules/local.rules`

2. Ajouter la règle (sans supprimer les autres) :
- `alert icmp any any -> 127.0.0.1 any (msg:"Loopback Ping Detected"; sid:10003; rev:1;)`

3. Sauver : `Ctrl+X` puis `Y`


