## Sender Policy Framework (SPF)

- Mécanisme d’**authentification email**
- Permet de vérifier que le serveur expéditeur est **autorisé par le domaine**
- Basé sur un **enregistrement DNS TXT** contenant :
    - IP autorisées
    - Domaines autorisés

### ⚙️ Fonctionnement

1. Un email est envoyé
2. Le serveur destinataire :
    - récupère le **SPF dans le DNS**
    - vérifie si l’IP expéditrice est autorisée
3. Action selon le résultat

![[phishing prevention SPF framework.png]]
### 📊 Résultats SPF & Actions

| Résultat SPF          | Action du serveur               |
| --------------------- | ------------------------------- |
| Pass / Neutral / None | ✅ Accepter                      |
| SoftFail / PermError  | ⚠️ Marquer comme suspect (flag) |
| Fail / TempError      | ❌ Rejeter                       |

### 🧩 Structure d’un SPF record

Exemple :

v=spf1 ip4:127.0.0.1 include:_spf.google.com -all

|Élément|Signification|
|---|---|
|v=spf1|Début du SPF|
|ip4:127.0.0.1|IP autorisée|
|include:_spf.google.com|Domaine autorisé|
|-all|Refus de tout le reste|

Further information on SPF Record Syntax can be found here : https://dmarcian.com/spf-syntax-table/

**🧠 Points importants**

- `include` → autorise **toutes les IP d’un domaine externe**
- Pas besoin de voir les IP directement → elles peuvent être **héritées via les includes**
- SPF **ne bloque pas toujours** → peut juste signaler (SoftFail)

### 🛠️ Outils utiles

