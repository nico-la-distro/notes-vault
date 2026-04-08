## 1) Objectif de la room

Comprendre **pourquoi** la gouvernance & la réglementation sont clés en cyber, et **quels cadres/standards** (GRC, GDPR, PCI DSS, NIST, ISO 27001, SOC 2…) structurent la sécurité d’une organisation.

---

## 2) Terminologie essentielle (à connaître par cœur)

|Terme|Définition (ultra court)|
|---|---|
|**Gouvernance**|Diriger/contrôler l’organisation pour atteindre ses objectifs + conformité.|
|**Réglementation**|Règle/loi imposée par une autorité (obligatoire).|
|**Conformité (Compliance)**|État de respect des lois/règles/standards applicables.|

---

## 3) Gouvernance SSI : ce que ça recouvre

La **gouvernance SSI** = structure + politiques + méthodes + guidelines pour garantir **Confidentialité / Intégrité / Disponibilité** des actifs info (CIA), portée par le top management.

**Processus typiques :**

- **Stratégie** (alignée business)
    
- **Politiques & procédures** (règles d’usage/protection)
    
- **Gestion des risques** (évaluer + réduire)
    
- **Mesure de performance** (KPIs)
    
- **Conformité** (réglementaire + bonnes pratiques)
    

**Réglementation SSI** = cadre légal/réglementaire qui impose la protection des données/systèmes (obligation, contrôles, sanctions).

---

## 4) Pourquoi c’est important (bénéfices)

- **Posture de sécurité + robuste** (réduction du risque de breach)
    
- **Confiance des parties prenantes** (clients/partenaires)
    
- **Éviter sanctions + dommages réputationnels** (non-conformité)
    
- **Alignement business** (investissements sécurité pertinents)
    
- **Décisions mieux informées** (visibilité risques)
    
- **Avantage compétitif** (preuve d’engagement sécurité)
    

---

## 5) Panorama lois / règlements cités

|Loi / Standard|Domaine|Idée clé|
|---|---|---|
|**GDPR**|Données perso (UE)|Exigences strictes sur traitement/protection des données perso UE.|
|**HIPAA**|Santé (USA)|Protection des infos de santé.|
|**PCI DSS**|Paiement / cartes|Exigences techniques & opérationnelles pour sécuriser les données carte.|
|**GLBA**|Finance (USA)|Protection des NPI + programme sécurité + notices privacy.|

---

## 6) Framework documentaire SSI (policies → baselines)

|Document|But|“Obligatoire ?” (dans l’organisation)|
|---|---|---|
|**Policy**|Direction/intentions + règles haut niveau|Oui (souvent)|
|**Standard**|Exigences précises (quoi respecter)|Oui|
|**Guideline**|Recommandations / best practices|Plutôt non (souvent)|
|**Procedure**|Étapes détaillées (comment faire)|Oui|
|**Baseline**|Minimum sécurité exigé|Oui|

### Cycle de création (mémo)

1. **Scope & purpose** (pourquoi + périmètre)
    
2. **Research & review** (lois/standards + existant interne)
    
3. **Draft** (clair, actionnable, aligné)
    
4. **Review & approval** (SMEs, légal, compliance, management)
    
5. **Implementation & communication** (diffusion + training)
    
6. **Review & update** (monitor compliance + maj continue)
    

**Exemples donnés :**

- Password Policy : exigences (longueur/complexité/expiration), usage, stockage/transmission (chiffrement), reset, communication, monitoring.
    
- Incident Response Procedure : types d’incidents, rôles, étapes (containment → evidence → analyse → recovery), reporting + doc, communication, mise à jour.
    

---

## 7) GRC (Governance, Risk, Compliance)

### Définition

Cadre **intégré** pour piloter gouvernance + gestion des risques + conformité de façon cohérente avec les objectifs orga.

