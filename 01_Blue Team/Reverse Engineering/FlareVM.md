FlareVM (“**Forensics, Logic Analysis, and Reverse Engineering**”) est une **VM/outillage** Windows regroupant une collection **complète et organisée** de tools pour :

- **Reverse engineering**
- **Analyse malware**
- **Incident response**
- **Forensics**
- **Pentest**

Elle est maintenue par la **FLARE Team (FireEye)** et sert à **comprendre le comportement des malwares** et **analyser des exécutables** (digital forensics / reverse) dans un environement Windows.

---

## **Tools**

### **Reverse Eng & Debugging**

**Reverse** = comprendre “à l’envers” un binaire fini. **Debug** = exécuter/pas-à-pas pour comprendre et corriger.

| Outil        | Usage principal                       |
| ------------ | ------------------------------------- |
| Ghidra       | Suite RE open-source (NSA)            |
| x64dbg       | Debugger x86/x64 (opensource)         |
| OllyDbg      | Debug assembly (classique)            |
| Radare2      | Plateforme RE open-source (puissante) |
| Binary Ninja | Désassemblage + décompilation         |
| PEiD         | Détection packer/cryptor/compiler     |

---

### **Disassemblers & Decompilers**

Rendre le binaire **lisible** (ASM / pseudo-code) pour comprendre logique + flow.

| Outil        | Usage principal                              |
| ------------ | -------------------------------------------- |
| CFF Explorer | Éditeur/analyse **PE** (Portable Executable) |
| Hopper       | Debugger + disassembler + decompiler         |
| RetDec       | Décompilateur open-source (machine code)     |

---

### **Static & Dynamic Analysis**

**Statique** = sans exécuter. **Dynamique** = observer à l’exécution.

|Outil|Type|Usage principal|
|---|---|---|
|Process Hacker|Dynamique|Process watcher + édition mémoire|
|PEview|Statique|Viewer/analyse PE|
|Dependency Walker|Statique|Dépendances DLL d’un exécutable|
|DIE (Detect It Easy)|Statique|Détection packer/compiler/cryptor|

---

### **Forensics & Incident Response**

Forensics = **collecter/analyser/préserver** la preuve. IR = **détecter/contenir/éradiquer/récupérer**.

|Outil|Usage principal|
|---|---|
|Volatility|Analyse dump RAM (memory forensics)|
|Rekall|Memory forensics (IR)|
|FTK Imager|Acquisition + analyse d’images disque|

---

### **Network Analysis**

Étudier le réseau : trafic, cartographie, interactions.

|Outil|Usage principal|
|---|---|
|Wireshark|Capture + analyse de protocoles|
|Nmap|Scan / mapping réseau (détection de services)|
|Netcat|Lecture/écriture sur connexions réseau (outil “couteau suisse”)|

---

### **File Analysis**

Examiner / éditer des fichiers (souvent en binaire/hex).

|Outil|Usage principal|
|---|---|
|FileInsight|Inspection + édition binaire|
|Hex Fiend|Hex editor léger/rapide|
|HxD|Hex editor (view/edit binaire)|

---

### **Scripting & Automation**

Automatiser tâches répétitives (PowerShell/Python).

|Outil|Usage principal|
|---|---|
|Python|Modules/outils d’automatisation|
|PowerShell Empire|Framework post-exploitation PowerShell|

---
### **Sysinternals Suite**

Utilitaires Windows avancés (diagnostic / monitoring).

|Outil|Usage principal|
|---|---|
|Autoruns|Programmes lancés au boot / logon|
|Process Explorer|Infos détaillées sur processus|
|Process Monitor|Logs temps réel activité process/thread|

---
## **Commonly Used Tools for Investigation (room)**

| Outil                         | À quoi ça sert (valeur enquête)                                                | Quand l’utiliser                                            |
| ----------------------------- | ------------------------------------------------------------------------------ | ----------------------------------------------------------- |
| **Procmon** (Process Monitor) | Trace **activité système** en temps réel : fichiers, registre, process/threads | “Qu’est-ce qui écrit/ouvre quoi ?” / comportements suspects |
| **Process Explorer**          | Vue détaillée des processus : **parent/enfant**, DLL chargées, chemin          | “Quel process a spawn celui-là ?” / chaîne d’exécution      |
| **HxD**                       | **Hex editor** (fichiers/mémoire) pour inspecter/altérer                       | Identifier vrai type de fichier / headers / corruption      |
| **Wireshark**                 | Analyse trafic réseau (protocoles, IP/ports, flux)                             | Exfil/C2, connexions bizarres, protocole inattendu          |
| **CFF Explorer**              | Infos PE + **hashes** (MD5/SHA1), vérif intégrité/altérations                  | Vérifier un binaire, repérer anomalies sur PE               |
| **PEStudio**                  | **Static analysis** sans exécuter : propriétés, indicateurs suspects           | Premier tri “clean vs louche”                               |
| **FLOSS**                     | Extraction + **désobfuscation de strings**                                     | Trouver URLs, IP, paths, clés, config, API…                 |