- **[SPF Surveyor](https://dmarcian.com/spf-survey/)** (dmarcian)** → visualisation des records
- **Google Admin Toolbox ([Messageheader](https://toolbox.googleapps.com/apps/messageheader/))** → analyse des headers email

### 🔍 Exemple concret

- SPF = **SoftFail**
- Signification :
    - serveur **non autorisé**
    - email **accepté mais suspect**
    - IP parfois **non identifiable**

![[phishing prevention SPF softfail.png]]

---

## DomainKeys Identified Mail (DKIM)

- Méthode d’**authentification email**
- Utilise une **signature cryptographique**
- Complémentaire à SPF (et utilisé par DMARC)
- ✅ **Avantage clé** : fonctionne même après transfert (forwarding)

### ⚙️ Fonctionnement

1. Serveur expéditeur :
    - signe l’email avec une **clé privée**
2. Serveur destinataire :
    - récupère la **clé publique (DNS)**
    - vérifie la signature
3. Résultat :
    - signature valide → email authentique
    - sinon → suspect ou rejeté

![[phishing prevention DKIM framework.png]]

**🔐 Principe clé**

- **Clé privée** → côté expéditeur (signature)
- **Clé publique** → dans le DNS (vérification)

### 🧩 Structure d’un DKIM record

Exemple :

v=DKIM1; k=rsa; p=<public_key>

|Élément|Signification|
|---|---|
|v=DKIM1|Version DKIM|
|k=rsa|Type de clé (RSA)|
|p=|Clé publique|

### 📊 Résultats DKIM (interprétation)

|Résultat|Signification|
|---|---|
|Pass|✅ Signature valide|
|Fail / PermError|❌ Signature invalide ou erreur|
|TempError|⚠️ Erreur temporaire|

### 🧠 Causes d’échec (ex : `permerror`)

- Signature invalide
- Clé publique absente ou incorrecte (DNS)
- Modification du message (ex : forwarding mal géré)
- Mauvaise configuration DKIM

![[phishing prevention DKIM permerror.png]]

- `dkim=permerror`
    - ❌ Pas de clé trouvée
    - → email marqué comme spam ou rejeté

### 🛠️ Outils utiles

- **DKIM Record Checker (dmarcian)** [DKIM Record Checker](https://dmarcian.com/dkim-inspector/)
- **DKIM Validator** [Validator(opens in new tab)](https://dmarcian.com/dkim-validator/)

---

## Domain-based Message Authentication, Reporting and Conformance (DMARC)

- Standard d’**authentification email avancé**
- Combine :
    - **SPF** (serveur autorisé)
    - **DKIM** (signature)
- Introduit la notion d’**alignment (alignement des domaines)**

👉 Vérifie que le **domaine visible (From)** correspond à :

- celui validé par SPF
- celui validé par DKIM

### ⚙️ Fonctionnement

1. Vérifie SPF + DKIM
2. Vérifie l’**alignement des domaines**
3. Applique une **politique (policy)** si échec

### 🧩 Structure d’un DMARC record

Exemple :

v=DMARC1; p=quarantine; rua=mailto:postmaster@website.com

|Élément|Signification|
|---|---|
|v=DMARC1|Version (obligatoire)|
|p=quarantine|Politique appliquée|
|rua=mailto:...|Email de rapport|

rua :  An optional tag. In this case, aggregate reports will be sent to the email specified

### 📊 Politiques DMARC

|Policy|Action|
|---|---|
|none|👀 Surveillance uniquement|
|quarantine|📩 Mettre en spam|
|reject|❌ Rejeter totalement|

### 🧠 Points clés

- DMARC = **couche de contrôle finale**
- Repose sur :
    - SPF ✅
    - DKIM ✅
    - Alignment ✅
- Permet aussi :
    - 📊 **rapports d’analyse** (rua)

### 🔍 Exemple concret

- Domaine (ex : microsoft.com)
    - SPF ✅
    - DKIM ✅
    - DMARC ✅
- Politique : `p=reject`  
    → ❌ tout email non conforme est rejeté

![[phishing prevention DMARC microsoft exemple.png]]

### 🛠️ Outils utiles

- **Domain Checker (dmarcian)**  
    → vérifie SPF, DKIM, DMARC en même temps

https://dmarcian.com/domain-checker/

---
## ⚖️ Récap global (SPF, DKIM, DMARC)

|Technologie|Rôle principal|Limite|
|---|---|---|
|SPF|Vérifie serveur (IP)|Forwarding ❌|
|DKIM|Vérifie intégrité (signature)|Config complexe|
|DMARC|Vérifie alignement + policy|Dépend des 2 autres|

---

## Secure/Multiperpose Internet Mail Extensions (S/MIME)

- Standard pour **emails signés et chiffrés**
- Basé sur la **cryptographie à clé publique**
    - 🔐 clé privée (secrète)
    - 🔓 clé publique (partagée)

### 🔐 1. Signature numérique (Digital Signature)

**⚙️ Fonctionnement**

- L’expéditeur signe avec sa **clé privée**
- Le destinataire vérifie avec la **clé publique**

**🎯 Garanties**

|Propriété|Rôle|
|---|---|
|Authentification|Vérifie l’identité|
|Non-répudiation|Impossible de nier|
|Intégrité|Détecte modification|

### 🔒 2. Chiffrement (Encryption)

**⚙️ Fonctionnement**

- Expéditeur chiffre avec la **clé publique du destinataire**
- Destinataire déchiffre avec sa **clé privée**

**🎯 Garantie**

|Propriété|Rôle|
|---|---|
|Confidentialité|Message lisible uniquement par le destinataire|

### 🔁 Workflow simplifié

|Étape|Action|
|---|---|
|1|Bob signe avec sa clé privée|
|2|Bob chiffre avec clé publique de Mary|
|3|Mary vérifie signature (clé publique Bob)|
|4|Mary déchiffre (clé privée Mary)|

### 🧠 Points clés

- Combine :
    - ✍️ **Signature** (authenticité + intégrité)
    - 🔐 **Chiffrement** (confidentialité)
- Utilise des **certificats numériques**
- Communication sécurisée **de bout en bout**

### ⚖️ Positionnement vs SPF/DKIM/DMARC

|Technologie|Rôle|
|---|---|
|SPF/DKIM/DMARC|Vérifient l’email (anti-spoofing)|
|S/MIME|Sécurise le **contenu** de l’email|

---

## Lab thm : https://tryhackme.com/room/phishingemails4gkxh

### Analysing SMTP Responses

**Ressource :** Display Filter Reference: Simple Mail Transfer Protocol https://www.wireshark.org/docs/dfref/s/smtp.html

![[smtp analysis traffic.pcap.png]]


![[smtp analysis code 220.png]]


![[smtp analysis spamhaus.org.png]]


![[smtp analysis code 552.png]]

---

### Inspecting Emails and Attachments

**Ressource** : Display Filter Reference: Internet Message Format https://www.wireshark.org/docs/dfref/i/imf.html


---

## How Organisation Stop Phishing

👉 2 axes principaux :

- 🔧 **Défenses techniques**
- 👤 **Utilisateurs (outils + formation)**

### 🔧 Défenses techniques

|Mécanisme|Rôle|
|---|---|
|Email Filtering|Bloque selon réputation IP/domaine|
|Secure Email Gateways (SEG)|Détecte spoofing, usurpation|
|Link Rewriting|Remplace les liens suspects (scan)|
|Sandboxing|Teste fichiers/liens en environnement isolé|
[**Email Filtering**(opens in new tab)](https://www.spamhaus.org/resource-hub/ip-domain-reputation/)
[**Secure Email Gateways**](https://www.cloudflare.com/learning/email-security/secure-email-gateway-seg/)
[**Link Rewriting**(opens in new tab)](https://learn.microsoft.com/en-us/defender-office-365/safe-links-about)
[**Sandboxing**](https://learn.microsoft.com/en-us/defender-office-365/safe-attachments-about)

### 👤 Outils & formation utilisateur

|Élément|Rôle|
|---|---|
|Indicateurs visuels|Alertes : “External”, “Suspicious”|
|Signalement phishing|Permet de reporter facilement|
|Formation utilisateurs|Apprentissage des attaques|
|Simulations phishing|Tests réels pour entraîner|

### 🧠 Points clés

- Même avec protection technique → **risque ≠ 0**
- L’utilisateur est la **dernière ligne de défense**
- Approche efficace = **tech + humain**

---

