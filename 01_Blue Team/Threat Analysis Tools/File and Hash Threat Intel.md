## Scenario

Contexte : Lundi d'avril, veille de release majeure chez Try Daily. Rôle : Analyste L1 (supervisé par un L2). Déclencheur : EDR flag plusieurs binaires sur plusieurs workstations lors d'un sweep d'alertes routinier. Input : Package de triage curié contenant les samples. Objectif : En 60 min, déterminer si les fichiers sont **bait**, **benign** ou **malicious**, avec preuves à l'appui.

---

## Filenames and Paths

Les chemins et noms de fichiers sont les premiers heuristiques disponibles. Seuls, ils ne prouvent pas la malveillance, mais révèlent des patterns d'attaque.

### Filepath Analysis

**Emplacements suspects fréquents :**

|Path|Raison|
|---|---|
|`C:\`|Cible pour mécanismes de persistance|
|`C:\Users\Public`|Accès cross-user pour outils adversaires|
|`C:\Users\Public\Public Downloads`|Répertoire à fort trafic, surveillance réduite|
|`C:\Windows\Temp\`|Payloads éphémères|
|`C:\ProgramData\`|Persistance furtive (writable system path)|

### Filename Heuristic Indicators

|Technique|Exemple|Mécanisme|
|---|---|---|
|Double extension|`invoice.pdf.exe`|Exploite le masquage d'extensions Windows|
|System binary impersonation|`scvhost.exe`|Abus de la familiarité avec les processus système|
|High-entropy string|`jh8F21.exe`|Packing automatisé / génération polymorphique|
|Masquerading|`backup-2300.exe`|Se fond dans les fichiers légitimes|

> Pour la **system binary impersonation** : créer une allowlist basée sur le chemin légitime du binaire, pas uniquement son nom.

---

## File Hash Lookup

Le hash (SHA256 / MD5) identifie un fichier de manière immuable, peu importe son nom. Indispensable car les attaquants renomment constamment leurs malwares.

**Génération du hash :**

bash

```bash
# Windows - CMD
certutil -hashfile bl0gger.exe SHA256

# Windows - PowerShell
Get-Filehash -Algorithm SHA256 bl0gger.exe

# Linux
sha256sum bl0gger.exe
```

**Bonnes pratiques :**

- Stocker les hashes en minuscules
- Hasher l'archive ET le binaire extrait
- Toujours contextualiser (où / quand rencontré)
- Tout changement d'un octet -> hash différent

### Analysis With VirusTotal

[VirusTotal](https://www.virustotal.com) -> agrège les résultats de dizaines d'antivirus en un seul rapport.

|Section|Question clé|Red flags|Notes analyste|
|---|---|---|---|
|Detection Score|Combien de vendors détectent le fichier ?|5+ vendors solides ; classifications contradictoires|Nouveau malware = faible détection initiale -> rechecke après 24h|
|Upload Time|Première soumission ?|>10 détections après 7 jours ; spike soudain|48-72h pour analyse complète vendor|
|Signatures|Fichier signé correctement ?|Cert invalide / manquant / émis à une entité non liée|Les certs valides peuvent être volés|
|Properties|Anomalies dans les métadonnées ?|Compile timestamp heure bizarre (ex: 3h AM) ; entropie >7.5 sur fichier non-média|UPX légitime augmente aussi l'entropie|
|Relations|Infrastructure réseau ?|IPs/domaines connus malveillants ; domaines DGA (ex: `xk8f92.xyz`)|Vérifier les IPs dans Shodan|
|Behavioral|Actions post-exécution ?|Modification registres critiques ; process injection|Corréler avec les logs endpoint|

### Cross-Reference With MalwareBazaar

[MalwareBazaar](https://bazaar.abuse.ch) -> base de données de samples malware + intelligence.

**Fonctionnalités clés :**

- **Malware Family tagging** : un fichier à 5/70 sur VT mais tagué `#IcedID` sur MalwareBazaar -> traiter comme malveillant
- **YARA rule integration** : noter les règles des submissions -> les ajouter à l'EDR/SIEM
- **Campaign attribution** : tags type `#TA551` -> lien avec des threat actors connus

**Recherche en production :**

```
sha256:<file_hash>
```

---

## Sandbox Analysis

### Dynamic Analysis for SOC

L'analyse statique (hash, strings) donne l'identité. Le sandbox exécute le fichier dans une VM instrumentée pour capturer processus, registry writes et paquets réseau.

**3 usages :**

