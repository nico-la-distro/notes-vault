# AS02 - Security Misconfigurations

- **Mauvaise config de sécurité** = système/serveur/app déployé avec **paramètres dangereux** (valeurs par défaut, réglages incomplets, services exposés).
    
- Ce ne sont **pas des bugs de code**, mais des **erreurs de déploiement / environnement / réseau**.
    
- Effet : **points d’entrée faciles** pour un attaquant.

---
#### Pourquoi c’est critique

- Une “petite” mauvaise config peut suffire à :
    
    - **exposer des données sensibles**
    - permettre une **élévation de privilèges**
    - donner un **premier accès (foothold)** au SI

- Les stacks modernes (cloud, APIs, tiers) multiplient les risques : **un seul élément mal configuré** peut compromettre tout le reste.

**Exemple**

|Cas|Erreur|Impact|
|---|---|---|
|Uber (2017)|**Bucket AWS S3 public** (backup)|Téléchargement direct de données sensibles **sans identifiants** (drivers + riders)|

---

**Patterns fréquents**

| Pattern                                                          | Ce que ça implique                    |
| ---------------------------------------------------------------- | ------------------------------------- |
| **Identifiants par défaut** / mdp faibles                        | accès trivial admin / services        |
| **Services/endpoints inutiles exposés Internet**                 | surface d’attaque ↑                   |
| **Stockage cloud mal configuré** (S3 / Azure Blob / GCP buckets) | fuite de données / exfiltration       |
| **API trop ouverte** (authN/authZ manquante)                     | accès non autorisé / actions abusives |
| **Erreurs verbeuses** (stack traces, infos système)              | fuite d’infos pour attaquer mieux     |
| **Logiciels/frameworks/containers obsolètes**                    | vulnérabilités connues exploitables   |
| **Endpoints AI/ML exposés sans contrôle**                        | abus / extraction / pivot             |

---

**Prévention**

|Mesure|Objectif|
|---|---|
|**Hardening** + supprimer features/services inutiles|réduire la surface d’attaque|
|**Auth forte** + **least privilege**|limiter l’impact d’un accès|
|**Limiter l’exposition réseau** + segmentation|éviter le “tout accessible”|
|**Patcher / maintenir à jour**|fermer les failles connues|
|**Masquer stack traces / infos système**|éviter l’info leak|
|**Audits réguliers cloud perms/config**|détecter dérives & erreurs|
|**Sécuriser endpoints AI/automation** + monitoring|empêcher abus & détecter|
|**Revues de config + checks auto en CI/CD**|empêcher l’erreur d’arriver en prod|

---
# AS03 - Software Supply Chain Failures

- **Faille de supply chain logicielle** = ta sécurité dépend de **composants externes** (libs, services, packages, modèles IA, datasets) qui sont **compromis / obsolètes / non vérifiés**
    
- Le problème n’est pas “ton code”, mais **ce que tu importes / déploies / mets à jour**.

---
#### **Pourquoi c’est critique**

- Les apps modernes = empilement de **tiers** (packages, APIs, modèles IA).
    
- **Une seule dépendance compromise** peut compromettre tout le système **sans toucher ton code**.
    
- Attaques souvent **automatisées + à grande échelle** → **détection difficile** + gros impact.

**Exemple**

|Cas|Vecteur|Point faible réel|Impact|
|---|---|---|---|
|SolarWinds Orion (2021)|code malveillant injecté dans une **mise à jour “de confiance”**|**build / vérif / distribution** des updates|milliers d’orgas infectées via installation automatique|

IA : risque similaire si on utilise des **modèles/datasets non vérifiés** → comportements cachés, **backdoors**, sorties biaisées, fuites de données.

---

**Patterns fréquents**

|Pattern|Risque|
|---|---|
|dépendances **non maintenues / non vérifiées**|code vulnérable ou malveillant|
|updates installées **automatiquement sans vérif**|propagation rapide d’un package compromis|
|confiance excessive en **modèles IA tiers** sans audit|backdoors / exfiltration / dérives|
|CI/CD **pas sécurisé** (tampering possible)|injection avant prod|
|tracking faible **licence/provenance**|composant douteux / non conforme|
|pas de suivi vulnérabilités **après déploiement**|exposition prolongée|

---

**Protection**

|Action|But|
|---|---|
|**Vérifier** composants/libs/modèles IA avant usage|réduire l’entrée de code malveillant|
|**Monitor + patch** dépendances régulièrement|corriger vite les CVE|
|**Signer / vérifier / auditer** packages & updates|garantir intégrité + provenance|
|**Verrouiller CI/CD** & build process|empêcher le tampering|
|tracer **provenance + licences**|confiance + conformité|
|**Runtime monitoring** (comportement anormal deps/IA)|détecter backdoors/dérives|
|**Threat modeling supply chain** dans le SDLC|couvrir test → déploiement → updates|

---
# AS04 - Cryptographic Failures

**Cryptographic failures** = chiffrement **absent** ou **mal utilisé** :