---

## **Analysis malicious files**

### Scénario

- Un utilisateur a téléchargé **`windows.exe`** le **09/24/2024 à 03:43 AM** (flagged menace).
- Fichier dispo : `C:\Users\Administrator\Desktop\Sample`
- Approche : **d’abord statique**, puis (selon besoin) **dynamique**.

---

### **Analyse statique - `windows.exe`**

#### **PEStudio : signaux utiles**

![[FlareVM - analysis malicious file.png]]

![[FlareVM Analysis malicious file 2.png]]

|Élément observé|Interprétation / pourquoi c’est intéressant|
|---|---|
|Hashes **MD5** `9FDD4767DE5AEC8E577C1916ECC3E1D6` / **SHA1** `A1BC55A7931BFCD24651357829C460FD3DC4828F`|À comparer à VirusTotal / bases internes → si inconnu : possible **campagne récente**|
|Description “REGEDIT / Registry Editor”|Probable **leurre** (social engineering / masquer l’intention)|
|Emplacement (download/user folder) ≠ `C:\Windows\System32`|**Anormal** pour un regedit légitime → suspicion renforcée|
|Métadonnées en **russe** (ex: “Редактор реестра”)|Suspicion si org non russophone (indice d’origine / campagne)|
|**Rich header absent**|Indice possible de **packing/obfuscation** (évasion statique)|
|Imports (IAT) + tri “blacklist”|Donne une idée du **comportement potentiel** (capabilités)|


---
#### Imports/indices comportement (extraits)

![[FlareVM - analysis malicious file 3.png]]

|API / Indice|Ce que ça suggère|
|---|---|
|`UseShellExecute`|Peut **spawn** d’autres processus via le shell (chaîne d’exécution)|
|`CryptoStream`, `RijndaelManaged`, `CipherMode`, `CreateDecryptor`|Usage **AES/Rijndael** → chiffrement comms/fichiers, potentiellement logique ransomware ou config chiffrée|

---

#### **Strings — FLOSS sur `windows.exe`**

**Commande**

`FLOSS.exe .\windows.exe > windows.txt`

```powershell
PS C:\Users\Administrator\Desktop\Sample > FLOSS.exe .\windows.exe > windows.txt
WARNING: floss: .NET language-specific string extraction is not supported yet
WARNING: floss: FLOSS does NOT attempt to deobfuscate any strings from .NET binaries
INFO: floss: disabled string deobfuscation
INFO: floss: extracting static strings
INFO: floss: finished execution after 0.34 seconds
INFO: floss: rendering results
```

![[FlareVM -analysis malicious file 4.png]]

**Résultat / lecture**

|Message FLOSS|À retenir|
|---|---|
|“.NET string extraction not supported / no deobfuscation for .NET”|FLOSS est **limité sur .NET** : il sort surtout du **statique** et ne déobfusque pas|
|Dans `windows.txt`, on retrouve des strings/imports déjà vus|Cohérent avec PEStudio → recoupe utile, mais pas forcément révélateur si obfusqué/dynamique|

---

### Analyse dynamique réseau — `cobaltstrike.exe` (Process Explorer + Procmon)

#### Objectif

Déterminer si le binaire tente une connexion vers un **C2** (command-and-control).

#### Process Explorer (procexp)

| Action                         | Ce que tu récupères                                                              |
| ------------------------------ | -------------------------------------------------------------------------------- |
| Lancer `cobaltstrike.exe`      | Confirmer chaîne **parent/enfant** (souvent `Explorer.exe` → `cobaltstrike.exe`) |
| Identifier le **PID** ici 4108 | Sert à recouper avec d’autres outils                                             |
| Propriétés → onglet **TCP/IP** | Vois **destination(s)** + état de connexion                                      |
![[FlareVM - analysis malicious file 5.png]]

![[FlaveVM - analysis malicious file 6.png]]
#### Procmon (Procmon) — validation (ne pas dépendre d’un seul outil)

![[FlareVM - analysis malicious file 7.png]]

Defanged IP : 47[.]120[.]46[.]210 (pour éviter que ce soit interprété)

**Problème :** trop de bruit → il faut filtrer.  
**Filtre (CTRL+L ou icône filtre) :**

1. **Process Name**
2. **contains**
3. valeur : `cobalt`
4. **Include**
5. **Add** → **Apply**

✅ Résultat : confirmation d’une connexion vers **`47.120.46.210`**.

