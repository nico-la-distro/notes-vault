## Introduction

Analyse des nouveaux TTPs du groupe **Boogeyman** suite à leur retour chez Quick Logistics LLC.

### Investigation Platform

Stack **ELK** -> accès via Kibana.

---

## The Chaos Inside

### Lurking in the Dark

- Compromission d'un employé -> accès email obtenu
- Cible secondaire : CEO **Evan Hutchinson**
- Evan a ouvert la pièce jointe -> rien de visible -> a signalé le phishing

![[boogeyman3_email.png]]

### Initial Investigation

- Pièce jointe trouvée dans le dossier **Downloads** du CEO
- Payload de type **ISO** contenant un fichier suspect
- Période de l'incident : **29 - 30 août 2023**

![[boogeyman3_iso.png]]

**Objectif** : Analyser et évaluer l'impact de la compromission.

### Questions
#### What is the PID of the process that executed the initial stage 1 payload?

I first started to search for the CEO computer.

```
evan OR hutchinson OR CEO
```

i find his host.name that is : WKSTN-0051.quicklogistics.org

so i filtered the log with his hostname, eventid 1 (process creation) and process parent with explorer.exe because the CEO downloaded it

```
host.name : WKSTN-0051.quicklogistics.org and event.code :1 and process.parent.name : explorer.exe
```

18 logs was found. i add md5 hash to quickly see repetitive pattern and found the hash 665d512bb2727713783b73f1b7feb808. This is the hash of mshta.exe a native binary that execute html application.

in the process fields i found out that this binary execute our malicious file

![[boogeyman3_t2q1.png]]

so i just checked for the pid to answer the question

**Answer** : 6392

#### The stage 1 payload attempted to implant a file to another location. What is the full command-line value of this execution?

with the pid of the first step of the payload, we can now search with 6392 as a parent pid to find what's next

```
host.name: WKSTN-0051.quicklogistics.org AND event.code: 1 AND process.parent.pid: 6392
```

then we can find the first command executed

![[boogeyman3_t2q2.png]]

**Answer** : "C:\Windows\System32\xcopy.exe" /s /i /e /h D:\review.dat C:\Users\EVAN~1.HUT\AppData\Local\Temp\review.dat

#### The implanted file was eventually used and executed by the stage 1 payload. What is the full command-line value of this execution?

so this is the very next command from the previous question

![[boogeyman3_t2q3.png]]

**Answer** : "C:\Windows\System32\rundll32.exe" D:\review.dat,DllRegisterServer

#### The stage 1 payload established a persistence mechanism. What is the name of the scheduled task created by the malicious script?

next command from the previous

![[boogeyman3_t2q4.png]]

**Answer** : Review

#### The execution of the implanted file inside the machine has initiated a potential C2 connection. What is the IP and port used by this connection? (format: IP:port)

i removed the filter and checked the destination ip filed

```
host.name: WKSTN-0051.quicklogistics.org
```

![[boogeyman3_t2q5.png]]

an ip is obviously suspect. i filter it and checked for the port that is 80

**Answer** : 165.232.170.151:80

#### The attacker has discovered that the current access is a local administrator. What is the name of the process used by the attacker to execute a UAC bypass?

fodhelper.exe is a well-known when it's about to bypass UAC

_**Fodhelper Exploit**: This is a widely used automated method (available in Metasploit as `bypassuac_fodhelper`) that abuses the Windows Features-On-Demand helper. It works by creating a registry key at `HKCU\Software\Classes\ms-settings\Shell\Open\command` to redirect `fodhelper.exe`'s execution to a payload, which then runs with elevated privileges._

**Answer** : fodhelper.exe

#### Having a high privilege machine access, the attacker attempted to dump the credentials inside the machine. What is the GitHub link used by the attacker to download a tool for credential dumping?

first i started filter

```
host.name: WKSTN-0051.quicklogistics.org and (user.name : Administrator OR SYSTEM) and event.code : 11
```

and found a very suspicious file

![[boogeyman3_t2q7.png]]

with this filter i found the url where mimikatz was downloaded

```
host.name: WKSTN-0051.quicklogistics.org and (user.name : Administrator OR SYSTEM) and message : mimikatz
```

