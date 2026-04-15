## Identifying Artifacts
### 🎯 Objectif

Collecter des **artifacts clés** pour :

- vérifier la légitimité
- identifier une menace
- préparer analyses avancées (reputation, threat intel, comportement)

### 📧 Header Artifacts (en-têtes)

|Élément|Question à se poser|Pourquoi c’est important|
|---|---|---|
|Sender email|D’où vient le mail ?|Détecter spoofing / domaine suspect|
|Sender IP|Quelle est l’IP source ?|Vérifier réputation / géolocalisation|
|Subject|Urgence / action demandée ?|Indice classique de phishing|
|Recipient|Qui reçoit (To/CC/BCC) ?|Ciblé ou en masse|
|Reply-To|Où vont les réponses ?|Peut différer → redirection malveillante|
|Date/Time|Quand envoyé ?|Horaires suspects / incohérents|

### 📨 Body Artifacts (contenu)

|Élément|Action|Pourquoi|
|---|---|---|
|URLs / liens|Développer liens raccourcis|Détecter redirection malveillante|
|Attachments (noms)|Vérifier noms / extensions|Fichiers suspects (ex: .exe, .zip)|
|Attachment hash|Générer hash|Lookup dans threat intel|

### ⚠️ Points clés à retenir

- Toujours commencer par **les headers → puis le body**
- Chercher :
    - incohérences
    - urgences artificielles
    - éléments détournés (Reply-To, liens)
- Les artifacts = **base de toute l’analyse**

---

## Email Header Analysis

### 🎯 Objectif

Automatiser et faciliter l’analyse des **email headers** + vérifier la **réputation IP / URL**

### 📧 Email Header Analysis

### 🔑 Idée clé

- Certaines infos visibles directement (Gmail, Yahoo)
- D’autres uniquement dans les **headers complets** :
    - IP source
    - Reply-To
    - routing

### 🛠️ Outils d’analyse de headers

|Outil|Fonction|Avantage|
|---|---|---|
|Google Admin Toolbox Messageheader|Analyse header|Extraction rapide (IP, routing, erreurs)|
|Microsoft Message Header Analyzer|Analyse header|Alternative fiable|

### 🌐 IP & URL Reputation Analysis

**🔑 Objectif**

- Identifier origine
- Vérifier si **malveillant ou légitime**

### 🛠️ Outils de réputation

