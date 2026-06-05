## Introduction

Room : [Boogeyman 2](https://tryhackme.com/room/boogeyman2) -> Analyse des nouveaux TTPs du groupe Boogeyman.

### Artefacts

Chemin : `/home/ubuntu/Desktop/Artefacts`

- Email de phishing
- Memory dump du poste victime

### Tools

|Outil|Usage|
|---|---|
|Volatility|`vol -f memorydump.raw <plugin>`|
|Olevba|`olevba document.doc`|

```bash
# Lister tous les plugins Volatility
vol -f memorydump.raw -h
```

Références :

- [Volatility 3 Plugins](https://volatility3.readthedocs.io/en/latest/volatility3.plugins.html)
- [Volatility GitHub](https://github.com/volatilityfoundation/volatility3)
- [Oletools GitHub](https://github.com/decalage2/oletools)

---

## Spear Phishing Human Resources

**Victime :** Maxine, HR Specialist @ Quick Logistics LLC

**Vecteur :** Faux CV envoyé via une offre d'emploi ouverte -> compromission du poste.

**Déclencheur de l'investigation :** Commandes suspectes détectées sur le poste de Maxine.

**Objectif :** Analyser et évaluer l'impact de la compromission.

![[boogeyman2_email.png]]

### Questions
#### What email was used to send the phishing email?

open up `Resume - Application for Junior IT Analyst Role.eml`

![[boogeyman2_t2q1.png]]

**Answer** : westaylor23@outlook.com

#### What is the email of the victim employee?

refer to the previous screenshot

**Answer** : maxine.beck@quicklogisticsorg.onmicrosoft.com

#### What is the name of the attached malicious document?

again refer to the screenshot question 1

**Answer** : Resume_WesleyTaylor.doc

#### What is the MD5 hash of the malicious attachment?

```bash
md5sum Resume_WesleyTaylor.doc 
52c4384a0b9e248b95804352ebec6c5b  Resume_WesleyTaylor.doc
```

**Answer** : 52c4384a0b9e248b95804352ebec6c5b

#### What URL is used to download the stage 2 payload based on the document's macro?

```bash
olevba Resume_WesleyTaylor.doc
```

![[boogeyman2_t2q5.png]]

**Answer** : https://files.boogeymanisback.lol/aa2a9c53cbb80416d3b47d85538d9971/update.png

#### What is the name of the process that executed the newly downloaded stage 2 payload?

in the same ouput of the previous command we can see the name of that process

![[boogeyman2_t2q6.png]]

**Answer** : wscript.exe

#### What is the full file path of the malicious stage 2 payload?

we can see its path in the previous screenshot

**Answer** : C:\ProgramData\update.js

#### What is the PID of the process that executed the stage 2 payload?

```bash
vol -f WKSTN-2961.raw windows.pstree
```

![[boogeyman2_t2q8.png]]

**Answer** : 4260

#### What is the parent PID of the process that executed the stage 2 payload?

refer to the previous screenshot

**Answer** : 1124

#### What URL is used to download the malicious binary executed by the stage 2 payload?

refer to the question 5

**Answer** : https://files.boogeymanisback.lol/aa2a9c53cbb80416d3b47d85538d9971/update.png

#### What is the PID of the malicious process used to establish the C2 connection?

```bash
vol -f WKSTN-2961.raw windows.pstree
```

refer to the url and pstree we can identified the process and its pid

![[boogeyman2_t2q11.png]]

**Answer** : 6216

#### What is the full file path of the malicious process used to establish the C2 connection?

![[boogeyman2_t2q12_2.png]]

```bash
vol -f WKSTN-2961.raw windows.cmdline --pid 6216
```

![[boogeyman2_t2q12.png]]

**Answer** : C:\Windows\Tasks\updater.exe

#### What is the IP address and port of the C2 connection initiated by the malicious binary? (Format: IP address:port)

![[boogeyman2_t2q13_2.png]]

```bash
vol -f WKSTN-2961.raw windows.netscan | grep 6216
```

![[boogeyman2_t2q13.png]]

**Answer** : 128.199.95.189:8080

#### What is the full file path of the malicious email attachment based on the memory dump?

```bash
vol -f WKSTN-2961.raw windows.cmdline | grep '.doc'
```

![[boogeyman2_t2q14.png]]

**Answer** : C:\Users\maxine.beck\AppData\Local\Microsoft\Windows\INetCache\Content.Outlook\WQHGZCFI\Resume_WesleyTaylor (002).doc

#### The attacker implanted a scheduled task right after establishing the c2 callback. What is the full command used by the attacker to maintain persistent access?

```bash
strings WKSTN-2961.raw | grep -i 'schtask'
```

![[boogeyman2_t2q15.png]]

**Answer** : schtasks /Create /F /SC DAILY /ST 09:00 /TN Updater /TR 'C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -NonI -W hidden -c \"IEX ([Text.Encoding]::UNICODE.GetString([Convert]::FromBase64String((gp HKCU:\Software\Microsoft\Windows\CurrentVersion debug).debug)))\"'

---

## Résumé

### Chaîne d'attaque - Boogeyman 2

**Vecteur initial :** Spear phishing RH -> faux CV (`Resume_WesleyTaylor.doc`) envoyé à Maxine.

**Étapes :**

1. Macro VBA dans le `.doc` -> télécharge `update.png` depuis `files.boogeymanisback.lol`
2. `wscript.exe` exécute `C:\ProgramData\update.js` (stage 2)
3. `update.js` télécharge et lance `updater.exe` -> `C:\Windows\Tasks\updater.exe`
4. `updater.exe` (PID 6216) établit la connexion C2 : `128.199.95.189:8080`
5. Persistance via scheduled task :

powershell

```powershell
schtasks /Create /F /SC DAILY /ST 09:00 /TN Updater /TR 'C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -NonI -W hidden -c "IEX ([Text.Encoding]::UNICODE.GetString([Convert]::FromBase64String((gp HKCU:\Software\Microsoft\Windows\CurrentVersion debug).debug)))"'
```

**Commandes clés utilisées :**

|Objectif|Commande|
|---|---|
|Arbre des process|`vol -f WKSTN-2961.raw windows.pstree`|
|Chemin d'un process|`vol -f WKSTN-2961.raw windows.cmdline --pid <PID>`|
|Connexions réseau|`vol -f WKSTN-2961.raw windows.netscan \| grep <PID>`|
|Macro VBA|`olevba Resume_WesleyTaylor.doc`|
|Persistance|`strings WKSTN-2961.raw \| grep -i 'schtask'`|