![[boogeyman3_t2q8.png]]

**Answer** : https://github.com/gentilkiwi/mimikatz/releases/download/2.2.0-20220919/mimikatz_trunk.zip

#### After successfully dumping the credentials inside the machine, the attacker used the credentials to gain access to another machine. What is the username and hash of the new credential pair? (format: username:hash)

2 logs after the mimikatz download with the same filter

![[boogeyman3_t2q9.png]]

**Answer** : itadmin:F84769D250EB95EB2D7D8B4A1C5613F2

#### Using the new credentials, the attacker attempted to enumerate accessible file shares. What is the name of the file accessed by the attacker from a remote share?

filter with this pid 6160 that is the powershell used by the attacker

![[boogeyman3_t2q10.png]]

**Answer** : IT_automation.ps1

#### After getting the contents of the remote file, the attacker used the new credentials to move laterally. What is the new set of credentials discovered by the attacker? (format: username:password)

right after the attacker connect to a new user

![[boogeyman3_t2q11.png]]

**Answer** : QUICKLOGISTICS\allan.smith:Tr!ckyP@ssw0rd987

#### What is the hostname of the attacker's lab machine for its lateral movement attempt?

we can see this in the same previous log

**Answer** : WKSTN-1327

#### Using the malicious command executed by the attacker from the first machine to move laterally, what is the parent process name of the malicious command executed on the second compromised machine?

with this filters that's focus on the second infected machine and user and add the filter to find the process

![[boogeyman3_t2q12.png]]

then check for the first log

![[boogeyman3_t2q12_2.png]]

**Answer** : wsmprovhost.exe

#### The attacker then dumped the hashes in this second machine. What is the username and hash of the newly dumped credentials?

with this filter we are able to find the right log

```
host.hostname : WKSTN-1327 and user.name allan.smith and message : mimikatz
```

![[boogeyman3_t2q14.png]]

**Answer** : administrator:00f80f2538dcb54e7adc715c0e7091ec

#### After gaining access to the domain controller, the attacker attempted to dump the hashes via a DCSync attack. Aside from the administrator account, what account did the attacker dump?

first add this filter, then take the older log

```
host.hostname :"DC01" and message : dcsync
```

![[boogeyman3_t2q15.png]]

**Answer** : backupda

#### After dumping the hashes, the attacker attempted to download another remote file to execute ransomware. What is the link used by the attacker to download the ransomware binary?

i took the pid of the previous question to filter then retrieved the url

```
host.hostname :"DC01" and process.parent.pid : 4008
```

![[boogeyman3_t2q16.png]]

**Answer** : http://ff.sillytechninja.io/ransomboogey.exe

---

## Boogeyman 3 - Key Points Summary

**Attack chain** : Phishing email -> ISO payload -> `mshta.exe` executes `.hta` file -> DLL sideloaded via `rundll32.exe` -> Persistence via Scheduled Task -> C2 -> Lateral movement -> DCSync -> Ransomware

**Initial Access** CEO opened phishing attachment -> `mshta.exe` (LOLBin) executed the `.hta` payload (PID `6392`). ISO contained `review.dat` (disguised DLL), copied to `Temp` via `xcopy`.

**Persistence** Scheduled Task named `Review` -> runs `rundll32.exe` + `review.dat,DllRegisterServer` daily at 06:00.

**C2** `165.232.170.151:80`

**Privilege Escalation** UAC bypass via `fodhelper.exe`.

**Credential Dumping** Mimikatz downloaded from GitHub -> dumped `itadmin:F84769D250EB95EB2D7D8B4A1C5613F2`.

**Lateral Movement** Credentials `QUICKLOGISTICS\allan.smith:Tr!ckyP@ssw0rd987` found in remote share `IT_automation.ps1`. Moved to `WKSTN-1327` -> parent process `wsmprovhost.exe` (WinRM). Dumped `administrator:00f80f2538dcb54e7adc715c0e7091ec`.

**Domain Compromise** DCSync on `DC01` -> dumped account `backupda`. Ransomware downloaded from `http://ff.sillytechninja.io/ransomboogey.exe`.