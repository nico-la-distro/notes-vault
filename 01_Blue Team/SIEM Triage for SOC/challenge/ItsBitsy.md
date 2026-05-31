
cf : https://tryhackme.com/room/itsybitsy
## Introduction

In this challenge room, we will take a simple challenge to investigate an alert by IDS regarding a potential C2 communication.

Room Machine

Before moving forward, deploy the machine. When you deploy the machine, it will be assigned an IP **Machine IP**: `MACHINE_IP`. The machine will take up to 3-5 minutes to start. Use the following credentials to log in and access the logs in the Discover tab.

**Username:** `Admin`

**Password:** `elastic123`

---
## Scenario - Investigate a potential C2 communication alert

### Scenario

During normal SOC monitoring, Analyst John observed an alert on an IDS solution indicating a potential C2 communication from a user Browne from the HR department. A suspicious file was accessed containing a malicious pattern THM:{ ________ }. A week-long HTTP connection logs have been pulled to investigate. Due to limited resources, only the connection logs could be pulled out and are ingested into the `connection_logs` index in Kibana.  

Our task in this room will be to examine the network connection logs of this user, find the link and the content of the file, and answer the questions.

### Questions
#### How many events were returned for the month of March 2022?

