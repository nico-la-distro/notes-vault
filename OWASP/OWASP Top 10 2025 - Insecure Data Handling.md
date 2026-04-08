## A04 - Cryptographic Failures

Échec cryptographique = **données sensibles mal protégées** à cause de :

- **absence de chiffrement**
    
- **implémentation incorrecte**
    
- **mesures de sécurité insuffisantes**
    
- **algos faibles / obsolètes**
    
- **clés exposées**
    
- **transmission non sécurisée**

---

|Problème|Exemple|Risque|
|---|---|---|
|Mot de passe non protégé|stocké en clair / sans hash|compromission immédiate|
|Algo obsolète/faible|MD5, SHA1, DES|cassable / collisions|
|Clé exposée|clé dans le code / repo|chiffrement inutile|
|Données en transit non sécurisées|pas de TLS|interception (MITM)|
|“On fait notre crypto maison”|algorithme custom|failles imprévisibles, non audité|

---

**Prévention**

|Objectif|Bonne pratique|
|---|---|
|Protéger les mots de passe|**hash lent + sel** : bcrypt / scrypt / Argon2|
|Chiffrer correctement|**algos + libs standards** (audités), pas de crypto maison|
|Gérer les secrets|**jamais de credentials/keys dans le code/config/repo**|
|Stockage des clés|utiliser **KMS / vault / env secrets** (gestion de secrets)|
|Données en transit|sécuriser via **TLS**|

---

## A05 - Injection

**Injection = entrée utilisateur mal gérée** → l’app **injecte** cette entrée dans un moteur **exécutable** (DB, shell, template, API…) au lieu de la traiter de façon sûre.

---
**Def / Mécanisme**

|Étape|Ce qui se passe|Pourquoi c’est dangereux|
|---|---|---|
|1|l’app reçoit un input user|input = **non fiable**|
|2|l’app le réutilise tel quel (ou concatène)|l’input devient du **code**|
|3|le système cible exécute (requête/commande/template)|exécution non prévue / fuite / RCE|

---

**Exemples classiques**

|Type|Cible|Effet typique|
|---|---|---|
|SQL Injection|base de données|bypass login, dump données|
|Command Injection|OS / shell|exécuter commandes système|
|AI Prompt Injection|modèle / agent|détourner instructions, exfiltration|
|SSTI|moteur de templates|exécution code via template, RCE|

---

**Prévention**

|Risque|Mesure efficace|
|---|---|
|SQLi via concaténation|**requêtes paramétrées / prepared statements**|
|Command injection via shell|**éviter l’appel au shell**, utiliser **API safe**|
|Input dangereux|**validation stricte** (types, formats, whitelist)|
|Caractères spéciaux|**escape / sanitisation** (selon contexte)|
|Input traité trop tard|**filtrer avant traitement** (dès la réception)|

---

## A08 - Software & Data Integrity Failures

Failles d’intégrité = l’app **fait confiance** à du **code / updates / data** supposés “safe” **sans vérifier** : _authenticité, intégrité, origine_.

|Confiance excessive sur…|Exemple|Risque|
|---|---|---|
|Mises à jour|update appli acceptée sans vérif|supply chain, backdoor|
|Ressources externes|scripts/config chargés depuis source non fiable|exécution code / config malveillante|
|Données impactant la logique|data utilisée pour décider (règles, droits, flux) sans validation|contournements logiques|
|Artefacts “fichiers”|binaires, templates, JSON acceptés sans vérifier altération|injection, RCE, corruption|

---

**Prévention**

| Principe                         | Mise en pratique                                                               |
| -------------------------------- | ------------------------------------------------------------------------------ |
| Définir des **trust boundaries** | identifier ce qui doit être **trusted** vs **untrusted**                       |
| **Ne jamais supposer légitime**  | vérifier _origine + intégrité_ avant usage                                     |
| Vérifier l’intégrité             | **checks cryptographiques** (ex : checksum) sur packages                       |
| Limiter qui peut modifier        | seuls **sources/acteurs de confiance** modifient les artefacts critiques       |
| Sécuriser la chaîne de build     | intégrer ces contrôles dans le **CI/CD** (pipeline, artefacts signés/vérifiés) |