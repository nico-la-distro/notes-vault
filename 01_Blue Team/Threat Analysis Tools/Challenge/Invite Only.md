cf :https://tryhackme.com/room/invite-only

You are an SOC analyst on the SOC team at Managed Server Provider TrySecureMe. Today, you are supporting an L3 analyst in investigating flagged IPs, hashes, URLs, or domains as part of IR activities. One of the L1 analysts flagged two suspicious findings early in the morning and escalated them. Your task is to analyse these findings further and distil the information into usable threat intelligence.

Flagged IP: **101[.]99[.]76[.]120**  
Flagged SHA256 hash: **5d0509f68a9b7c415a726be75a078180e3f02e59866f193b0a99eee8e39c874f**

We recently purchased a new threat intelligence search application called TryDetectThis2.0. You can use this application to gather information on the indicators above.

---
### Questions
#### What is the name of the file identified with the flagged SHA256 hash?

go check on VT

![[invite_only_q1.png]]

**Answer** : syshelpers.exe

#### What is the file type associated with the flagged SHA256 hash?

![[invite_only_q2.png]]

**Answer** : Win32 EXE

#### What are the execution parents of the flagged hash? List the names chronologically, using a comma as a separator. Note down the hashes for later use.

VT in relations tab :

![[invite_only_q3.png]]

```hashes
047c5eec0445746862710d20e50a5dd04510b7e625fa5c1f5d48ce078001c0de
fa102d4e3cfbe85f5189da70a52c1d266925f3efd122091cdc8fe0fc39033942
```

**Answer** : 361GJX7J,installer.exe

#### What is the name of the file being dropped? Note down the hash value for later use.

VT in relations tab :

![[invite_only_q4.png]]

```sha256
dd02c105809e4ca41a5489e585ba025eddb89a91703b73a566c9903e6406a08c
```

**Answer** : AClient.exe

#### 

check in the trydetectme app

![[invite_only_q5.png]]

**Answer** : searchhost.exe,syshelpers.exe,nat.vbs,runsys.vbs

#### Analyse the files related to the flagged IP. What is the malware family that links these files?

I checked in the community tab

![[invite_only_q6.png]]

**Answer** : AsyncRAT

#### What is the title of the original report where these flagged indicators are mentioned? Use Google to find the report.

the answer is in the 'tittle' part of the previous screeshot

**Answer** : From Trust to Threat: Hijacked Discord Invites Used for Multi-Stage Malware Delivery

#### Which tool did the attackers use to steal cookies from the Google Chrome browser?

again in the previous screenshot there is a link to a post on this attaque. the tool used is in it

cf : https://research.checkpoint.com/2025/from-trust-to-threat-hijacked-discord-invites-used-for-multi-stage-malware-delivery/

![[invite_only_q7.png]]

**Answer** : ChromeKatz

#### Which phishing technique did the attackers use? Use the report to answer the question.

The answer is in the same post

![[invite_only_q8.png]]

**Answer** : ClickFix

#### What is the name of the platform that was used to redirect a user to malicious servers?

Same post

![[invite_only_q9.png]]

**Answer** : Discord

---

## Résumé de l'analyse

On reçoit un hash inconnu. On le soumet sur VirusTotal : c'est **syshelpers.exe**, un Win32 EXE.

On consulte l'onglet _Relations_ pour savoir ce qui l'a lancé. Deux parents remontent, **361GJX7J** puis **installer.exe**. syshelpers.exe n'est pas le point d'entrée, il a été déposé par un loader en amont. Toujours dans _Relations_, on voit ce qu'il droppe : **AClient.exe**. On est face à une chaîne multi-étages.

En élargissant aux fichiers associés, on trouve aussi **searchhost.exe, nat.vbs, runsys.vbs**. Les .vbs signalent un mécanisme de persistance via tâches planifiées.

On passe au deuxième IOC, l'IP **101.99.76.120**. L'onglet _Community_ identifie la famille **AsyncRAT**, ce qui confirme que c'est un serveur C2.

On cherche le rapport de threat intel mentionné dans les commentaires : c'est le rapport Check Point **"From Trust to Threat"**. Il complète le tableau : **ClickFix** comme technique de phishing, **Discord** comme vecteur initial, **ChromeKatz** pour le vol de cookies.

Deux IOCs bruts ont permis de remonter toute la kill chain et de rattacher l'activité à une campagne documentée.