| Outil                                                                                                                       | Type        | Utilité principale                         |
| --------------------------------------------------------------------------------------------------------------------------- | ----------- | ------------------------------------------ |
| IPinfo [IPinfo(opens in new tab)](https://ipinfo.io/)                                                                       | IP          | Localisation + organisation                |
| URLScan.io [URLScan.io](https://urlscan.io/)                                                                                | URL         | Analyse site sans y accéder (sandbox)      |
| Cisco Talos IP & Domain Reputation Center [IP & Domain Reputation Center](https://talosintelligence.com/reputation_center/) | IP / Domain | Vérifie réputation + activité malveillante |

- [Messageheader](https://toolbox.googleapps.com/apps/messageheader/analyzeheader) helps analyze email headers. By simply pasting the full header into the tool, you can quickly extract key details such as the sender’s IP address, routing path, and potential misconfigurations.
- [Message Header Analyzer](https://mha.azurewebsites.net/) idem

### ⚠️ Points clés

- Toujours :
    - extraire les **headers complets**
    - analyser IP + URL
- Chercher :
    - incohérences géographiques
    - IP inconnues ou suspectes
    - domaines mal notés

---

## Email Body Analysis

### 🎯 Objectif

Analyser le **contenu du mail (body)** pour détecter :

- liens de phishing
- pièces jointes malveillantes

### 📨 Analyse du Body

**🔑 Idée clé**

👉 C’est ici que se trouve **le payload (attaque réelle)** :

- lien malveillant
- pièce jointe infectée

### 🔗 Analyse des liens (URLs)

|Méthode|Action|Avantage|
|---|---|---|
|Copier le lien|Clic droit → copier URL|Vérification sans risque|
|Analyse HTML|Inspecter code source|Détecter liens cachés|
|Outils d’extraction|Parser contenu brut|Trouver URLs obfusquées|

![[phishing analysis tools url copy.png]]

**🛠️ Outils**

| Outil                                                                             | Utilité                          |
| --------------------------------------------------------------------------------- | -------------------------------- |
| CyberChef                                                                         | Extraction + analyse de données  |
| URL Extractor [URL extraction tool](https://www.convertcsv.com/url-extractor.htm) | Extraction automatique des liens |

### 📎 Analyse des pièces jointes

**⚠️ Règle critique**

❌ Ne jamais ouvrir directement  
✅ Utiliser :

- VM
- sandbox

### 🔐 Hashing

Créer une **empreinte (hash SHA256)** pour analyse

```bash
user@tryhackme$ sha256sum shady_attachment.pdf 025ba9ce4a2118a9ca7b115c8869ff73bc16bad3732ba359cef1e60ad8f961f9 shady_attachment.pdf
```

👉 Sert à :

- vérifier réputation
- identifier malware connu

### 🛠️ Outils d’analyse

|Outil|Type|Utilité|
|---|---|---|
|Cisco Talos IP & Domain Reputation Center|Hash / IP / Domain|Détection activité malveillante|
|VirusTotal|Multi-analyse|Vérification via plusieurs antivirus|

### ⚠️ Points clés

- Toujours :
    - analyser **liens sans cliquer**
    - extraire **toutes les URLs**
- Pour pièces jointes :
    - environnement contrôlé obligatoire
    - générer un **hash**
- Utiliser plusieurs outils pour confirmer

---

## Malware Sandbox

### 🎯 Objectif

Analyser un fichier suspect **sans risque** grâce aux **malware sandboxes**

### 🧪 Malware Sandboxes

👉 Environnement contrôlé pour :

- exécuter fichiers suspects
- observer comportement
- détecter **IOCs (Indicators of Compromise)**

**📊 Ce qu’on peut voir**

- connexions réseau (URLs contactées)
- payloads téléchargés
- modifications système
- comportement global du malware

### 🛠️ Outils principaux

| Outil           | Type                | Spécificité                             |
| --------------- | ------------------- | --------------------------------------- |
| ANY.RUN         | Sandbox interactive | Analyse en temps réel + interaction     |
| Hybrid Analysis | Sandbox             | Rapports détaillés (comportement + IOC) |
| JOESandbox      | Sandbox avancée     | Analyse statique + dynamique            |

- [ANY.RUN](https://app.any.run/)
- [Hybrid Analysis](https://hybrid-analysis.com/)
- [JOESandbox](https://www.joesandbox.com/)

### ⚠️ Points clés

- Sandbox = **sécurité + visibilité**
- Permet de :
    - comprendre le comportement
    - identifier rapidement des IOCs
- Différence importante :
    - **interactive (ANY.RUN)** → exploration manuelle
    - **automatisée (Hybrid / Joe)** → rapports détaillés

---

## Using PhishTool

### 🎯 Objectif

Utiliser **PhishTool** ([PhishTool](https://www.phishtool.com/)) pour :

- automatiser l’analyse phishing
- centraliser les informations
- accélérer les investigations SOC

### 🛠️ PhishTool – Vue globale

👉 Plateforme centralisée qui combine :

- threat intelligence
- OSINT
- metadata email
- workflows automatisés

➡️ Permet d’identifier rapidement :

- intent malveillant
- IOC

### 📊 Fonctionnalités clés

**1️⃣ Identification des artifacts**

|Élément|Description|
|---|---|
|Rendered HTML|Vue comme inbox|
|Raw HTML|Code brut|
|Message source|Headers complets|

👉 Accès direct à tous les artifacts importants

![[phishing analysis tooles identify artifacts.png]]

**2️⃣ Analyse avancée**

|Section|Utilité|
|---|---|
|Authentication|SPF / DKIM / DMARC|
|Transmission|Chemin du mail|
|URLs|Analyse des liens|
|Attachments|Analyse intégrée|

![[phishing analysis tools further analysis.png]]

**3️⃣ Intégration Threat Intel**

- Intégration directe avec VirusTotal  
    👉 Vérification rapide :
- réputation
- détections

![[phishing analysis tools threat intel.png]]

**4️⃣ Résolution du cas**

|Action|Objectif|
|---|---|
|Classifier (malicious / legit)|Conclusion analyse|
|Flag artifacts|Mettre en évidence IOC|
|Ajouter notes|Documentation|
|Resolve|Clôturer le cas|
![[phishing analysis tools resolving case.png]]

### ⚠️ Points clés

- Outil **tout-en-un**
- Gain de temps énorme (automatisation)
- Reproduit un workflow **réel SOC**
- Centralise :
    - analyse
    - intel
    - reporting

---

## Your Account is on Hold (Practice)

**Cas de phihsing**

![[phishing analysis tools cas de phishing.png]]

**Intended recipient

redacted@yahoo.com

**IP address of sender / domain**

![[phishing analysis tools IP & domain.png]]

**Shortened URL**

![[phishing analysis tools shortened URL.png]]

---

## TP ANYRUN cf : https://tryhackme.com/room/phishingemails3tryoe

