
[https://tryhackme.com/room/wazuhct](https://tryhackme.com/room/wazuhct)
## Task 1 Introduction

Wazuh : plateforme de sécurité unifiée open-source, née EDR, étend vers event management, vulnerability assessment, cloud security monitoring.

Fonctionnalités :

- Audit des vulnérabilités des devices
- Monitoring d'activité suspecte
- Dashboards/graphs
- Compliance reporting (PCI DSS, HIPAA, NIST)

Fondé en 2015 -> [wazuh.com](https://wazuh.com/)

Modèle manager/agent :

- **Manager** : serveur unique, stocke/traite les données
- **Agent** : hosts qui envoient les logs au manager

## Task 2 Deploy Wazuh Server

Connexion au serveur Wazuh via `https://MACHINE_IP` (attendre 5 min avant accès, sinon "Kibana Server is not ready yet").

```
Username: wazuh (lowercase)
Password: eYa0M1-hG0e7rjGi-lRB2qGYVoonsG1K
```

Sélectionner "Global Tenant" après login. Les agents apparaîtront _disconnected_ (normal).

## Task 3 Wazuh Agents

Agent = device qui enregistre events/process (auth, gestion utilisateurs) et les envoie au collector.

Prérequis déploiement agent :

- OS
- Adresse du serveur Wazuh (DNS ou IP)
- Groupe d'appartenance de l'agent

Wizard : **Wazuh -> Agents -> Deploy New Agent** (étape 4 = commande à copier/coller pour installer/configurer l'agent).

## Task 4 Wazuh Vulnerability Assessment & Security Events

Module Vulnerability Assessment : scan périodique des apps installées + versions -> comparaison base CVE. Exemple : Vim vulnérable à **CVE-2019-12735**.

Scan complet à l'install, puis intervalle configurable (par défaut 5 min) dans `/var/ossec/etc/ossec.conf`.

Ruleset par défaut sensible : ex. 769 events "maintenance quotidienne" détectés comme security event.

Analyse events : tri par timestamp, tactics, description.

## Task 5 Wazuh Policy Auditing

Audit config agent à l'install -> score selon frameworks : NIST, MITRE, GDPR, SCA.

Module : **Wazuh -> Modules -> Policy Management**

Frameworks détaillés dans [Pentesting Fundamentals](https://tryhackme.com/room/pentestingfundamentals).

## Task 6 Monitoring Logons with Wazuh

Rule ID **5710** : tentative de connexion SSH échouée.

|Field|Value|Description|
|---|---|---|
|agent.ip|10.10.73.118|IP de l'agent|
|agent.name|ip-10-10-73-118|hostname de l'agent|
|rule.description|sshd: Attempt to login using a non-existent user|description brève|
|rule.mitre.technique|Brute-Force|technique MITRE|
|rule.mitre.id|T1110|ID MITRE|
|rule.id|5710|ID règle Wazuh|
|location|/var/log/auth.log|fichier source sur l'agent|

Fichier d'alertes serveur : `/var/ossec/logs/alerts/alerts.log` (grep/nano).

```shell-session
sudo less /var/ossec/logs/alerts/alerts.log
** Alert 1634284538.566764: - pam,syslog,authentication_success,...
2021 Oct 15 07:55:38 ip-10-10-218-190->/var/log/auth.log
Rule: 5501 (level 3) -> 'PAM: Login session opened.'
User: root
```

Module rules : **Wazuh -> Management -> Rules**

## Task 7 Collecting Windows Logs with Wazuh

Events Windows (auth, réseau, fichiers, apps) stockés via **Sysmon**.

Config Sysmon en XML, ex : monitoring lancement de `powershell.exe`.

Lancement : `Sysmon64.exe -accepteula -i detect_powershell.xml`

Vérification via Event Viewer (module "Sysmon").

Config agent Wazuh Windows : `C:\Program Files (x86)\ossec-agent\ossec.conf`

```shell-session
<localfile>
<location>Microsoft-Windows-Sysmon/Operational</location>
<log_format>eventchannel</log_format>
</localfile>
```

Redémarrer l'agent Wazuh après modification.

Règle locale serveur Wazuh : `/var/ossec/etc/rules/local_rules.xml`

```shell-session
<group name="sysmon,">
 <rule id="255000" level="12">
 <if_group>sysmon_event1</if_group>
 <field name="sysmon.image">\\powershell.exe||\\.ps1||\\.ps2</field>
 <description>Sysmon - Event 1: Bad exe: $(sysmon.image)</description>
 <group>sysmon_event1,powershell_execution,</group>
 </rule>
</group>
```

Redémarrer le Wazuh Management server pour appliquer.

## Task 8 Collecting Linux Logs with Wazuh

Log collector service de l'agent -> envoie logs choisis au manager.

Règles préinstallées : `/var/ossec/ruleset/rules` (~900 rulesets : Docker, FTP, WordPress, SQL Server, MongoDB, Firewalld...).

Exemple : ruleset Apache2 `0250-apache_rules.xml`.

Config agent : `/var/ossec/etc/ossec.conf`

```shell-session
<!-- Apache2 Log Analysis -->
<localfile>
  <location>/var/log/example.log</location>
  <log_format>syslog</log_format>
</localfile>
```

Redémarrer l'agent Linux ensuite.

## Task 9 Auditing Commands on Linux with Wazuh

`auditd` (Debian/Ubuntu, CentOS) : monitore actions système -> log file -> lu par Wazuh log collector.

Install :

```shell-session
sudo apt-get install auditd audispd-plugins
sudo systemctl enable auditd.service
sudo systemctl start auditd.service
```

Règles : `/etc/audit/rules.d/audit.rules` (édition manuelle, ex : monitorer commandes exécutées en root, ou tcpdump/netcat/cat /etc/passwd).

```shell-session
## First rule - delete all
-D

## Increase the buffers to survive stress events.
-b 8192

## This determine how long to wait in burst of events
--backlog_wait_time 0

## Set failure mode to syslog
-f 1

-a exit,always -F arch=b64 -F euid=0 -S execve -k audit-wazuh-c
```

Recharger règles : `sudo auditctl -R /etc/audit/rules.d/audit.rules`

Config agent Wazuh (`/var/ossec/etc/ossec.conf`) :

```shell-session
<localfile>
    <location>/var/log/audit/audit.log</location>
    <log_format>audit</log_format>
</localfile>
```

## Task 10 Wazuh API

**Using Our Own Client**

API accessible via `curl` (auth requise -> token).

Authentification :

```shell-session
TOKEN=$(curl -u : -k -X GET "https://WAZUH_MANAGEMENT_SERVER_IP:55000/security/user/authenticate?raw=true")
```

Vérification :

```shell-session
curl -k -X GET "https://MACHINE_IP:55000/" -H "Authorization: Bearer $TOKEN"
```

Méthodes HTTP standards : `GET/POST/PUT/DELETE` via `-X`.

Exemples :

```shell-session
curl -k -X GET "https://MACHINE_IP:55000/manager/status?pretty=true" -H "Authorization: Bearer $TOKEN"
curl -k -X GET "https://MACHINE_IP:55000/manager/configuration?pretty=true&section=global" -H "Authorization: Bearer $TOKEN"
curl -k -X GET "https://MACHINE_IP:55000/agents?pretty=true&offset=1&limit=2&select=status%2Cid%2Cmanager%2Cname%2Cnode_name%2Cversion&status=active" -H "Authorization: Bearer $TOKEN"
```

**Using Wazuh's API Console**

Console intégrée : heading Wazuh -> "Tools". Requêtes prêtes à l'emploi, sélection + bouton run.

Syntaxe identique (méthodes HTTP + endpoints). Doc complète : [Wazuh API reference](https://documentation.wazuh.com/current/user-manual/api/reference.html)

## Task 11 Generating Reports with Wazuh

Module reporting : vue résumée des events sur un agent.

Ex : Security Events des dernières 24h -> **Modules -> Security Events** -> générer report (bouton grisé si pas de data -> ajuster range/query).

Accès rapport : **Wazuh -> Management -> Reporting** (sous "Status and Reports").

Téléchargement : icône save -> PDF.

Dashboard reporting direct : `https://machine_ip/app/wazuh#/manager/?tab=reporting`

## Task 12 Loading Sample Data

Données d'exemple non activées par défaut (perf).

Activation :

1. **Wazuh -> Settings -> Sample Data**
2. Bouton "Add Data" sur chaque carte
3. Import ~1 min/carte, bouton devient "Remove data" une fois fait

Nécessite date range >= Last 7 days + refresh dashboard pour voir les données.