
- Les attaquants essaient de **laisser le moins de traces possibles**, mais les équipes sécu peuvent **reconstituer l’attaque** grâce aux **traces** laissées (indices).  

- Dans le numérique, ces traces sont surtout dans les **logs** → **empreintes digitales** de toute activité (normale ou malveillante), utiles pour **tracer l’action** et parfois **l’auteur**.

## **Logs definition**

- **Logs = “digital footprints”** : enregistrements d’événements/activités sur un système ou une application.

- Servent à **retrouver quoi / quand / où / comment** une action a eu lieu.

**Problème + solution (idée clé)**

- Problème : un fichier de logs contient **énormément d’événements** → on se perd vite.

- Solution : les logs sont **classés par catégories** → on consulte **le bon type** selon l’incident.
    - Exemple : enquête sur des **logins réussis** sur Windows → regarder **Security Logs** (pas tout le reste).

---

|Use case|À quoi ça sert concrètement|
|---|---|
|**Security Events Monitoring**|Détecter des comportements **anormaux** via du **monitoring temps réel**|
|**Incident Investigation & Forensics**|Traces détaillées pour comprendre l’incident + **root cause analysis**|
|**Troubleshooting**|Diagnostiquer des **erreurs** système/app et aider à corriger|
|**Performance Monitoring**|Indices sur la **performance** des applications/systèmes|
|**Auditing & Compliance**|Construire une **piste d’audit** (qui a fait quoi) pour conformité|

## **Types of logs**

|Type de log|Usage|Exemples d’événements|
|---|---|---|
|**System Logs**|Troubleshooting / activités OS|démarrage/arrêt, chargement drivers, erreurs système, événements hardware|
|**Security Logs**|Détection + investigation sécu|authentification, autorisation, changements de policy, changements de comptes, activités anormales|
|**Application Logs**|Événements liés à une appli|interactions utilisateur, changements, mises à jour, erreurs applicatives|
|**Audit Logs**|Traçabilité détaillée (compliance + monitoring)|accès aux données, changements système, activités utilisateur, enforcement de policy|
|**Network Logs**|Trafic entrant/sortant + troubleshooting réseau|trafic entrant/sortant, connexions réseau, firewall|
|**Access Logs**|Accès à des ressources|web access logs, DB access logs, app/API access logs|

## **Windows Event Logs Analysis**

Windows enregistre beaucoup d’activités dans des **logs séparés par catégories**. Les plus importants :

|Log|À quoi ça sert|Exemples|
|---|---|---|
|**Application**|Infos liées aux applications|erreurs, warnings, problèmes de compatibilité|
|**System**|Infos liées au fonctionnement de l’OS|drivers/hardware, démarrage/arrêt, services|
|**Security**|**Le plus important côté sécu**|authentification, changements de comptes, changements de policy|

### **Event viewer (outil Windows)**

Contrairement à d’autres logs “bruts”, Windows a une GUI intégrée : **Event Viewer** (Observateur d’événements) pour **voir + chercher + filtrer**.

- Ouvrir : Start → taper **“Event Viewer”**
- Dans **Windows Logs**, tu retrouves **Application / System / Security**
- Interface (logique) :
    1. liste des fichiers de logs
    2. liste des événements du log sélectionné
    3. options d’analyse (dont filtres)

---

**structure d'un event log (champs clés)**

|Champ|Rôle|
|---|---|
|**Description**|détails de l’activité|
|**Log Name**|nom du log (Security/System/…)|
|**Logged**|timestamp de l’événement|
|**Event ID**|identifiant unique d’un type d’activité (super utile pour enquêter)|
**Exemple important** : **4624 = login réussi** → pour enquêter sur les logins réussis, tu filtres directement sur **4624**.

---

**Event IDs Windows (à connaitre)**

|Event ID|Signification|
|--:|---|
|**4624**|login réussi|
|**4625**|login échoué|
|**4634**|logoff réussi|
|**4720**|création de compte|
|**4724**|tentative de reset mot de passe|
|**4722**|compte activé|
|**4725**|compte désactivé|
|**4726**|compte supprimé|

## **Web Server Access Logs Analysis**

- Chaque action sur un site = **requête HTTP** (visiter, login, upload, etc.)
- Le serveur **log toutes les requêtes** dans un fichier d’accès (ex : Apache)
    - chemin typique : **`/var/log/apache2/access.log`**

---

**Champs d’un access log Apache**

|Champ|Exemple|À quoi ça sert|
|---|---|---|
|**IP address**|`172.16.0.1`|identifier la source de la requête|
|**Timestamp**|`[06/Jun/2024:13:58:44]`|quand la requête a eu lieu|
|**Request**|`"GET /products HTTP/1.1"`|action + ressource + version HTTP|
|**HTTP Method**|`GET`|type d’action (récupérer, envoyer, etc.)|
|**URL**|`/` , `/about`|ressource demandée|
|**Status code**|`200`, `404`, `500`|résultat côté serveur (OK / not found / erreur serveur…)|
|**User-Agent**|`Mozilla/5.0 ...`|infos navigateur + OS (utile pour profiling / détection)|

---

**Analyse manuelle sous Linux : commandes essentielles**

|Commande|Usage|Exemple|
|---|---|---|
|**cat**|afficher le contenu d’un log|`cat access.log`|
|**cat** (fusion)|combiner plusieurs logs (utile si logs “rotatés”)|`cat access1.log access2.log > combined_access.log`|
|**grep**|rechercher une chaîne/pattern (IP, URL, code…)|`grep "192.168.1.1" access.log`|
|**less**|lire un gros fichier **page par page**|`less access.log`|
**Navigation dans `less` (à retenir)**

- `Space` : page suivante
- `b` : page précédente
- `/pattern` : chercher
- `n` : occurrence suivante
- `N` : occurrence précédente

