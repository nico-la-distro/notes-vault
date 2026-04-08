
|Besoin / Question|Concept associé|
|---|---|
|Identifier un utilisateur de façon unique|**Identification**|
|Prouver qui il est|**Authentication**|
|Empêcher l’usurpation|**Mots de passe forts** + **MFA**|
|Définir ce qu’il peut accéder / faire + l’imposer|**Authorisation** + **Access Control**|
|Savoir ce qu’il fait et le tenir responsable|**Logging** + **Auditing** (→ **Accountability**)|

## Model IAAA

→ 4 étapes en chaîne : **Identifier → Authentifier → Autoriser → Rendre responsable (tracer)**  
But : renforcer **CIA** (Confidentialité, Intégrité, Disponibilité) et réduire **accès non autorisés / fuites / incidents**.

|Étape|Question à laquelle ça répond|Définition courte|Exemples|
|---|---|---|---|
|**Identification**|“Qui es-tu ?”|L’utilisateur **revendique** une identité via un identifiant **unique**|email, username, ID|
|**Authentication**|“Prouve-le”|Vérifier que l’utilisateur est bien celui qu’il prétend être|mot de passe, code email, MFA|
|**Authorisation**|“Qu’as-tu le droit de faire ?”|Déterminer les **droits d’accès/actions** selon privilèges|rôles/permissions, clearance|
|**Accountability**|“Qu’as-tu fait ?”|**Tracer** les actions pour attribuer la responsabilité|logs centralisés, audit incident|

- **Identification ≠ Authentication** : _dire qui tu es_ vs _le prouver_.
- **Authorisation** = “least privilege” : accès **uniquement** à ce qui est nécessaire.
- **Accountability** repose sur **logging + stockage centralisé** pour enquêter après incident.

---
## Identification

**Identification = “je déclare qui je suis”** (utilisateur / process / système).  
➡️ C’est **une revendication d’identité**, pas une preuve.

**Exemple**

- **Username** : tanderson, thomasa, thomas01, ta001, neo…
- **Numéros uniques** 
    - National ID / SSN
    - Student ID
    - Passport number
    - Mobile phone number
- **Email** (très fréquent) : souvent choisi car **unique**, évite de trouver/retinir un username unique.

|Concept|Ce que ça fait|Exemple|
|---|---|---|
|**Identification**|“Je suis X”|donner un nom / email / username|
|**Authentication**|“Prouve que tu es X”|montrer une carte d’identité, mot de passe, etc.|

## Authentification

**Authentication = vérifier / prouver l’identité** revendiquée lors de l’identification.

**Facteurs d'authentification :**

|Facteur|Définition|Exemples|
|---|---|---|
|**Something you know**|info mémorisée|password, passphrase, PIN, pattern|
|**Something you have**|objet possédé|téléphone/SIM (code SMS), clé de sécurité USB/NFC, générateur OTP|
|**Something you are**|biométrie|empreinte, face ID, rétine, voix|

- **TryHackMe** :
    - identification = username/email (unique)
    - authentication = password
    - login via Google : Google authentifie, puis “atteste” auprès de THM.
- **SMS code** : prouve que tu **possèdes** le numéro (SIM/eSIM).
- **Biométrie smartphone** : souvent + **backup** PIN/password.

**MFA / 2FA**

| Exemple            | Facteurs                      |
| ------------------ | ----------------------------- |
| ATM : carte + PIN  | **have** + **know** → **2FA** |
| Coffre : clé + PIN | **have** + **know** → **2FA** |

---

## Authorisation vs Access Control

- **Authorisation** = décide **ce que l’utilisateur a le droit de faire / accéder** (la _politique_).
- **Access Control** = **fait appliquer** cette décision (le _mécanisme_ / l’exécution).

| Concept            | Rôle                      | Question                                           | Exemples                                                                                                                                                       |
| ------------------ | ------------------------- | -------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Authorisation**  | Décision                  | “Qu’est-ce que tu as le droit de faire ?”          | membre du gym → utiliser machines aux heures d’ouverture ; Linda → accéder à _sa_ chambre ; Edward (sales) → fichiers sales                                    |
| **Access Control** | Application / enforcement | “Comment on l’empêche / l’autorise concrètement ?” | staff qui empêche de sortir un tapis roulant ; serrure + clé / carte RFID ; permissions sur fichiers ; mail server qui bloque l’accès aux boîtes des collègues |

---
## Accountability & Logging

**Accountability = pouvoir tenir un utilisateur responsable de ses actions** après **authentification + autorisation**.  
➡️ Possible uniquement si on peut **auditer**, donc si on a des **logs** fiables.

---
**Logging** = enregistrer les événements du système :
- actions utilisateur, événements système, erreurs
- permet de savoir **qui a accédé à quoi et quand**
- sert à : **compliance**, **incident response**, **forensics**

**Ce que les logs permettent**
- **Tracer** les actions d’un utilisateur
- **Détecter** anomalies / accès non autorisés
- **Alerter** (ex : tentative d’accès à des données sensibles)
- Repérer des patterns suspects : **échecs de login répétés**, **accès inhabituels**

**Sécuriser les Logs**
- Logs doivent être **corrects, sécurisés** et parfois **tamper-proof** (inaltérables)
- Objectif : empêcher un attaquant de **modifier/supprimer** les traces pour cacher ses actions
- Bonne pratique : **serveur de logs séparé** dédié au stockage → base du **log forwarding**

