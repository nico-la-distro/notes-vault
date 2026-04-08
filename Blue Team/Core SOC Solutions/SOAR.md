**Security Orchestration, Automation and Response**

Le **SOAR** aide les équipes **SOC** à mieux gérer les incidents de sécurité. Le SOAR est conçu pour **réduire la charge manuelle**, **centraliser les actions** et **rendre la réponse du SOC plus rapide et plus efficace**.

---

## Traditional SOC & Challenges

Un **SOC (Security Operations Center)** est un centre centralisé chargé de **surveiller**, **détecter** et **répondre** aux menaces pour protéger les actifs numériques de l’organisation.

Son efficacité repose sur 3 piliers :

- **People** (personnes)
    
- **Processes** (processus)
    
- **Technologies** (technologies)
    

### Capacités principales d’un SOC

|Capacité|Rôle|Exemples|
|---|---|---|
|**Monitoring & Detection**|Surveiller en continu et détecter les activités suspectes|nombreux échecs de connexion, connexion depuis un lieu inconnu|
|**Recovery & Remediation**|Répondre aux incidents et limiter l’impact|isoler un endpoint via EDR, bloquer une IP sur le firewall, désactiver un compte IAM|
|**Threat Intelligence**|Utiliser des IOC / renseignements de menace à jour|bloquer un domaine malveillant identifié par les feeds TI|
|**Communication**|Coordonner avec IT et management|créer un ticket pour vérifier un patch déployé|

---

### Défis d’un SOC traditionnel

|Problème|Explication|Impact|
|---|---|---|
|**Alert Fatigue**|trop d’alertes, souvent fausses ou peu utiles|analystes submergés, vraies menaces plus dures à repérer|
|**Disconnected Tools**|outils non intégrés entre eux|investigations plus lentes et complexes|
|**Manual Processes**|procédures souvent manuelles et mal documentées|réponse incohérente, dépendance à l’expérience individuelle|
|**Talent Shortage**|manque d’analystes qualifiés|surcharge de travail, délais de réponse plus longs|

---

### Idées clés à retenir

- Le **SOC traditionnel** protège l’organisation grâce à la **surveillance**, la **réponse**, la **threat intel** et la **communication**.
    
- Son principal problème est la **complexité opérationnelle**.
    
- Trop d’outils, trop d’alertes et trop de tâches manuelles rendent la réponse **plus lente** et **moins efficace**.
    
- C’est justement ce type de limites que le **SOAR** cherche à corriger.

---

## Overcoming SOC Challenges with SOAR

Le **SOAR (Security Orchestration, Automation, and Response)** est une plateforme qui **centralise les outils de sécurité** d’un SOC dans **une seule interface**.

Il permet aussi :

- le **ticketing**
    
- le **case management**
    
- le **suivi structuré des incidents**
    

---

### Les 3 capacités principales du SOAR

|Capacité|Rôle|Bénéfice|
|---|---|---|
|**Orchestration**|connecte plusieurs outils dans une même plateforme et définit des **playbooks**|évite de jongler entre SIEM, EDR, TI, IAM, ticketing|
|**Automation**|exécute automatiquement les étapes prévues par les playbooks|réduit les clics manuels, accélère le traitement|
|**Response**|permet d’agir directement sur les outils connectés|blocage, désactivation, ouverture de ticket, etc.|

---

### Playbook : idée clé

Un **playbook** = une suite d’étapes prédéfinies pour traiter un type d’alerte.

Exemple sur une **alerte VPN brute force** :

1. réception de l’alerte depuis le **SIEM**
    
2. vérification dans le **SIEM** des connexions habituelles de l’utilisateur
    
3. vérification de la réputation de l’IP via la **Threat Intelligence**
    
4. recherche d’éventuelles connexions réussies
    
5. si nécessaire, passage aux actions de **containment**
    

#### Point important

Les playbooks sont **dynamiques** :

- le résultat d’une étape détermine la suivante
    
- le workflow peut s’arrêter tôt si l’alerte semble bénigne
    

---

### Ce que l’automatisation apporte

Avec l’automatisation, le SOAR peut :

- interroger automatiquement le **SIEM**
    
- vérifier automatiquement une IP via la **TI**
    
