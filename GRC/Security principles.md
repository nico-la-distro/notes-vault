## CIA (and more)

| Concept                  | But (ultra court)           | Exemple / impact                                               |
| ------------------------ | --------------------------- | -------------------------------------------------------------- |
| **Confidentialité (C)**  | seuls les autorisés lisent  | fuite CB / dossier médical ⇒ fraude, sanctions                 |
| **Intégrité (I)**        | pas modifié (ou détectable) | adresse de livraison changée / dossier altéré ⇒ erreurs graves |
| **Disponibilité (A)**    | accessible quand nécessaire | site/app down / système clinique indispo ⇒ service bloqué      |
| **Authenticité**         | source vraie (pas fake)     | vérifier que la commande vient bien du client                  |
| **Non-répudiation**      | la source ne peut pas nier  | client/acteur ne peut pas dire “pas moi” (banque, achats…)     |
| **Utility** _(Hexad)_    | info utilisable             | données chiffrées sans clé ⇒ inutiles                          |
| **Possession** _(Hexad)_ | garder le contrôle          | vol backup / ransomware ⇒ perte de contrôle des données        |

---

## PIM / PAM

Pour gérer les accès à un SI, on définit les droits selon :
- **le rôle** de la personne dans l’organisation
- **la sensibilité** des données/systèmes

Le principe central reste : **least privilege**  
➡️ donner **le minimum de droits nécessaires**, rien de plus.

| Concept                                  | Rôle                                                                                       |
| ---------------------------------------- | ------------------------------------------------------------------------------------------ |
| **PIM (Privileged Identity Management)** | associe l’**identité / rôle métier** d’un utilisateur à un **rôle d’accès** sur le système |
| **PAM (Privileged Access Management)**   | gère les **privilèges** de ces rôles d’accès + applique des contrôles de sécurité          |

---
## DAD

| Élément                  | Cible (CIA)         | Définition ultra courte                      | Exemple (dossiers médicaux)                        | Note clé                                   |
| ------------------------ | ------------------- | -------------------------------------------- | -------------------------------------------------- | ------------------------------------------ |
| **Disclosure**           | **Confidentialité** | fuite / accès non autorisé                   | vol + publication de dossiers                      | attaque = “voir ce qui doit rester secret” |
| **Alteration**           | **Intégrité**       | modification non autorisée                   | dossier modifié ⇒ mauvais traitement               | peut être **mortel**                       |
| **Destruction / Denial** | **Disponibilité**   | destruction ou indisponibilité               | DB down ⇒ hôpital paralysé (même si retour papier) | “service inutilisable”                     |

---

| **DAD triad**            | Opposé de **CIA**   | Disclosure + Alteration + Destruction/Denial | —                                                  | protéger DAD = maintenir CIA               |
| ------------------------ | ------------------- | -------------------------------------------- | -------------------------------------------------- | ------------------------------------------ |
| **Trade-off CIA**        | —                   | trop de C/I ↓ A ; trop de A ↓ C/I            | —                                                  | sécurité = **équilibre**                   |

---

## Concepts of Security Models

| Modèle                  | But principal                   | Règles (traduction humaine)                                                                                                                                                        | Mémo                                      |
| ----------------------- | ------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------- |
| **Bell–LaPadula (BLP)** | **Confidentialité**             | **No read up** : un “bas niveau” ne lit pas du “haut niveau” \| **No write down** : un “haut niveau” n’écrit pas vers le bas (évite la fuite)                                      | **Read down / Write up**                  |
| **Biba**                | **Intégrité**                   | **No read down** : un “haut niveau d’intégrité” ne lit pas du “bas” (évite de se “contaminer”) \| **No write up** : un “bas” n’écrit pas vers le haut (évite de corrompre le haut) | **Read up / Write down**                  |
| **Clark–Wilson**        | **Intégrité (pratique métier)** | Données critiques = **CDI** ; entrées = **UDI** ; seules des **TP** (transactions) peuvent modifier CDI ; **IVP** vérifie l’état valide des CDI                                    | **Intégrité par transactions contrôlées** |

![[The Bell LaPadula Model.png]]

![[The Biba Model.png]]

**Clark-wilson**

- **Constrained Data Item (CDI)** : This refers to the data type whose integrity we want to preserve.
- **Unconstrained Data Item (UDI)** : This refers to all data types beyond CDI, such as user and system input.
- **Transformation Procedures (TPs)** : These procedures are programmed operations, such as read and write, and should maintain the integrity of CDIs.
- **Integrity Verification Procedures (IVPs)** : These procedures check and ensure the validity of CDIs.

---

**Other Sec Models**

- Brewer and Nash model
- Goguen-Meseguer model
- Sutherland model
- Graham-Denning model
- Harrison-Ruzzo-Ullman model

---
## ISO/IEC 19249:2017 — principes

