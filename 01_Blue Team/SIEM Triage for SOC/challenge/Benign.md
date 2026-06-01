## Introduction

We will investigate host-centric logs in this challenge room to find suspicious process execution. To learn more about Splunk and how to investigate the logs, look at the rooms [splunk101](https://tryhackme.com/room/splunk101) and [splunk201](https://tryhackme.com/room/splunk201).

Room Machine

Before moving forward, deploy the machine. When you deploy the machine, it will be assigned an IP. Access this room via the AttackBox, or via the VPN at `MACHINE_IP`. The machine will take up to 3-5 minutes to start. ll the required logs are ingested in the index `win_eventlogs`.

---
## Scenario: Identify and Investigate an Infected Host

One of the client’s IDS indicated a potentially suspicious process execution indicating one of the hosts from the HR department was compromised. Some tools related to network information gathering / scheduled tasks were executed which confirmed the suspicion. Due to limited resources, we could only pull the process execution logs with Event ID: 4688 and ingested them into Splunk with the index **win_eventlogs** for further investigation.  

### About the Network Information

The network is divided into three logical segments. It will help in the investigation.  

**IT Department**

- James
- Moin
- Katrina

**HR department**

- Haroon
- Chris
- Diana

**Marketing department**

- Bell
- Amelia
- Deepak

### Questions
#### How many logs are ingested from the month of March, 2022?

![[benign_t2q1.png]]

**Answer** : 13959

#### Imposter Alert: There seems to be an imposter account observed in the logs, what is the name of that user?

![[benign_t2q2.png]]

Amel1a trying to usurp Amelia

**Answer** : Amel1a

#### Which user from the HR department was observed to be running scheduled tasks?

```
index=win_eventlogs
| search schtasks.exe
| stats values(UserName)
```

![[benign_t2q3.png]]

**Answer** : Chris.fort

#### Which user from the HR department executed a system process (LOLBIN) to download a payload from a file-sharing host.

![[benign_t2q4.png]]

search for common binaries to download files. i search for bitsadmin, but there is nothing so i search for certutil and find a suspicious log :

```
index=win_eventlogs
| search Certutil
```

![[benign_t2q4_log.png]]

**Answer** : haroon

#### To bypass the security controls, which system process (lolbin) was used to download a payload from the internet?

we can refer to the previous log

**Answer** : certutil.exe

#### What was the date that this binary was executed by the infected host? format (YYYY-MM-DD)

again refer to the previous log

**Answer** : 2022-03-04

#### Which third-party site was accessed to download the malicious payload?

same log

**Answer** : controlc.com

#### What is the name of the file that was saved on the host machine from the C2 server during the post-exploitation phase?

**Answer** : benign.exe

#### The suspicious file downloaded from the C2 server contained malicious content with the pattern THM{..........}; what is that pattern?

go to the location https://controlc.com/e4d11035

**Answer** : THM{KJ&*H^B0}

#### What is the URL that the infected host connected to?

previous url

**Answer** : https://controlc.com/e4d11035

---

## Points clés à retenir

- **certutil.exe** est un binaire Windows natif (gestion de certificats) -> abusé pour télécharger des fichiers depuis internet (LotL/LOLBIN).
- **controlc.com** est un site de partage de texte (comme Pastebin) utilisé ici comme C2.
- Compte imposteur détecté par variation orthographique : `Amel1a` vs `Amelia` -> penser à auditer les noms d'utilisateurs suspects dans les logs.
- Event ID **4688** = création de processus -> source principale pour détecter l'exécution de binaires suspects.

