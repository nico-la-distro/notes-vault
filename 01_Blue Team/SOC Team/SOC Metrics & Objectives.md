
- Protéger la **confidentialité**, **intégrité** et **disponibilité** des actifs numériques (**CIA**).
    
- Process SOC : **développer / recevoir / trier (triage)** des alertes.
    
- Rôle **L1** : identifier correctement les **True Positives** et **escalader** vers **L2**.

---

## Core Metrics

| Metric                          | Formule                                       | Ce que ça mesure                   | Interprétation / cible                                                                                                                                                                      |
| ------------------------------- | --------------------------------------------- | ---------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Alerts Count (AC)**           | AC = **nb total d’alertes reçues**            | **Charge** de travail SOC          | Trop haut → surcharge + risques de louper un vrai threat. Trop bas → **SIEM en panne / manque de visibilité**. Repère : **~5 à 30 alertes/jour/L1** (selon taille).                         |
| **False Positive Rate (FPR)**   | FPR = **False Positives / Total Alerts**      | **Bruit** (noise) dans les alertes | **≥ 80% = gros problème** → fatigue, baisse de vigilance, “tout est du spam”. Solution : **tuning des outils/règles** (“**False Positive Remediation**”). 0% = idéal théorique, irréaliste. |
| **Alert Escalation Rate (AER)** | AER = **alertes escaladées / total alertes**  | **Expérience / autonomie** des L1  | L1 doit filtrer le bruit mais ne pas être “trop confiant” sur ce qu’il ne comprend pas. Cible : **< 50%**, idéal **< 20%**.                                                                 |
| **Threat Detection Rate (TDR)** | TDR = **threats détectées / threats totales** | **Fiabilité SOC** (détection)      | Doit viser **100%** : chaque menace manquée peut être critique (ex : **ransomware**, **exfiltration**). Exemple 4/6 = **67% = très mauvais**.                                               |

---

### À retenir

- **AC** équilibre : **trop d’alertes = bruit**, **pas assez = aveugle**.
    
- **FPR** élevé = **SOC noyé**, se corrige par **remédiation FP** (tuning règles/SIEM).
    
- **AER** = proxy d’**expérience L1** (trop haut = escalade excessive / manque d’autonomie).
    
- **TDR** = métrique “ultime” : **un seul échec peut coûter très cher**, donc objectif **100%**.

---

## Triage Metrics

Une **alerte seule n’arrête pas** une compromission : il faut **détecter**, **prendre en charge (triage)**, puis **répondre** avant que l’attaquant atteigne son objectif.

![[SOC Metrics & Objectives (Triages Metrics).png]]

---

### SLA (Service Level Agreement)

- **Document d’engagement** entre :
    
    - SOC interne ↔ management de l’entreprise, **ou**
        
    - **MSSP** ↔ client
        
- Fixe des exigences de **rapidité** sur toute la chaîne **détection → triage → réponse**.
    

---

#### Timeline des métriques (ordre logique)

1. **MTTD** : temps **attaque → détection** (outils SOC)
    
2. **MTTA** : temps **alerte → début de triage L1**
    
3. **MTTR** : temps **triage → action de remédiation** (stopper la propagation)
    

---

#### Table de référence (SLA typiques)

|Metric|SLA courant|Mesure / définition|
|---|---|---|
|**SOC Team Availability**|**24/7**|Plage de couverture SOC (ex : **8/5** ou **24/7**)|
|**MTTD (Mean Time to Detect)**|**5 min**|Temps moyen entre **l’attaque** et sa **détection par les outils**|
|**MTTA (Mean Time to Acknowledge)**|**10 min**|Temps moyen pour qu’un **L1 commence le triage** d’une nouvelle alerte|
|**MTTR (Mean Time to Respond)**|**60 min**|Temps moyen pour **stopper réellement** l’incident (ex : **isoler** un device, **sécuriser** un compte)|

---

#### Note importante

- Les **définitions/formules peuvent varier** selon les équipes (selon ce qu’elles veulent mesurer).
    
- Pour les exercices, il faut s’aligner sur **l’illustration** et la **table de référence** donnée.

---

## Improving Metrics

### Pourquoi les métriques comptent pour toi (L1)

- Elles servent à **rendre le SOC plus efficace** → attaques **moins réussies**.
    
- Elles sont aussi un **outil d’évaluation** : de bons résultats = **progression** (vers **L2**) + **augmentation**.
    
- Logique : les métriques doivent **s’améliorer en continu**.

---

### Comment améliorer les métriques

| Issue (seuil “mauvais”) | Ce que ça signifie                                                    | Recommandations concrètes                                                                                                                                                                     |
| ----------------------- | --------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **FPR > 80%**           | Trop de **bruit** (noise) → fatigue + risque de louper un vrai threat | 1) **Exclure** les activités “trusted” (ex : **updates système**) des règles **EDR/SIEM** 2) **Automatiser** le triage des alertes fréquentes via **SOAR** ou **scripts**                     |
| **MTTD > 30 min**       | Détection **trop lente** (règles/logs)                                | 1) Contacter les **SOC engineers** pour accélérer / augmenter la fréquence d’exécution des **règles** 2) Vérifier que les logs SIEM arrivent **en temps réel** (pas de délai ex : **10 min**) |
| **MTTA > 30 min**       | Les L1 commencent le triage **trop tard**                             | 1) S’assurer de la **notification en temps réel** à l’arrivée d’une alerte 2) **Répartir** la file d’alertes **équitablement** entre analystes en shift                                       |
| **MTTR > 4 h**          | La remédiation est **trop lente** → breach continue de s’étendre      | 1) En tant que L1, **escalader rapidement** vers L2 tout ce qui est menaçant/actionnable 2) Avoir des **playbooks / procédures** documentés par scénario d’attaque                            |
