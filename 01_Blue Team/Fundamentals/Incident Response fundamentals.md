
- En cybersécurité, un **incident** = cyberattaque/événement entraînant des **pertes/impacts** (souvent financiers), rapportés très fréquemment.

- **Incident Response (IR)** = gérer un incident **de bout en bout** :  
    **préparation + détection + lutte/containment + réduction d’impact + retour à la normale** (guidelines complètes).

|Concept|À retenir (1 ligne)|
|---|---|
|Mesures proactives|Sécurité avant attaque (prévention/dissuasion).|
|Mesures post-attaque|Procédures quand l’attaquant est déjà dedans.|
|Cyber Security Incident|Événement/attaque avec impact (pertes, dégâts).|
|Incident Response|Processus complet pour gérer l’incident du début à la fin.|
|Frameworks (SANS/NIST)|Modèles de phases IR standardisés.|
|Playbooks|Procédures “prêtes à l’emploi” pour répondre vite et correctement.|
|Incident Response Plan|Document/plan global d’organisation et de réponse.|

## **What are incidents ?**

| Terme               | Définition courte                             | Exemple                              |
| ------------------- | --------------------------------------------- | ------------------------------------ |
| Événement (event)   | Action effectuée par un process               | ouverture fichier, connexion réseau… |
| Log                 | Événements collectés/stockés                  | journaux système/app                 |
| Alerte (alert)      | Signal généré par une solution de sécurité    | “transfert de données anormal”       |
| False Positive (FP) | Alerte suspecte mais bénigne                  | backup cloud = gros transfert        |
| True Positive (TP)  | Alerte confirmée malveillante                 | email de phishing réel               |
| Incident            | TP traité comme événement de sécurité à gérer | phishing confirmé, malware, etc.     |
| Sévérité            | Impact → priorité de traitement               | Critical > High > Medium > Low       |

**Détection → Alerte → Triage (FP/TP) → Incident (si TP) → Sévérité → Priorisation de la réponse.**

## **Types of incidents**

- Les incidents peuvent survenir **séparément** ou **en combinaison** chez la même victime.

- La **sévérité dépend du contexte** : le même type d’incident peut être **critique** pour une orga et **mineur** pour une autre (ex : fuite de données “peu utile” vs DoS sur site vital).

|Type d’incident|Définition|Point clé / Exemple|
|---|---|---|
|**Malware Infection**|Programme malveillant qui endommage système/réseau/app|Très fréquent ; souvent via fichiers (doc, exe, etc.)|
|**Security Breach**|Accès **non autorisé** à des données confidentielles|Priorité élevée car touche le “confidentiel”|
|**Data Leak**|Données confidentielles **exposées** à des non-autorisés|Peut être **malveillant** ou **accidentel** (erreur humaine/mauvaise config)|
|**Insider Attack**|Attaque initiée **depuis l’intérieur**|Dangereux car l’insider a souvent plus d’accès (ex : employé + USB)|
|**DoS (Denial of Service)**|Inonder de requêtes pour rendre un service **indisponible**|Impacte la **disponibilité** (épuisement ressources)|

**à retenir**

**Pas de comparaison “universelle”** de sévérité entre types : tout dépend de l’activité de l’organisation.  
Exemple : fuite de données peu sensible = faible impact ; **DoS sur site principal** = pertes majeures.

## **Incident Response Process**

Frameworks **SANS** et **NIST** : approches génériques (très proches) pour répondre efficacement.

### **SANS IR = 6 phases (PICERL)**

|Phase|But (ultra-court)|Exemple typique|
|---|---|---|
|**Preparation**|Se préparer (équipes, plan, outils)|sensibilisation phishing|
|**Identification**|Détecter/qualifier l’incident|exfiltration détectée, host compromis|
|**Containment**|Limiter l’impact|isoler machine / désactiver comptes|
|**Eradication**|Retirer la menace|scan / nettoyage malware|
|**Recovery**|Remettre en service|restore backup / rebuild + tests|
|**Lessons Learned**|Améliorer après coup|post-mortem, root cause, correctifs|

---

### **NIST IR = 4 phases**

|Phase (NIST)|But (ultra-court)|Exemple typique|
|---|---|---|
|**Preparation**|Mettre en place équipes, procédures, outils|former au phishing, déployer SIEM/EDR, définir l’IRP|
|**Detection & Analysis**|Détecter, analyser, confirmer + évaluer la sévérité|alerte exfiltration → tri FP/TP → incident confirmé + criticité|
|**Containment, Eradication & Recovery**|Stopper la propagation, supprimer la cause, remettre en service|isoler host → supprimer malware/persistance → restaurer backup + tests|
|**Post-Incident Activity**|Capitaliser et améliorer le dispositif|post-mortem, RCA, mise à jour playbooks/IRP, durcissement/patch|

---

### **SANS vs NIST**

![[SANS (ir) vs NIST (ir).png]]

---
### **Incident Response Plan (IRP)**

- **Document officiel** (validé management) qui décrit quoi faire **avant / pendant / après** un incident.

**Composants clés**

- **Rôles & responsabilités**
- **Méthodologie IR** (souvent basée SANS/NIST)
- **Plan de communication** (stakeholders, éventuellement forces de l’ordre)
- **Chemin d’escalade** (qui alerter, quand, comment)

## **Incident Response Techniques**

|Solution|Rôle principal|Ce que ça apporte en IR|
|---|---|---|
|**SIEM**|Centralise les **logs** + **corrèle** pour détecter des incidents|Vision globale, détection par corrélation multi-sources|
|**AV (Antivirus)**|Détecte des malwares **connus** + scans réguliers|Protection “signature-based”, baseline|
|**EDR**|Agent endpoint : détecte des menaces avancées + **réponse** possible|Peut **contenir** et **éradiquer** (isolation, kill process, etc.)|

---

### **Playbooks vs Runbooks**

|Terme|Définition|Niveau|
|---|---|---|
|**Playbook**|**Guide** / procédure type pour gérer un **type d’incident**|“quoi faire” (workflow)|
|**Runbook**|**Exécution détaillée** étape par étape (selon outils/ressources dispo)|“comment le faire” (opérationnel)|

**exemple de playbook : Phishing email**

- Notifier les **stakeholders**
- Vérifier si malveillant : analyse **header + body**
- Identifier / analyser les **pièces jointes**
- Vérifier si quelqu’un a **ouvert/exécuté** la PJ
- **Isoler** les systèmes infectés
- **Bloquer** l’expéditeur (et idéalement les IOC associés)

