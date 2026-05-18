## Security hierarchy

- Les **priorités cybersécurité** varient selon le métier :
    
    - Cabinets d’avocats → **confidentialité** (privacy des documents)
        
    - Usines → **disponibilité** (production)
        
    - Hôpitaux → **sécurité des patients**
        
- Donc chaque entreprise a une **approche sécurité** et une **structure d’équipe** différentes.
    
- Les **dirigeants (CEO)** visent les objectifs business globaux et ne gèrent pas le technique.
    
- D’où le rôle du **CISO** : comprendre les besoins business et organiser des **départements sécurité adaptés**.
    
- Chaîne hiérarchique type : **analystes SOC (techniques) → manager → CISO → CEO**.

### Structure des départements sécurité

- **Très petites entreprises** : l’IT fait aussi la sécurité.
    
- **PME** : équipe “Information Security” généraliste (multi-tâches).
    
- **Grandes entreprises** : un **CISO** supervise **plusieurs équipes spécialisées**.

### Équipes principales

|Équipe|Objectif|Missions typiques|
|---|---|---|
|**Red Team**|Attaquer pour trouver des failles|Pentest, offensive security, ethical hacking|
|**GRC**|Gouvernance & conformité|Politiques, audits, conformité (ex : **PCI DSS**)|
|**Blue Team**|Défendre & détecter|SOC analyst, ingénierie sécu, réponse à incident|

![[SOC Hierarchy.png]]

---
## Blue Team

- **Objectif :** surveiller en continu, **détecter** les attaques et **répondre vite**.
    
- Taille typique : **~3 à 50 personnes** selon secteur + taille de l’entreprise.
    

---
### Départements Blue Team les plus courants

#### 1) SOC (Security Operations Center)

- **Rôle central / 1ère ligne de défense** : hub sécurité de l’organisation.
    
- **Travail :** traite les **alertes**, gère la plupart des attaques, coopère avec l’IT, **crée/ajuste des règles de détection**.
    

![[SOC Team.png]]

|Rôle|Niveau|Missions principales|
|---|---|---|
|**L1 Analyst**|Junior|**Triage** des alertes, escalade des cas complexes vers L2|
|**L2 Analyst**|Expérimenté|Investigation d’attaques **plus avancées**|
|**Engineer**|Expert outils|Configuration/optimisation **EDR, SIEM** (et autres)|
|**SOC Manager**|Management|Pilotage global de l’équipe SOC|

> Point “carrière” : **SOC = point d’entrée fréquent** en cybersécurité.

---
#### 2) CIRT / CSIRT / CERT (Cyber Incident Response Team)

- **“Pompiers”** : intervient quand le SOC ne suffit pas / incident **critique** / situation qui dégénère.
    
- Profils : **forensics**, **threat intel**, **threat hunting**, **malware analysis**.
    
- Attendu : **large culture des menaces** + capacité à gérer une compromission **sans dépendre** uniquement d’outils (EDR/SIEM).
    
- Job : **stressant + forte responsabilité**, mais **très formateur/récompensant**.
    

![[CIRT,CSIRT,CERT.png|697]]

|Exemple|Type|Portée|
|---|---|---|
|**JPCERT**|CERT national|Gestion d’incidents à l’échelle du Japon|
|**Mandiant**|Équipe privée|Réponse à incidents globale|
|**AWS CIRT**|Équipe interne fournisseur|Incidents de sécurité liés aux clients AWS|

---
### Rôles défensifs spécialisés

- Dans les grandes entreprises / startups tech / agences gov : rôles **très “narrow”**, très recherchés, mais demandent souvent une base solide (SOC/IT).
    

![[Specialized Def Roles.png]]

|Rôle|Focus|Mission clé|
|---|---|---|
|**Digital Forensics Analyst**|Forensics|Détecter/extraire des traces dans **disque & mémoire**|
|**Threat Intelligence Analyst**|Renseignement|Suivre **groupes émergents**, tendances, IOCs/TTPs|
|**AppSec Engineer**|Applicatif|Assurer un **SDLC sécurisé**|
|**AI Researcher**|IA & sécurité|Étudier les **menaces IA** et les défenses|

---
## Évolution de carrière SOC

#### Pourquoi démarrer en SOC L1

- Bon point d’entrée pour **élargir ta vision** de la cybersécurité et comprendre les rôles spécialisés.
    
- Même en junior : **vraies attaques**, protection contre des **groupes avancés**, grosse courbe d’apprentissage.
    

#### “SOC Path”

1. **Acquérir/pratiquer les compétences SOC** (un bagage IT + notions red team = bonus)
    
2. Être **proactif** : CTF, veille cyber, envisager la **certif SAL1**
    
3. **Préparer l’entretien** + comprendre **SOC interne vs MSSP** + postuler
    
4. Après un temps en junior : viser des **rôles plus seniors** (L2, etc.)
    

---
### SOC interne vs MSSP

> MSSP = prestataire qui fournit des services sécurité externalisés (souvent un SOC) à plusieurs clients.

|Aspect|SOC interne|MSSP|
|---|---|---|
|Exemple|SOC d’une banque → protège **la banque**|MSSP global → protège **~60 clients** en Europe|
|Rythme|Shifts souvent plus **calmes**|**Haute pression** : file d’alertes urgentes dès le début|
|Outils|Peu d’outils, mais à maîtriser **à fond**|Beaucoup d’outils / plateformes **très variés**|
|Exposition incidents|Peu de gros incidents (ex : 2/an)|**Très fréquent** (attaques/breaches quasi chaque semaine) → apprentissage rapide|

> En bref : **MSSP = accélérateur d’expérience**, mais plus intense.

---
### Next steps après L1

- Trajectoire “naturelle” : **SOC L2**
    
- Alternatives possibles selon ce qui te plaît :
    
    - **Engineering** (SIEM/EDR/détection)
        
    - **CIRT/IR** (réponse à incident)
        
    - **Management** → potentiellement jusqu’au **CISO**
        
- Priorité des **1–2 premières années** : obtenir de la **vraie expérience terrain** et l’exploiter au max.
    

---
### 4 règles d’or (SOC analyst)

- **Apprendre de chaque alerte**
    
- **Penser comme un attaquant**
    
- **Tout vérifier**
    
- **S’impliquer dans les incidents**

---