- Confirmer l'exécution (rien ne se passe -> possible decoy)
- Extraire les IOCs runtime (domaines, mutexes, payloads droppés)
- Mapper vers ATT&CK (auto-labellisé par la plupart des sandboxes)

> L1 : utilise la lookup par hash. L2/RE : tâches avancées (memory dumps, detection engineering).

### Sandboxing Tools

|Outil|Focus|Usage|
|---|---|---|
|[Hybrid Analysis](https://hybrid-analysis.com/)|Behaviour trees + MITRE ATT&CK heatmap|Executive summary rapide|
|[Joe Sandbox](https://www.joesandbox.com/)|System calls, strings, memory dumps|Reverse engineers / detection engineers|

### Sandboxing Limitations

**1. Sandbox Evasion**

- Environment awareness checks : le malware détecte les signes de virtualisation
- Anti-debugging : détection de debugger, vérification des hardware IDs

**2. Temps d'exécution limité**

- Analyse stoppée après 2-5 min -> malware multi-stage ou time-delayed non détecté
- -> Cross-référencer avec d'autres sources TI

**3. Trafic chiffré / obfusqué**

- SSL/TLS souvent non déchiffré -> C2 HTTPS invisible
- DNS tunnelling pour exfiltration invisible

**4. Fileless / LotL Malware**

- Ne touche jamais le disque -> bypass du sandbox traditionnel
- Ex : attaques PowerShell, persistence WMI

---

## Threat Intelligence Challenge

#### Practice

Fichier à investiguer : `Challenge.bin.sample`

Workflow à appliquer :

1. Analyse du nom / chemin -> heuristiques
2. Hash SHA256 -> lookup VirusTotal + MalwareBazaar
3. Sandbox -> Hybrid Analysis ou Joe Sandbox
4. Extraire : score de détection, labels, IOCs réseau, TTPs ATT&CK

### Questions
#### What is the SHA256 hash of the file?

```powershell
Get-FileHash -Algorithm SHA256 .\Challenge.bin.sample
```

**Answer** : 43b0ac119ff957bb209d86ec206ea1ec3c51dd87bebf7b4a649c7e6c7f3756e7

#### What family labels are assigned to the file on VirusTotal?

![[file_and_hash_threat_intel_t5q2.png]]

**Answer** : akira, filecryptor

#### When was the first time the file was recorded in the wild? (Answer Format: YYYY-MM-DD HH:MM:SS UTC)

Check in VT in details

**Answer** : 2024-10-30 17:17:24 UTC

#### Name the text file dropped during the execution of the malicious file.

Check in the sandobex analysis of trydetectthis

**Answer** : akira_readme.txt

#### What PowerShell command is observed to be executed?

Check in the command execution tab of trydetectethis sandbox analysis

**Answer** : Get-WmiObject Win32_Shadowcopy | Remove-WmiObject

#### What MITRE ATT&CK ID is associated with the actions of the command?

![[file_and_hash_threat_intel_t5q6.png]]

**Answer** : T1490

---

## Résumé - File and Hash Threat Intel

### Workflow analyste L1

```
Nom / Chemin -> Hash -> VirusTotal / MalwareBazaar -> Sandbox
```

#### 1. Filepath & Filename Analysis

Premiers heuristiques disponibles. Ne prouvent pas la malveillance seuls.

**Paths suspects :** `C:\Users\Public`, `C:\Windows\Temp\`, `C:\ProgramData\`

**Techniques de filename :** double extension, system binary impersonation, high-entropy strings, masquerading

#### 2. File Hash

Empreinte immuable du fichier, indépendante du nom.

- SHA256 privilégié
- Hasher l'archive ET le binaire extrait
- Stocker en minuscules avec contexte

#### 3. VirusTotal + MalwareBazaar

|Outil|Priorité|
|---|---|
|VT|Score détection, signatures, relations réseau, comportement|
|MalwareBazaar|Tagging famille malware, rules YARA, attribution campagne|

-> Un fichier à 5/70 sur VT mais tagué `#IcedID` sur MalwareBazaar = **malveillant**

#### 4. Sandbox

|Outil|Usage|
|---|---|
|Hybrid Analysis|Summary rapide + ATT&CK heatmap|
|Joe Sandbox|Analyse profonde (RE / detection eng.)|

**Limites critiques :** évasion de VM, timeout 2-5 min, TLS non déchiffré, fileless/LotL invisible

### Règles d'or

- Rechecke VT après 24h (nouveau malware = faible détection initiale)
- Toujours croiser plusieurs sources (VT + MalwareBazaar + Sandbox)
- Shodan pour pivotter sur les IPs identifiées
- Ne jamais faire confiance à un seul résultat sandbox