- algos faibles/dépréciés
- clés/secrets **hardcodés**
- mauvaise gestion/rotation des clés
- données sensibles non chiffrées (repos / transit)

---

#### Pourquoi c’est critique

- La crypto est partout : **trafic réseau**, **données stockées**, **auth/identité**, **secrets**.
    
- Si elle échoue → fuite de **mdp**, **tokens**, **PII** → **account takeover** ou **breach**.

- Exploitation typique :
    
    - **MITM** (si TLS faible/invalide)
    - **brute-force** sur clés faibles
    - secrets “trouvés” car **jamais protégés** (hardcodés / en clair)

---

**Patterns fréquents**

|Pattern|Exemple / impact|
|---|---|
|algos faibles/dépréciés|**MD5**, **SHA-1**, mode **ECB** → cassable / fuites de patterns|
|secrets hardcodés|clés/API tokens dans code/config → compromission directe|
|mauvaise rotation/gestion|clés jamais renouvelées → fenêtre d’attaque longue|
|pas de chiffrement repos/transit|DB / backups / HTTP → lecture en clair|
|certificats TLS self-signed / invalides|MITM facilité, confiance brisée|
|IA/ML sans gestion de secrets|paramètres / inputs sensibles exposés|

---

**Prévention**

|Mesure|But|
|---|---|
|algos modernes (**AES-GCM**, **ChaCha20-Poly1305**) + **TLS 1.3** + certs valides|confidentialité + intégrité solides|
|KMS / vault (**AWS KMS**, **Azure Key Vault**, **HashiCorp Vault**)|stockage/accès secrets sécurisé|
|rotation régulière (crypto periods)|limiter l’impact en cas de fuite|
|politiques + procédures de cycle de vie des clés|éviter l’improvisation / erreurs|
|inventaire certs/clés + owners|gouvernance, expiration maîtrisée|
|IA/agents : jamais de secrets en clair|éviter fuites via logs/outputs|

---

# AS06 - Insecure Design

- **Insecure design** = défauts **d’architecture / logique métier / frontières de confiance** intégrés dès le départ.
    
- Causes typiques : **pas de threat modeling**, pas d’exigences sécu, pas de revue design, erreurs d’hypothèses.
    
- Avec l’IA : le design devient fragile si on **suppose** que le modèle est “safe/prévisible” ou que son code est “correct”.

---
#### Pourquoi c’est critique

- **Tu ne “patches” pas un mauvais design** : c’est dans le workflow + la logique + le modèle de confiance.
    
- Corriger = **repenser** décisions, contrôles, limites (et désormais comment l’IA décide/agit).

---

**Exemple**

|Cas|Hypothèse de design|Faiblesse|Résultat|
|---|---|---|---|
|Clubhouse (début)|“Les users passent par l’app mobile”|API backend **sans vraie auth**|requêtes directes → données users/rooms + “privé” qui s’écroule|

---

**Designs insécures courants (2025)**

|Catégorie|Exemple|
|---|---|
|logique métier faible|recovery / approval flows contournables|
|hypothèses fausses|comportement user/modèle “raisonnable”|
|IA avec autorité excessive|LLM/agent qui exécute sans limites|
|pas de guardrails LLM|pas de règles, pas de validation|
|bypass test/debug en prod|chemins de contournement oubliés|
|pas d’abuse-case review|pas d’analyse “comment abuser” + pas de threat modeling IA|

---

**Insecure design à l'ère IA**

|Risque IA|Mécanisme|Impact|
|---|---|---|
|**Prompt injection**|input user mélangé au prompt système|hijack contexte, extraction de données|
|**Blind trust output**|le système agit sur la réponse IA sans validation|actions dangereuses / erreurs coûteuses|
|**Poisoned models**|modèle/dataset non vérifié ou fine-tuning unsafe|comportements cachés, backdoors|

---

**Comment concevoir "secure by design"**

|Principe|Actions concrètes|
|---|---|
|Modèle = **non fiable par défaut**|traiter toute sortie IA comme non sûre|
|Validation I/O|filtrer/valider **inputs + outputs** (intégrité, exactitude)|
|Séparer système vs user|**ne jamais fusionner** prompts système et contenu user|
|Données sensibles minimisées|éviter secrets dans prompts; contrôles stricts si nécessaire|
|Humain dans la boucle|**review humaine** pour actions IA à risque|
|Observabilité IA|provenance modèle, monitoring, (privacy/diff privacy si pertinent)|
|Threat modeling IA|prompt attacks, inference risks, misuse agents, supply chain|
|Threat modeling continu|à **chaque étape** (pas juste au début)|
|Exigences sécu par feature|définir avant d’implémenter|
|Least privilege partout|users/APIs/services|
|AuthN/AuthZ/sessions solides|cohérence sur tout le système|
|Dépendances vérifiées + à jour|supply chain maîtrisée|
|Tests/monitoring continus|logique, abuse paths, risques émergents (nouvelles features/IA)|

