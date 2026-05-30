## Objectifs

- Comprendre ce qu'est la CTI et pourquoi elle est utile au SOC L1
- Connaître le lifecycle du threat intelligence et les indicateurs à surveiller
- Se familiariser avec le partage de renseignements via des feeds et des plateformes

---

## Cyber Threat Intelligence

In concrete terms, CTI seeks to answer three essential questions:

1. **Who, or what, is on the other end of this alert indicator?**
2. **What was their behaviour in the past?**
3. **How does my organisation respond, and what should I do about it right now?**

### From Raw Data to Usable Intelligence

|Layer|Définition|Exemple|Action SOC L1|
|---|---|---|---|
|Data|Observable brut|`45.155.205.3:443`|Capturer l'artefact|
|Information|Data + annotation factuelle|IP enregistrée chez Hetzner, first seen 2023-07-14|Enregistrer les attributs|
|Intelligence|Information analysée -> répond au "so what"|IP = C2 BumbleBee actif -> bloquer immédiatement|Escalader ou supprimer|

- **IOC** (**Indicator of Compromise**) : preuve d'une compromission (ex: adresse C2 dans les logs)
- **IOA** (**Indicator of Attack**) : action malveillante en cours (ex: PowerShell lançant un service inconnu)
- **TTP** (**Tactics, Techniques, and Procedures**) : méthodologies détaillées de l'adversaire, exprimées via MITRE ATT&CK

### Indicator Types Essential to First-Line Triage

|Indicator|Exemple|Ressources|IOA / TTP associés|
|---|---|---|---|
|IPv4/IPv6|`45.155.205.3`|WHOIS, VirusTotal, Shodan|T1110.003 Password Guessing|
|Domain/FQDN|`malicious-updates[.]net`|WHOIS age, SecurityTrails, urlscan.io|Surge DNS vers domaine < 24h|
|URL|`hxxp://malicious-updates[.]net/login`|URLhaus, urlscan.io, Any.Run|Browser POST /gateway.php avec payload|
|File hash|`e99a18c428cb38d5...`|VirusTotal, Hybrid-Analysis, MalShare|T1055 Process Injection|
|E-mail|`billing@evil-corp.com`|MXToolbox, HaveIBeenPwned|SPF failure + domaine récent|
|Artefact local|`HKCU\Software\Run\updater.exe`|Sigma rules, EDR, vendor KB|T1060.001 Registry Run Keys|

### Feeds, Platforms, and Why the Distinction Matters

