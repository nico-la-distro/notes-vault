
cf : https://tryhackme.com/room/itsybitsy
## Introduction

In this challenge room, we will take a simple challenge to investigate an alert by IDS regarding a potential C2 communication.

---
## Scenario - Investigate a potential C2 communication alert

### Scenario

During normal SOC monitoring, Analyst John observed an alert on an IDS solution indicating a potential C2 communication from a user Browne from the HR department. A suspicious file was accessed containing a malicious pattern THM:{ ________ }. A week-long HTTP connection logs have been pulled to investigate. Due to limited resources, only the connection logs could be pulled out and are ingested into the `connection_logs` index in Kibana.  

Our task in this room will be to examine the network connection logs of this user, find the link and the content of the file, and answer the questions.

### Questions
#### How many events were returned for the month of March 2022?

![[itsbitsy_t2q1.png]]

**Answer** : 1482

#### What is the IP associated with the suspected user in the logs?

![[itsbitsy_t2q2.png]]

**Answer** : 192.166.65.54

#### The user’s machine used a legit windows binary to download a file from the C2 server. What is the name of the binary?

![[itsbitsy_t2q3.png]]

**Answer** : bitsadmin

#### The infected machine connected with a famous filesharing site in this period, which also acts as a C2 server used by the malware authors to communicate. What is the name of the filesharing site?

![[itsbitsy_t2q4.png]]

**Answer** : pastebin.com

#### What is the full URL of the C2 to which the infected host is connected?

![[itsbitsy_t2q5.png]]

**Answer** : pastebin[.]com/yTg0Ah6a

#### A file was accessed on the filesharing site. What is the name of the file accessed?

open the url in a browser

![[itsbitsy_t2q6.png]]

**Answer** : secret.txt

#### The file contains a secret code with the format THM{_____}.

we can see the content in the previous screenshot

**Answer** : 1. THM{SECRET__CODE}

---

## Points clés à retenir

- **bitsadmin** est un binaire Windows natif (`BITS` - Background Intelligent Transfer Service) -> souvent abusé pour télécharger des fichiers depuis un C2 (technique de Living off the Land / LotL).
- **Pastebin** est régulièrement utilisé comme C2 léger : les attaquants y déposent des payloads ou instructions en clair.
- Workflow : filtrer sur l'IP suspecte dans Kibana -> identifier les domaines contactés -> repérer `pastebin.com` -> ouvrir l'URL dans un navigateur pour lire le fichier.

