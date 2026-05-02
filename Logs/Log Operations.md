# Log Configuration

“Do you dare to configure your logs, or are you happy to be lost in the madness of the thousands of lines?”

Log configuration = gestion des logs pour :
- sécurité
- stabilité opérationnelle
- conformité légale
- debug

Objectif : visibilité système + gestion ressources + détection + conformité

# Log Configuration Options

## SECURITY

Configuration orientée détection et réponse aux menaces.

Focus :
- détection anomalies / menaces
- logs authentification utilisateurs
- contrôle autorisations (access control)
- intégrité système + confidentialité données

## OPERATIONAL

Configuration orientée santé et performance système.

Focus :
- monitoring statut système/composants
- alertes et reporting
- troubleshooting
- capacity planning
- billing / facturation services

## LEGAL

Configuration orientée conformité réglementaire.

Focus :
- respect normes / lois / standards
- exemples : ISO 27001, COBIT, GDPR, PCI DSS, HIPAA, FISMA

Exemple PCI DSS :
- centralisation logs obligatoire
- configuration correcte des logs
- rétention 12 mois
- 3 derniers mois facilement consultables
- contrôles sécurité systèmes
- audits annuels

## DEBUG

Configuration orientée développement et tests.

Focus :
- visibilité accrue pour debugging
- amélioration efficacité dev
- accélération cycle de développement
- identification bugs et erreurs

Usage :
- surtout en dev/test
- rarement en production

---

# Where To Start After Deciding the Logging Purpose

## Where To Start and What To Do After Deciding the Log Purpose?

Point de départ :
- objectif + périmètre déjà définis
- utiliser réunions + brainstorming équipe

Rôle de la réunion :
- déclencher le brainstorming
- explorer plusieurs aspects
- produire un draft de plan

Principe clé :
- chaque objectif de logging est différent
- le questionnement guide l’identification des besoins

Étapes après les questions initiales :
- plan détaillé
- choix outils / technologies
- monitoring
- processus de review / analyse

## Questions To Ask In Planning Meeting/Session

| Domaine | Question |
|--------|----------|
| Scope | What will you log, and for what (asset scope and logging purpose)? |
| Requirements | Is additional commitment or effort required to achieve the purpose (requirements related to the purpose)? |
| Volume | How much are you going to log (detail scope)? |
| Volume | How much do you need to log? |
| Collection | How are you going to log (collection)? |
| Storage | How are you going to store collected logs? |
| Compliance | Is there a standard, process, legislation, or law that you must comply with due to the data you log? |
| Security | How are you going to protect the logs? |
| Analysis | How are you going to analyse collected logs? |
| Resources | Do you have enough resources and workforce to do logging? |
| Budget | Do you have enough budget to plan, implement and maintain logging? |

---

# Configuration Dilemma: Planning and Implementation

## Configuration Dilemma: Requirements, Aspirations, Resources, and Investment

Le dilemme de configuration = difficultés d’implémentation du logging.

Idée centrale :
- log configuration ≠ simple activation de logs
- c’est une décision structurée et contrainte

Chaque plan dépend de :
- scope
- assets
- objectifs
- exigences
- attentes

Décisions prises avec :
- sysadmins
- juristes / compliance
- finance / management

Problème principal :
- équilibre entre :
  - requirements
  - scope / niveau de détail
  - coût (argent + travail)
  - risques + investissement

Objectif des réunions :
- satisfaire exigences opérationnelles + sécurité (non négociable)
- évaluer amélioration possible via plus de données / insights

Approche recommandée :
- risk assessment complet
- prioriser sécurité + conformité + légal
- viser :
  - efficacité
  - résilience
  - approche proactive
  - durabilité

## Translating "Requirements" and "Aspirations" To Operational Level

| Base Requirements | Aspirations for Better Insights |
|------------------|----------------------------------|
| What happened? | Is it possible to have more data? |
| When did it happen? | More details |
| With time data (if possible) | How sure can I be that this is true? |
| Where did it happen? (network/system/path/interface) | What is affected? |
| Who/what caused it? | What will happen next? |
| From which log source? | Is there anything else that requires attention? |
| What should I do about the incident? | — |

## Interpretation des deux approches

Base Requirements :
- focus incident detection
- approche réactive
- bon pour threats connus
- fondation essentielle du logging