- désactiver automatiquement un utilisateur dans l’**IAM**
    
- ouvrir automatiquement un ticket
    

#### Résultat

- gain de temps important
    
- traitement de nombreux alertes à grande échelle
    
- réduction de la charge des analystes
    

---

### Comment le SOAR répond aux problèmes du SOC

|Problème du SOC|Apport du SOAR|
|---|---|
|**trop d’outils séparés**|centralisation dans une seule interface|
|**processus manuels**|automatisation via playbooks|
|**lenteur des investigations**|coordination rapide entre outils|
|**alert fatigue**|traitement automatique des tâches répétitives|

---

### SOAR remplace-t-il les analystes SOC ?

**Non.**

Le SOAR :

- automatise les tâches répétitives
    
- structure les investigations
    
- accélère la réponse
    

Mais les **analystes SOC restent indispensables** pour :

- les investigations complexes
    
- les décisions critiques
    
- la compréhension du contexte métier
    
- la création et l’amélioration des playbooks
    

---

### Idées clés à retenir

- **SOAR = centralisation + automatisation + réponse**
    
- **Playbook = procédure prédéfinie de traitement**
    
- le SOAR **réduit la charge opérationnelle**, mais **ne remplace pas l’humain**
    
- il rend le SOC **plus rapide, plus cohérent et plus efficace**

![[SOAR purpose.png]]

---

## Building SOAR Playbooks

Un **playbook SOAR** est un **workflow prédéfini** créé par les analystes SOC pour traiter des alertes récurrentes.

Il suit une logique conditionnelle :

- **si ceci arrive → faire cela**
    
- **sinon → autre action**

---

### 1) Phishing Playbook

#### Objectif

Automatiser l’analyse et la remédiation des emails suspects, car le phishing est :

- très fréquent
    
- chronophage à analyser
    
- souvent manuel (URL, pièces jointes, threat intel)
    

#### Logique générale du playbook

|Étape|Action|
|---|---|
|1|réception de l’alerte **“Suspicious email received”**|
|2|création d’un **ticket**|
|3|vérification : email avec **URL** ou **pièce jointe** ?|
|4|si non → **notification aux utilisateurs**|
|5|si oui → analyse de l’URL / pièce jointe|
|6|vérification via **Threat Intelligence**|
|7|si phishing confirmé → **remédiation**|

#### Idée clé

Le playbook phishing automatise surtout :

- l’analyse technique
    
- les vérifications de réputation
    
- les premières actions de réponse

![[SOAR playbook phishing.png]]

---

### 2) CVE Patching Playbook

#### Objectif

Automatiser la gestion des nouvelles **CVE** pour éviter :

- l’accumulation de backlog
    
- les retards de patch
    
- l’exposition prolongée aux vulnérabilités

#### Logique générale du playbook

|Étape|Action|
|---|---|
|1|détection / récupération des détails de la **CVE**|
|2|analyse de la vulnérabilité|
|3|évaluation du **niveau de risque**|
|4|création d’un **ticket de patching**|
|5|test du patch|
|6|déploiement en **production** si validation|

#### Idée clé

Le SOAR aide à traiter plus vite les CVE en automatisant :

- l’analyse
    
- la priorisation
    
- l’ouverture de ticket
    
- le suivi du patching
    

![[SOAR playbook CVE patching.png]]

---

### Comparatif rapide

|Playbook|But principal|Actions automatisées|
|---|---|---|
|**Phishing**|analyser et contenir un email suspect|ticketing, analyse URL/PJ, TI, remédiation|
|**CVE Patching**|gérer les vulnérabilités publiées|analyse CVE, scoring risque, ticketing, test et déploiement|

---

#### Rôle de l’analyste SOC

Même avec le SOAR, l’analyste reste nécessaire pour :

- les **décisions critiques**
    
- les **validations importantes**
    
- les **vérifications finales**
    
- la **création des playbooks**
    

---

#### Idées clés à retenir

- un playbook SOAR formalise une procédure en **workflow automatisé**
    
- il repose sur une logique **conditionnelle**
    
- **phishing** et **CVE patching** sont deux bons cas d’usage
    
- le SOAR automatise beaucoup, mais **l’humain reste indispensable** aux étapes sensibles

---