|Composant|Rôle|
|---|---|
|**Governance**|Direction/stratégie sécurité (policies, standards…) + mesure performance.|
|**Risk Management**|Identifier/évaluer/prioriser risques + contrôles/mitigation + suivi.|
|**Compliance**|Respect obligations légales/industrielles + audits/assessments + reporting.|

### Étapes “génériques” pour construire un programme GRC

- Définir **scope & objectifs**
    
- Faire une **risk assessment**
    
- Écrire **policies/procedures**
    
- Mettre en place **processus de gouvernance** (comité, rôles)
    
- Implémenter **contrôles** (techniques + humains : FW, IDS/IPS, SIEM, formation)
    
- **Monitor & measure** (métriques, conformité)
    
- **Continuous improvement** (retours, incidents, évolution risques)
    

---

## 8) Privacy & Data Protection (GDPR + PCI DSS)

### GDPR (mémo)

- Loi UE (mise en œuvre **mai 2018**) pour protéger les données perso.
    
- Principes cités : **consentement préalable**, **minimisation**, **mesures de protection**.
    
- Applicabilité : toute entité qui opère dans l’UE et traite des données de résidents UE.
    

|Sanctions GDPR (dans la room)|Max|
|---|---|
|**Tier 1** (violations plus sévères)|**4% CA** ou **20M€** (le plus élevé)|
|**Tier 2** (moins sévères)|**2% CA** ou **10M€** (le plus élevé)|

### PCI DSS (mémo)

Standard centré sur la **sécurisation des transactions cartes** (contrôle d’accès, monitoring, protections type WAF + chiffrement). Créé par grandes marques de cartes (Visa, MasterCard, AmEx).

---

## 9) NIST Special Publications

### NIST 800-53 (mémo)

Catalogue de **contrôles sécurité & privacy** pour protéger CIA, utilisé comme framework d’évaluation/amélioration + conformité. La room mentionne **Revision 5** avec contrôles organisés en **20 familles**.

**Bonnes pratiques de conformité (résumé) :**

1. **Découvrir/inventorier** actifs, systèmes, menaces, flux, dépendances, vulnérabilités
    
2. **Mapper** familles de contrôles ↔ actifs/risques identifiés
    
3. Mettre en place **gouvernance** (responsabilités + procédures de maintien)
    
4. **Monitor & evaluate** la conformité régulièrement
    
5. **Détection + audits + amélioration continue**
    

### NIST 800-63B (mémo)

Guidelines sur **l’identité numérique** : authentification/verification, niveaux d’assurance, facteurs (passwords, biométrie, tokens), gestion sécurisée des credentials.

---

## 10) IS Management & Compliance : ISO 27001 + SOC 2

### ISO/IEC 27001 (ISMS) — composants cités

Standard international pour **planifier / construire / opérer / améliorer** un **ISMS**.

|Composant|À quoi ça sert|
|---|---|
|**Scope**|Délimiter périmètre ISMS (actifs/process)|
|**Politique SSI**|Document haut niveau, ligne directrice|
|**Risk assessment**|Identifier/évaluer risques CIA|
|**Risk treatment**|Choisir/implémenter contrôles pour réduire le risque|
|**SoA**|Dire quels contrôles s’appliquent / non|
|**Internal audit**|Audits périodiques ISMS|
|**Management review**|Revue régulière performance ISMS|

### SOC 2 (AICPA) — l’essentiel

Framework d’audit/conformité pour évaluer l’efficacité des contrôles de sécurité d’un service provider ; utile comme **preuve** pour clients/partenaires. Audits par **tiers indépendants**, rapport avec constats/reco.

**Étapes audit SOC 2 (mémo) :**  
scope → choisir auditeur → planifier → préparer (gap analysis) → audit/tests/interviews → rapport.

---

## 11) Conclusion (idée centrale)

La room martèle : **100% sécurité n’existe pas**, mais une orga mature met en place **politiques + contrôles + conformité + amélioration continue** pour réduire les risques et protéger les données/systèmes.