## Introduction

Volatility -> outil de forensic mémoire open-source (Python), maintenu par Volatility Foundation. Utilisé par analystes malware / SOC (blue team). Fonctionne via plugins Python modulaires (plug-and-play). Compatible Windows, Linux, Mac OS.

cd : C:\~\Documents\thm\volatility

---

## Volatility Overview

Framework d'extraction d'artefacts depuis la RAM, indépendant du système analysé. Workflow de base : identifier le type d'image -> lancer les plugins appropriés.

Deux versions :

- Volatility2 -> Python 2 (syntaxe différente, souvent citée dans les anciens articles)
- Volatility3 -> Python 3 **(version utilisée dans cette room)**

Repo officiel : [volatility3](https://github.com/volatilityfoundation/volatility3)

---

## Installing Volatility

**Installation depuis les sources**

Dépendances obligatoires :

- Python >= 3.5.3
- [pefile >= 2017.8.1](https://pypi.org/project/pefile/)

Dépendances optionnelles (recommandées) :

- [yara-python >= 3.8.0](https://github.com/VirusTotal/yara-python)
- [capstone >= 3.0.0](https://www.capstone-engine.org/download.html)

bash

```bash
git clone https://github.com/volatilityfoundation/volatility3.git
python3 vol.py -h
```

Alternative Windows : exécutable prépackagé (.whl, sans dépendances) -> [releases page](https://github.com/volatilityfoundation/volatility3/releases/tag/v1.0.1)

Pour analyser des dumps Linux/Mac -> télécharger les [symbol tables](https://github.com/volatilityfoundation/volatility3#symbol-tables)

---

## Memory Extraction

**Outils d'extraction (bare-metal)**

FTK Imager, Redline, DumpIt.exe, win32dd.exe / win64dd.exe, Memoryze, FastDump

Output généralement en `.raw` (sauf Redline -> format propriétaire agent/session). Extraction bare-metal -> peut prendre beaucoup de temps.

**Fichiers mémoire selon hyperviseur (VM)**

|Hyperviseur|Extension|
|---|---|
|VMware|`.vmem`|
|Hyper-V|`.bin`|
|Parallels|`.mem`|
|VirtualBox|`.sav` (partiel)|

---

## Plugins Overview

**Volatility2 vs Volatility3**

| |Volatility2|Volatility3|
|---|---|---|
|Profils OS|Obligatoire (manuel, risque faux positifs)|Supprimé -> détection automatique|
|Syntaxe plugin|`pluginname`|`os.pluginname`|

**Préfixes OS obligatoires en V3**

- `windows.`
- `linux.`
- `mac.`

Exemple : `windows.info` / `linux.info`

Lister les plugins disponibles :

bash

```bash
python3 vol.py -h
```

---

## Identifying Image Info and Profiles

`imageinfo` -> Volatility2 uniquement, déprécié en V3. Résultats pas toujours fiables -> tester plusieurs profils de la liste retournée.

Pour obtenir les infos du système depuis un dump en V3 :

```bash
python3 vol.py -f <file> windows.info
python3 vol.py -f <file> linux.info
python3 vol.py -f <file> mac.info
```

---

## Listing Processes and Connections

|Plugin|Méthode|Particularité|
|---|---|---|
|`pslist`|Liste doublement chaînée (= Task Manager)|Ne voit pas les processus délinkés (rootkits)|
|`psscan`|Scan via structures `_EPROCESS`|Détecte les processus cachés, risque de faux positifs|
|`pstree`|Même méthode que `pslist`|Affiche l'arborescence parent/enfant|
|`netstat`|Scan des structures réseau|Instable sur vieux builds Windows|
|`dlllist`|Liste les DLLs associées aux processus|Utile pour filtrer sur une DLL suspecte|

bash

```bash
python3 vol.py -f <file> windows.pslist
python3 vol.py -f <file> windows.psscan
python3 vol.py -f <file> windows.pstree
python3 vol.py -f <file> windows.netstat
python3 vol.py -f <file> windows.dlllist
```

Si `netstat` est trop instable -> utiliser [bulk_extractor](https://tools.kali.org/forensics/bulk-extractor) pour extraire un PCAP depuis le dump.

---

## Advanced Memory Forensics

### Hooking

5 méthodes de hooking : SSDT, IRP, IAT, EAT, Inline Hooks -> Focus sur **SSDT** (le plus courant, plugin dispo en natif).

**SSDT** (System Service Descriptor Table) -> table du kernel Windows pour les fonctions système. Un rootkit peut modifier les pointeurs de cette table pour rediriger vers son propre code.

bash

```bash
python3 vol.py -f <file> windows.ssdt
```

Output volumineux -> comparer avec une baseline ou utiliser après avoir déjà une piste d'investigation.

### Drivers

|Plugin|Méthode|Limite|
|---|---|---|
|`modules`|Liste les modules kernel chargés|Manque les drivers cachés/inactifs|
|`driverscan`|Scan complet des drivers présents|Souvent vide, mais complète `modules`|

bash

```bash
python3 vol.py -f <file> windows.modules
python3 vol.py -f <file> windows.driverscan
```

Ordre recommandé : `modules` d'abord -> puis `driverscan` si rien trouvé.

### Autres plugins utiles

`modscan`, `driverirp`, `callbacks`, `idt`, `apihooks`, `moddump`, `handles`

Certains sont Volatility2 uniquement ou des plugins tiers -> peuvent nécessiter une installation séparée.

---

## Practical Investigations

### Case 001 - BOB! THIS ISN'T A HORSE!

- Contexte : banking trojan se faisant passer pour un document Adobe
- IP suspecte connue : `41.168.5.140`
- Fichier : `/Scenarios/Investigations/Investigation-1.vmem`

### Case 002 - That Kind of Hurt my Feelings

- Contexte : ransomware international, analyse post-incident
- Clé de déchiffrement déjà récupérée, systèmes restaurés
- Objectif : identifier les acteurs et reconstituer l'attaque
- Fichier : `/Scenarios/Investigations/Investigation-2.raw`

### Questions
#### What is the build version of the host machine in Case 001?

```bash
python3 /opt/volatility3/vol.py -f /Scenarios/Investigations/Investigation-1.vmem windows.info
```

![[volatility_t10q1.png]]

**Answer** : 2600.xpsp.080413-2111

#### At what time was the memory file acquired in Case 001?

cf previous screenshot

**Answer** : 2012-07-22 02:45:08

#### What process can be considered suspicious in Case 001?  
Note: Certain special characters may not be visible on the provided VM. When doing a copy-and-paste, it will still copy all characters.

```bash
python3 /opt/volatility3/vol.py -f /Scenarios/Investigations/Investigation-1.vmem windows.psscan
```

![[volatility_t10q3.png]]

**Answer** : reader_sl.exe

#### What is the parent process of the suspicious process in Case 001?

```bash
 python3 /opt/volatility3/vol.py -f /Scenarios/Investigations/Investigation-1.vmem windows.pstree
```

![[volatility_t10q4.png]]

PID is the PPID of reader_sl.exe

**Answer** : explorer.exe

#### What is the PID of the suspicious process in Case 001?

cf screenshot question 3

**Answer** : 1640

#### What is the parent process PID in Case 001?

cf screenshot question 3

**Answer** : 1484

#### What user-agent was employed by the adversary in Case 001?

```bash
strings /Scenarios/Investigations/Investigation-1.vmem | grep -i "user-agent"
```

![[volatility_t10q7.png]]

**Answer** : Mozilla/5.0 (Windows; U; MSIE 7.0; Windows NT 6.0; en-US)

#### Was Chase Bank one of the suspicious bank domains found in Case 001? (Y/N)

```bash
strings /Scenarios/Investigations/Investigation-1.vmem | grep -i "chase"
```

**Answer** : Y

#### What suspicious process is running at PID 740 in Case 002?

```bash
 python3 /opt/volatility3/vol.py -f /Scenarios/Investigations/Investigation-2.raw windows.psscan
```

![[volatility_t10q9.png]]

**Answer** : @WanaDecryptor@

#### What is the full path of the suspicious binary in PID 740 in Case 002?

```bash
 python3 /opt/volatility3/vol.py -f /Scenarios/Investigations/Investigation-2.raw windows.dlllist --pid 740
```

![[volatility_t10q10.png]]

**Answer** : C:\Intel\ivecuqmanpnirkt615\@WanaDecryptor@.exe

#### What is the parent process of PID 740 in Case 002?

```bash
python3 /opt/volatility3/vol.py -f /Scenarios/Investigations/Investigation-2.raw windows.pstree --pid 740
```

![[volatility_t10q11.png]]

**Answer** : tasksche.exe
#### What is the suspicious parent process PID connected to the decryptor in Case 002?

cf previous screenshot

**Answer** : 1940

#### From our current information, what malware is present on the system in Case 002?

search @WanaDecryptor@.exe on inernet

**Answer** : wannacry

#### What DLL is loaded by the decryptor used for socket creation in Case 002?

```bash
python3 /opt/volatility3/vol.py -f /Scenarios/Investigations/Investigation-2.raw windows.dlllist
```

![[volatility_t10q13.png]]

**Answer** : Ws2_32.dll

#### What mutex can be found that is a known indicator of the malware in question in Case 002?

```bash
python3 /opt/volatility3/vol.py -f /Scenarios/Investigations/Investigation-2.raw windows.handles | grep 1940
```

![[volatility_t10q14.png]]

**Answer** : MsWinZonesCacheCounterMutexA

#### What plugin could be used to identify all files loaded from the malware working directory in Case 002?

**Answer** : windows.filescan

---

## Conclusion

Livre recommandé pour approfondir : **The Art of Memory Forensics**

Ressources :

- [Volatility Wiki](https://github.com/volatilityfoundation/volatility/wiki)
- [Volatility Documentation Project](https://github.com/volatilityfoundation/volatility/wiki/Volatility-Documentation-Projec)
- [SANS Memory Forensics Poster](https://digital-forensics.sans.org/media/Poster-2015-Memory-Forensics.pdf)
- [Finding Advanced Malware Using Volatility](https://eforensicsmag.com/finding-advanced-malware-using-volatility/)

---

## Résumé - Room Volatility

### C'est quoi Volatility

Outil Python de forensic mémoire (RAM). Analyse des dumps sans toucher au système d'origine. Version à utiliser : **Volatility3** (Python 3).

### Obtenir un dump mémoire

- Machine physique -> outil dédié (FTK Imager, DumpIt, etc.) -> fichier `.raw`
- Machine virtuelle -> fichier déjà sur le disque hôte (`.vmem`, `.bin`, `.mem`)

### Workflow de base

1. Identifier l'OS du dump -> `windows.info`
2. Lister les processus -> `pslist` / `psscan` / `pstree`
3. Vérifier les connexions réseau -> `netstat`
4. Analyser les DLLs -> `dlllist`
5. Si malware avancé -> `ssdt`, `modules`, `driverscan`

### Identifier un processus suspect

- Connaître les process Windows légitimes par coeur
- Vérifier les PPID (parent inattendu = suspect)
- Croiser avec le contexte de l'investigation
- Chercher des strings, mutex, user-agents dans le dump

### Différence clé V2 vs V3

V2 -> profil OS manuel obligatoire V3 -> détection automatique, préfixe OS dans le nom du plugin (`windows.`, `linux.`, `mac.`)

### Plugins essentiels à retenir

|Objectif|Plugin|
|---|---|
|Info système|`windows.info`|
|Processus|`pslist`, `psscan`, `pstree`|
|Réseau|`netstat`|
|DLLs|`dlllist`|
|Fichiers ouverts|`filescan`|
|Hooks kernel|`ssdt`|
|Drivers|`modules`, `driverscan`|
|Handles/Mutex|`handles`|