**Log forwarding**
- **envoyer les logs d’un système vers un autre** (souvent central)
- agrégation multi-sources → analyse/corrélation plus facile
- possible vers un service **cloud**

**SIEM**
- **agrège + analyse** les logs multi-sources pour détecter des menaces 
- identifie anomalies, incidents potentiels
- génère des **alertes** pour les équipes sécurité

|Usage|Pourquoi c’est utile|
|---|---|
|Détection / réponse|meilleure visibilité + alertes|
|Compliance reporting|données nécessaires aux audits|
|Forensic investigations|historique détaillé pour trouver source/cause d’un incident|

---

## Identity Management

**IdM = politiques + technologies** pour gérer :
- **Identification**
- **Authentication**
- **Authorisation**

**Fonctions typiques d’un IdM**
- Base **centralisée** des identités + droit    
- **User provisioning** : création/gestion des comptes
- **Authentication** + **Authorisation**
- Gestion/monitoring des accès (single “source of truth”)

---
### **IdM vs IAM**

**IAM** = Identity and Access Management (IAM) is a framework/process for controlling and securing digital identities and user access in organisations.

|Point|**IdM**|**IAM**|
|---|---|---|
|Portée|plutôt centrée sur **identités + attributs + permissions**|plus **global** : identités **+ gestion/sécurisation des accès**|
|Focus (selon certaines sources)|“gérer” utilisateurs/devices/groupes (attributs, permissions)|“évaluer” et **accorder/refuser** selon **policy**|
|Fonctions|provisioning, authN, authZ (et contrôle d’accès)|+ access control, **identity governance**, **compliance management**, monitoring|
|Techs souvent citées|—|**RBAC**, **MFA**, **SSO**|
|Cycle de vie|—|onboarding, offboarding, **revocation** + audit|

Note : certains utilisent **IdM et IAM comme synonymes**, la frontière est souvent **vague** (ici présentés comme distincts).

IAM = IdM + gestion plus large des accès + gouvernance + conformité + audit

---

## Attacks against Authentifcation

But : montrer pourquoi il faut **éviter les protocoles “maison”** et préférer des protocoles **éprouvés/testés** (peer review), car l’auth est pleine de pièges.

|Problème|Cause|Fix (principe)|
|---|---|---|
|Eavesdropping (phrase secrète)|secret entendu|mécanisme plus robuste (souvent crypto)|
|Sniffing cleartext|credentials en clair|canal/protocole sécurisé|
|Replay attack|réponse identique réutilisable|**nonce/timestamp** → réponse unique|

**À retenir** : _le chiffrement seul ne suffit pas si tu ne garantis pas la fraîcheur (freshness) de la réponse._

---

## Access Control Models

| Modèle   | Principe                                                    | Comment on attribue l’accès                               | Avantages                                                    | Limites / Quand l’utiliser                                          | Exemples                                                          |
| -------- | ----------------------------------------------------------- | --------------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------------- | ----------------------------------------------------------------- |
| **DAC**  | Le **propriétaire** de la ressource décide                  | Ajout **explicite** d’utilisateurs + permissions          | Simple, flexible pour petits groupes                         | **Difficile à scaler** quand beaucoup d’utilisateurs/roles changent | Partage d’un album photos à des comptes précis                    |
| **RBAC** | Accès basé sur le **rôle**                                  | Rôles → **groupes** ; permissions accordées au groupe     | **Scalable**, maintenance facile (ajouter/retirer du groupe) | Moins flexible pour cas très spécifiques                            | Comptable → accès compta, pas R&D                                 |
| **MAC**  | Politique **imposée par le système** (sécurité prioritaire) | Permissions très **restreintes**, contrôlées centralement | Très sécurisé, adapté données sensibles/classifiées          | Peu de liberté utilisateur (pas installer, pas changer permissions) | **AppArmor**, **SELinux** (Linux : Debian/Ubuntu, Red Hat/Fedora) |

**DAC** = Discretionary Access Control

**RBAC** = Role-Based Access Control

**MAC** = Mandatory Access Control

_**ressources** = https://www.apparmor.net/ / https://github.com/SELinuxProject_

---

## Single Sign-On (SSO)

**Problème :** au travail, un utilisateur doit accéder à plusieurs services (email, fichiers partagés, imprimantes…).  
→ Trop d’identifiants/mots de passe à gérer, surtout si on ne **réutilise pas** les mêmes passwords.

**SSO :** l’utilisateur s’authentifie **une seule fois** avec **un seul couple d’identifiants**, puis obtient l’accès aux autres services nécessaires.

| Point                 | Sans SSO                              | Avec SSO                               |
| --------------------- | ------------------------------------- | -------------------------------------- |
| Identifiants          | plusieurs comptes / passwords         | **1** compte / password principal      |
| Passwords forts       | difficile d’en retenir beaucoup       | plus réaliste d’en retenir **un fort** |
| MFA                   | à déployer sur chaque service (lourd) | MFA configuré **une fois**             |
| Support (reset, etc.) | dispersé sur plusieurs comptes        | plus simple (un seul compte)           |
| Productivité          | re-login fréquent                     | accès fluide sans se relogger partout  |