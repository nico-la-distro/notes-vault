- **Forensics (criminalistique)** : application de méthodes/procédures pour **enquêter** et **résoudre** des crimes.

- **Digital forensics** : branche dédiée aux **cybercrimes** (crimes commis **sur** ou **avec** un appareil numérique).

- **Objectif** : utiliser des **outils/techniques** pour **examiner** des devices, **trouver/analyser** des preuves, afin de soutenir des **actions légales**.

---
### Procédures clés en digital forensics

- Suivi de procédures tout au long de :
    1. **Collecte** des preuves
    2. **Stockage** (préservation)
    3. **Analyse**
    4. **Reporting** (rapport pour le légal)

## **Methodology**

**Process NIST (4 phases) - Digital Forensics**

|Phase|But|Actions clés|Exemple|
|---|---|---|---|
|**Collection**|Récupérer les données **sans altérer l’original**|Identifier tous les devices (PC, laptop, caméra, USB…), préserver l’intégrité, **documenter** précisément ce qui est saisi|Saisir tous les supports + tenir un inventaire/chaîne de possession|
|**Examination**|**Réduire/filtrer** la masse de données|Trier, extraire la donnée pertinente (par date, utilisateur, type de fichier, etc.)|Ne garder que les médias d’une **date/heure** précise|
|**Analysis**|**Interpréter** + relier les preuves|Corrélation entre sources, reconstruction **chronologique**, conclusions adaptées au scénario|Reconstituer les actions du suspect dans le temps|
|**Reporting**|Produire un **rapport exploitable légalement**|Méthodologie, résultats détaillés, parfois recommandations + **executive summary** adapté au public (police/management)|Rapport final remis aux autorités|

**Idée clé :** même si chaque cas demande des outils/techniques différents, le **cadre général NIST** reste le même.

---

**Types de Digital Forensics**

|Type|Cible|Exemples de preuves|
|---|---|---|
|**Computer forensics**|Ordinateurs (le plus fréquent)|fichiers, historiques, artefacts système|
|**Mobile forensics**|Smartphones/tablettes|appels, SMS, GPS, apps, etc.|
|**Network forensics**|Réseau (au-delà d’un seul device)|logs de trafic, captures, journaux réseau|
|**Database forensics**|Bases de données|traces d’intrusion, **modification** ou **exfiltration** de données|
|**Cloud forensics**|Données sur infra cloud|preuves souvent limitées côté client → enquête parfois plus complexe|
|**Email forensics**|Emails pro/perso|phishing, fraude, campagnes malveillantes, en-têtes, contenu, liens|

## **Evidence Acquisition** (preuves)

- **But** : collecter toutes les preuves **de façon sécurisée**, **sans modifier** les données originales.

- Les méthodes varient selon le **type d’appareil**, mais il existe des **bonnes pratiques générales**.

---

**Bonne pratiques**

|Pratique|Pourquoi c’est crucial|À retenir|
|---|---|---|
|**Proper Authorization (autorisation)**|Sans autorisation, la preuve peut être **irrecevable au tribunal** ; les données sont souvent **privées/sensibles**|Toujours obtenir l’accord / mandat des autorités compétentes avant de collecter|
|**Chain of Custody (chaîne de possession)**|Évite perte/altération “non attribuable” ; prouve **intégrité** et **fiabilité** en justice|Document officiel qui trace _qui a eu quoi, quand, où, et pourquoi_|
|**Write Blockers**|Empêche la station forensic (tâches en arrière-plan) de **modifier** le disque (ex : timestamps) → évite analyses faussées|Utiliser un write blocker lors de la collecte sur disque pour garder l’original **inchangé**|

## **Windows Forensics**

- **Preuves fréquentes** : PC fixes + laptops (souvent impliqués dans des crimes).
- Objectif : **acquisition** + **analyse** sur **Windows**.
- En phase de collecte, on prend des **forensic images** = copies **bit-à-bit** de l’OS/support.

---

**Types d'images forensic (windows)

| Type d’image     | Contenu                             | Nature des données                    | Exemples                                                      | Priorité                             |
| ---------------- | ----------------------------------- | ------------------------------------- | ------------------------------------------------------------- | ------------------------------------ |
| **Disk image**   | Données du stockage (HDD/SSD, etc.) | **Non-volatile** (reste après reboot) | fichiers, documents, médias, historique de navigation, etc.   | Après la RAM (si machine allumée)    |
| **Memory image** | Données de la **RAM**               | **Volatile** (perdu si arrêt/reboot)  | fichiers ouverts, processus en cours, connexions réseau, etc. | **À faire en premier** (sinon perdu) |
**À retenir :** si le système est allumé → **capturer la mémoire d’abord**, puis le disque.

---

## **Outils forensic**

**Outils courants (windows)**

| Outil          | Sert à                                 | Points clés                                                                                                                    |
| -------------- | -------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| **FTK Imager** | **Disk imaging** + analyse basique     | Très utilisé, interface graphique, plusieurs formats, acquisition + analyse                                                    |
| **Autopsy**    | Analyse de **disk image** (plateforme) | Open-source, analyse étendue : recherche mots-clés, récup fichiers supprimés, métadonnées, détection extension ≠ contenu, etc. |
| **DumpIt**     | **Memory imaging**                     | Acquisition RAM via CLI, plusieurs formats                                                                                     |
| **Volatility** | Analyse de **memory image**            | Open-source, plugins par artefact, support Windows/Linux/macOS/Android                                                         |

**Note** : il existe d’autres outils, mais ceux-ci sont “classiques” pour disk/memory sur Windows.

---

|Outil|Sert à|
|---|---|
|`pdfinfo`|Lire les métadonnées d’un PDF|
|`exiftool`|Lire (et écrire) les métadonnées/EXIF d’images (et autres fichiers)|