| Catégorie             | Principe                                        | Idée clé                                                          | Exemple / lien                                                         |
| --------------------- | ----------------------------------------------- | ----------------------------------------------------------------- | ---------------------------------------------------------------------- |
| **Architectural (5)** | **Domain Separation**                           | regrouper composants en **domaines** avec mêmes attributs de sécu | rings x86 : kernel ring0 vs user ring3                                 |
|                       | **Layering**                                    | couches/niveaux → appliquer politiques + valider plus facilement  | OSI 7 couches ; API haut niveau masque syscalls (**defence in depth**) |
|                       | **Encapsulation**                               | cacher l’implémentation, accès via méthodes/API                   | OOP `increment()` vs accès direct ; API pour DB                        |
|                       | **Redundancy**                                  | augmente **dispo** + peut aider **intégrité**                     | double alim ; RAID5 (parité détecte altération)                        |
|                       | **Virtualization**                              | isolation/sandbox via partage hardware multi-OS                   | cloud, sandboxing, “detonation” malware                                |
| **Design (5)**        | **1. Least Privilege**                          | droits minimum nécessaires (“need-to-know”)                       | read sans write si juste consulter                                     |
|                       | **2. Attack Surface Minimisation**              | réduire points d’attaque (features/services)                      | désactiver services inutiles (hardening)                               |
|                       | **3. Centralized Parameter Validation**         | valider entrées **au même endroit**                               | limiter DoS / RCE via validation centralisée                           |
|                       | **4. Centralized General Security Services**    | centraliser services sécu (auth, etc.)                            | serveur d’auth central (attention SPOF)                                |
|                       | **5. Preparing for Error & Exception Handling** | gérer erreurs, **fail safe**, pas de leak                         | firewall crash ⇒ bloque tout ; erreurs sans fuite mémoire              |

object-oriented programming (OOP)

---

## Zero Trust vs Trust but Verify

| Concept                            | Idée clé                                                           | “Vérifier” comment ?                                    | Avantage                                                  | Limite / coût                            |
| ---------------------------------- | ------------------------------------------------------------------ | ------------------------------------------------------- | --------------------------------------------------------- | ---------------------------------------- |
| **Trust but Verify**               | on fait confiance **mais on contrôle**                             | logs + revue (souvent **automatisée** : proxy, IDS/IPS) | détecte comportements anormaux                            | impossible de tout vérifier manuellement |
| **Zero Trust**                     | la confiance = **vulnérabilité** → **jamais confiance par défaut** | authN + authZ **avant chaque ressource** (même interne) | limite l’impact d’un breach (containment), couvre insider | trop poussé = friction/impact business   |
| **Microsegmentation** (Zero Trust) | segments très petits (jusqu’à **1 host**)                          | trafic inter-segments : auth + ACL + contrôles          | réduit propagation latérale                               | complexité opérationnelle                |

AuthN = authentification

AuthZ = authorization

---

## Threat vs Risk

| Terme               | Définition (ultra court)                         | Exemple vitrine (verre)        | Exemple SI (hôpital)                                       |
| ------------------- | ------------------------------------------------ | ------------------------------ | ---------------------------------------------------------- |
| **Vulnérabilité**   | **faiblesse** exploitable                        | verre standard = fragile       | DB a une faille connue                                     |
| **Menace (Threat)** | **danger potentiel** lié à la vuln               | risque que le verre soit cassé | exploit PoC publié ⇒ menace **réelle**                     |
| **Risque (Risk)**   | **probabilité d’exploitation × impact business** | chance de casse + coût/impact  | chance d’attaque via PoC + impact patient/légal/opérations |

PoC = Proof of Concept

---
### STRIDE

| Lettre | Menace                     | Idée clé / contrôle typique                                               |
| ------ | -------------------------- | ------------------------------------------------------------------------- |
| **S**  | **Spoofing**               | usurpation d’identité → **authentification** (API keys, signatures, etc.) |
| **T**  | **Tampering**              | modification de données → mesures d’**intégrité** (anti-tamper)           |
| **R**  | **Repudiation**            | nier une action → **logs / traçabilité**                                  |
| **I**  | **Information Disclosure** | fuite d’info → bonne **segmentation/contrôle d’accès**                    |
| **D**  | **Denial of Service**      | épuisement de ressources → protections anti-abus / résilience             |
| **E**  | **Elevation of Privilege** | montée de privilèges → pire cas, mène souvent à compromission plus large  |

---

### 6 Phases de l'Incident Response

|Phase|But|
|---|---|
|**1. Preparation**|ressources, plans, procédures prêtes|
|**2. Identification**|confirmer la menace / l’attaquant|
|**3. Containment**|limiter la propagation / l’impact|
|**4. Eradication**|supprimer la menace active|
|**5. Recovery**|restaurer les systèmes / retour à la normale|
|**6. Lessons Learned**|tirer les leçons (ex : formation phishing)|

**P I C E R L** = **Prepare → Identify → Contain → Eradicate → Recover → Lessons learned**

---

## CSIRT

### Incident / Incident Response (IR)

- Une **violation** de sécurité = **incident**
- La réponse/remédiation = **Incident Response (IR)**
- Menée par un **CSIRT** (équipe préparée avec compétences techniques)

### Priorisation d’un incident

- **Urgency** = type d’attaque / vitesse de réponse nécessaire
- **Impact** = effet sur systèmes + opérations business