- **Feed** : flux d'indicateurs (CSV, JSON, STIX, TAXII) -> sur-ingestion sans curation = faux positifs
- **Platform** : dépôt structuré, stocke IOCs, enrichissement, relations, permissions (ex: [MISP](https://www.misp-project.org/), [OpenCTI](https://opencti.io/)) -> source de vérité unique

### Sources of Cyber Threat Intelligence

- **Internal telemetry** : SIEM, EDR, phishing mailbox -> relevance maximale
- **Commercial** : feeds payants, sandboxes closed-source -> haute fidélité, mais restrictions de licence
- **OSINT** : AbuseIPDB, URLhaus, blogs, recherche académique -> nécessite cross-confirmation
- **Communities / ISACs** : listes sectorielles avec contexte riche (ex: FS-ISAC)

### Threat Intelligence Classifications

|Classe|Focus|Exemple|
|---|---|---|
|**Strategic**|Tendances macro, impact business|Rapport annuel ransomware -> shift data-wiping dans la santé|
|**Tactical**|TTPs adversaires|Advisory : abus T1059.005 (VBS) dans malspam|
|**Operational**|Motifs et cibles d'une campagne|Assets critiques (personnes, process, tech) à risque|
|**Technical**|IOCs atomiques|IPs, hashes liés à une attaque|

> L1 -> escalade les IOCs **Technical**, documente les IOAs **Tactical**, identifie les patterns **Operational**.

---

## CTI Lifecycle

### Traffic Light Protocol (TLP) - A Primer for Proper Sharing

|Label|Périmètre de partage|Action SOC L1|
|---|---|---|
|**TLP: CLEAR**|Aucune restriction|Poster sur le wiki interne / platform|
|**TLP: GREEN**|Communauté pair, pas public|Upload MISP / Slack restreint aux SOCs partenaires|
|**TLP: AMBER**|Organisation + clients need-to-know|Garder dans la CTI platform, référencer sans copier dans les tickets|
|**TLP: RED**|Destinataires nommés uniquement|Note chiffrée, ne pas poster dans le ticketing sans autorisation|

-> Le label TLP le plus strict prévaut en cas de collision entre sources.

### Intel Formats

**STIX** (Structured Threat Information Expression) : schéma JSON machine-readable pour décrire indicateurs, relations et contexte.

### Scenario Premise

TryHatMe Corp holds sensitive customer data in a PostgreSQL server in a segmented subnet. The server is fronted by a next-generation firewall (NGFW) and monitored on-host by an Endpoint Detection and Response (EDR) agent. Senior management has asked the SOC to "bring in cyber-threat intelligence" so that the controls react rapidly to emerging threats. The morning-shift L1 analyst, Alex, is assigned to build and exercise that workflow.
#### Step 1: Direction - Defining the Mission

Traduire le mandat en **intelligence requirements** mesurables.

Exemple :

- Q1 : Quelles IPs/domaines exploitent actuellement PostgreSQL ou exfiltrent des données ?
- Q2 : Quelles familles de malware ciblant PostgreSQL sont actives cette semaine (hashes) ?

-> Les questions deviennent les critères de succès de la phase Feedback.

#### Step 2: Collection - Assembling the Raw Material

|Source|Exemple d'artefacts|
|---|---|
|Feed commercial NGFW vendor|37 IPv4 flaggées C2 database-exfil (24h)|
|AbuseIPDB (tag PostgreSQL-brute-force)|15 IPs, 4 domaines|
|MISP interne|2 SHA-256 (credential stealers PgSQL)|
|Vendor threat report hebdo|1 hash malware, 3 domaines C2|

Stocker chaque feed (STIX ou CSV) dans un bucket "raw-intel" daté -> reproductibilité.

#### Step 3: Processing - Normalising and Correlating the Data

- **Normalisation** : syntaxe unifiée (IPv6 compressé, domaines lowercase, masques subnet retirés)
- **Corrélation / déduplication** : croisement avec la table d'indicateurs existante
- **Tagging** : source + date + TLP
- **Output** :
    - `firewall_blocklist.csv` -> NGFW
    - `edr_hash_rules.yar` -> EDR (YARA)

#### Step 4: Analysis - Turning Information Into Judgement

Évaluer la pertinence avant de bloquer (éviter les faux positifs).

|Confidence|Source agreement|Sightings locaux|Action|
|---|---|---|---|
|High|IOC dans >= 2 sources|>= 1 tentative locale|Blocage immédiat|
|Medium|1 source de confiance|Aucun hit local|Alert-only|
|Low|OSINT uniquement|Pas de contexte|Monitor 14 jours|

#### Step 5: Dissemination - Getting Intelligence to the Right Consumers

|Stakeholder|Format|Raison|
|---|---|---|
|Firewall team|CSV + change-ticket|Documentation du risque + TLP|
|Endpoint team|YARA ruleset dans EDR console|Chargement dans la policy|
|CTI platform|Indicator objects taggés|Historique, corrélations futures, TLP respecté|
|Management|Résumé 200 mots (weekly memo)|ROI du processus|

#### Step 6: Feedback - Measuring and Refining the Cycle

Mesurer les KPIs après chaque cycle -> valider les objectifs de la Direction -> itérer.

Exemple :

|KPI|Baseline|Après 1er cycle|
|---|---|---|
|Dwell time IPs brute-force PgSQL|48h|0h (blocage préemptif)|
|Taux faux positifs|-|0%|

-> Feedback validé = expansion du scope au sprint suivant -> mise à jour du Direction document.

---

## CTI Standards & Frameworks

### MITRE ATT&CK

Chaque technique (ex: T1059.001 PowerShell, T1048.003 DNS tunnel) = label universel entre vendors, équipes et auditeurs.

Usage L1 :

1. Matcher le comportement de l'alerte à une paire tactic/technique
2. Écrire l'ID dans la note de triage : `"Observed T1071.001 (web-based C2) against FINANCE-TRYHATME-00"`
3. Passer au L2/IR -> ils savent immédiatement quelles mitigations et profils d'acteurs s'appliquent

### MITRE D3FEND

Catalogue les réponses défensives (ex: Credential Hardening, Data Obfuscation).

Usage L1 : alerte T1048.003 DNS tunnel -> chercher dans D3FEND la technique défensive correspondante -> ajouter le contrôle feasible dans "next actions".

### Cyber Kill Chain

Développé par Lockheed Martin. 7 phases :

|Phase|Objectif|Exemples|
|---|---|---|
|Reconnaissance|Collecter infos sur la cible|OSINT, scan réseau, harvesting emails|
|Weaponisation|Créer le malware adapté|Exploit + backdoor, doc Office malveillant|
|Delivery|Livrer le malware|Email, lien web, USB|
|Exploitation|Exploiter des vulnérabilités|EternalBlue, Zero-Logon|
|Installation|Installer malware / outils d'accès|Password dumping, RAT, backdoor|
|Command & Control|Contrôle à distance, pivot, élévation|Empire, Cobalt Strike|
|Actions on Objectives|Atteindre l'objectif final|Ransomware, exfiltration, defacement|

-> Étendu depuis via ATT&CK -> **Unified Kill Chain**.

### CVEs, CVSS, and the NVD

- **CVE** : identifiant unique d'une vulnérabilité (ex: `CVE-2023-4863`)
- **CVSS** : score de sévérité 0-10 avec modificateurs temporels et environnementaux
- **NVD** : dépôt canonique liant CVE + CVSS + exploits + produits affectés -> [nvd.nist.gov](https://nvd.nist.gov)

### Sharing and Processing Intel

- **STIX** : schéma JSON structuré pour décrire la threat intelligence
- **TAXII** : APIs sécurisées d'échange en quasi-temps réel. Deux modèles :
    - **Collection** : intel hébergée par un producteur, consommée à la demande
    - **Channel** : intel publiée depuis un serveur central vers les abonnés

Limites au partage :

- Lois sur la vie privée, NDAs clients, infos compétitives internes
- Partager un IOC trop tôt peut alerter l'adversaire que sa campagne est détectée

---

## Résumé - Cyber Threat Intelligence for SOC L1

### Ce qu'est la CTI

Contexte qui permet de décider quelles alertes représentent un danger réel. Répond à 3 questions : **qui**, **comportement passé**, **que faire maintenant**.

Data brute -> Information (annotée) -> Intelligence (actionnable).

### Les indicateurs essentiels

- **IOC** : preuve de compromission
- **IOA** : action malveillante en cours
- **TTP** : méthodologies adversaires (MITRE ATT&CK)

Types : IPv4/IPv6, domaine, URL, hash, email, artefact local -> chaque type a ses outils de lookup dédiés.

### Les 4 classifications du threat intel

|Classe|Utilisateur principal|
|---|---|
|Strategic|Management|
|Tactical|L2 / IR|
|Operational|CTI analysts|
|Technical|L1 -> enrichissement IOCs|

### Le CTI Lifecycle (6 étapes)

**Direction** -> **Collection** -> **Processing** -> **Analysis** -> **Dissemination** -> **Feedback**

Point critique : grader chaque IOC (High/Medium/Low) avant de bloquer -> éviter les faux positifs.

### Standards à connaître

- **MITRE ATT&CK** : labelliser les techniques dans les notes de triage
- **MITRE D3FEND** : trouver la contre-mesure associée
- **Cyber Kill Chain** : situer l'attaquant dans sa progression (7 phases)
- **CVE / CVSS / NVD** : gérer les vulnérabilités
- **STIX / TAXII** : format + transport pour le partage d'intel
- **TLP** : gouverner qui peut voir et partager quoi

### Réflexes L1

1. Identifier le type d'indicateur -> choisir les bons outils de lookup
2. Toujours noter la source de chaque IOC
3. Faire voyager le label TLP avec l'indicateur
4. Écrire l'ID ATT&CK dans chaque note de triage
5. Ne pas partager un IOC actif trop tôt -> peut alerter l'adversaire