Aspirations :
- threat hunting mindset
- approche proactive
- plus de ressources nécessaires
- adapté aux menaces avancées/sophistiquées

Conclusion :
- base = indispensable pour détection incident
- aspirations = recommandées pour anticipation des menaces évolutives

---

# Principles and Difficulties

## Logging Principles

Logging = composant critique cyber-sécurité + IT ops  
Processus :
- coûteux en ressources
- indispensable pour efficacité + sécurité

### Collection
- définir le but du logging
- collecter uniquement ce qui est utile
- éviter données inutiles
- réduire le log noise

### Format
- log au bon niveau de détail
- format cohérent
- timestamps précis + synchronisés

### Archiving and Accessibility
- définir politiques de rétention
- stocker logs utiles pour analyse
- backups des logs + systèmes

### Monitoring and Alerting
- alertes pour événements importants
- privilégier alertes actionnables
- éviter bruit (noise)

### Security
- contrôle d’accès aux logs
- chiffrement si nécessaire
- utiliser solution dédiée log management

### Continuous Change
- logs évoluent constamment
- accepter changements continus
- formation des équipes

## Challenges

Les challenges font partie intégrante du log management.

### Data Volume and Noise
- multiples sources de données
- volumes de logs très variables
- certaines apps génèrent peu de logs
- autres génèrent volumes massifs
- bruit important → difficulté d’analyse

### System Performance and Collection
- collecte peut ralentir systèmes
- systèmes anciens/sensibles difficiles à modifier
- contraintes techniques héritées
- déploiement + optimisation complexes
- gestion versions agents à grande échelle difficile

### Process and Archive
- multiples formats de logs
- parsing long et sujet aux erreurs
- gestion rétention complexe
- contraintes compliance multiples

### Security
- sécurisation des logs = challenge constant

### Analysis
- corrélation multi-sources complexe
- nécessite ressources + expertise
- temps réel difficile
- faux positifs / faux négatifs difficiles à gérer

### Misc
- absence de planification / roadmap
- manque de budget
- manque de playbooks / exercices
- manque de compétences techniques
- focus excessif sur collecte vs analyse
- oubli du facteur humain et erreurs systèmes

## Where To Go From Here?

Points clés :
- principes à respecter
- challenges à anticiper
- approche proactive nécessaire
- adaptation selon contexte

---

# Common Mistakes and Best Practices

## Common Mistakes and Best Practices

Logging = outil critique cyber sécurité + IT ops  
Mais :
- sans planification → inefficace
- consommation de ressources élevée
- maintenance continue nécessaire

Principe clé :
- “if it works, don't touch it” = faux
- environnement + menaces évoluent
- configuration doit être mise à jour régulièrement

Actions de base :
- apprendre des erreurs
- suivre évolution des menaces sectorielles
- tester régulièrement (scope + résilience)
- suivre bonnes pratiques des experts

## Exemple réel (EternalBlue)

### Experience
- mauvaise configuration logging → analyse difficile

### Storyline
- Windows 7 default logging insuffisant
- vulnérabilité : EternalBlue (CVE-2017-0144)
- logs système, sécurité, application quasi inexistants lors compromission

### Attack Details
- impact : accès complet au système
- gravité : élevée
- CVSS 3 : 8.1 (High)

### Notes
- Windows 7 SP1 support terminé en 2020
- exploit utilisé en 2017 dans la nature

## Common Mistakes and Best Practices

### Mistakes ("don’ts")

- logging d’informations sensibles
- création de logs non planifiée / non contrôlée
- collecte sans analyse
- collecte de tout sans stratégie
- absence de configuration adaptée
- ignorer tests (scale / fonctionnalité / stabilité)
- analyser seulement une partie du système (ignorer l’interne)
- biais d’analyse :
  - chercher ce qu’on veut trouver
  - ignorer ce qui est réellement observé
- oublier que le logging = cycle complet (planification + gestion + analyse)

### Best Practices ("dos")

- créer un plan de logging adapté au système
- tester à l’échelle + stabilité + fonctionnalité
- ne pas logger de données sensibles
- sécuriser les logs
- créer alertes utiles et actionnables
- privilégier insights exploitables
- former les analystes
- maintenir et mettre à jour les systèmes et plans

---

