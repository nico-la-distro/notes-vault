## The Email Adress
### 📧 Structure d’une adresse email

Une adresse email est **le premier élément à analyser en phishing** → des indices importants peuvent s’y cacher.

### 🧩 Composition d’une adresse email

|Élément|Rôle|
|---|---|
|**Username**|Identifie la boîte mail du destinataire (ex: `david`)|
|**@**|Séparateur + indique au système où envoyer le message|
|**Domain**|Serveur de messagerie du destinataire (ex: `tryhackme.com`)|

👉 Exemple : `david@tryhackme.com`

- Username = `david`
- Domain = `tryhackme.com`

### 🧠 Analogie simple

|Élément email|Équivalent réel|
|---|---|
|Domain|Rue / immeuble|
|Username|Personne / boîte aux lettres|

➡️ Permet au serveur mail de livrer correctement le message.

### 🎯 Point clé pour le phishing

- Une **mauvaise compréhension du format** = risque de rater des indices
- Les attaquants jouent souvent sur :
    - des **domaines similaires** (ex: `paypaI.com`)
    - des **usernames trompeurs**

---

## Email Delivery

### 📬 Protocoles d’email

|Protocole|Rôle|Fonction principale|
|---|---|---|
|**SMTP**|Envoi|Envoie les emails|
|**POP3**|Réception|Télécharge les emails sur **un seul appareil**|
|**IMAP**|Réception|Synchronise les emails sur **plusieurs appareils**|

### ⚖️ POP3 vs IMAP

|Caractéristique|**POP3**|**IMAP**|
|---|---|---|
|Stockage|Appareil local|Serveur|
|Accès|Un seul device|Plusieurs devices|
|Synchronisation|❌ Non|✅ Oui|
|Emails serveur|Supprimés après download|Restent sur le serveur|
|Emails envoyés|Stockés localement|Stockés sur le serveur|

### 🔄 Parcours d’un email

1. User sends an email: The sender’s email client sends the message to their mail server using SMTP
2. Mail server queries DNS: The sending server asks DNS for the recipient domain’s mail server
3. DNS responds: DNS returns the address of the recipient’s mail server
4. Email is delivered: The message is sent across the Internet to the recipient’s server
5. The recipient checks their mailbox: The recipient’s email client connects to their mail server
6. Email is retrieved: The message is downloaded (POP3) or synced (IMAP) to the recipient’s device

![[phishing analysis email's journey.png]]

### 🎯 Points clés pour le phishing

- Comprendre le chemin permet de :
    - identifier **où un email peut être falsifié**
    - analyser les **headers (traces SMTP, serveurs, etc.)**
- SMTP est souvent exploité pour :
    - **spoofing (usurpation d’adresse)**

---

## Email Headers

### 📧 Structure d’un email

Un email est composé de **2 parties principales** :

|Partie|Contenu|
|---|---|
|**Header**|Métadonnées (infos techniques)|
|**Body**|Contenu du message (texte / HTML)|

### 🧩 Champs importants du header

|Champ|Rôle|
|---|---|
|**From**|Expéditeur|
|**To**|Destinataire|
|**Reply-To**|Adresse de réponse (peut être différente → suspect)|
|**Subject**|Sujet|
|**Date**|Date et heure d’envoi|
![[phishing analysis champs emails.png]]

### 🔍 Message Source (très important)

Permet d’afficher :

- **Tous les headers complets**
- **Le contenu brut (raw)**
- Des infos cachées comme :
    - **IP d’origine**
    - **serveurs traversés**
    - **détails SMTP**

📌 Accès :

- Menu → _View → Message Source_
- Raccourci → **Ctrl + U**

![[phishing analysis massage view.png]]

![[phishing analysis message view metadata.png]]

### 🎯 Points clés pour le phishing

- Le header = **preuve technique principale**
- À vérifier en priorité :
    - incohérence **From vs Reply-To**
    - **IP d’origine suspecte**
    - serveurs inhabituels

➡️ Beaucoup d’éléments ne sont **pas visibles dans la vue normale**

---

## Email Body

### 📩 Email Body (contenu du message)

- Contient le **message principal**
- Deux formats :
    - **Texte simple**
    - **HTML** (images, liens, mise en forme)

👉 Les clients mail affichent une **version rendue** (visuelle), pas le code réel.

### 🔍 Analyse du HTML

|Vue|Description|
|---|---|
|**Rendue**|Ce que voit l’utilisateur|
|**Source HTML**|Code réel (liens, images, scripts cachés)|

📌 Intérêt :

- Voir les **vrais liens (href)**
- Détecter :
    - liens masqués
    - images externes
    - contenu suspect

⚠️ Les images peuvent être **bloquées par défaut** (ex: Thunderbird)

![[phishing analysis view html source code.png]]

### 📎 Analyse des pièces jointes

Dans le **source email**, une pièce jointe contient :

|Champ|Rôle|
|---|---|
|**Content-Type**|Type de fichier (ex: PDF)|
|**Content-Disposition**|Indique que c’est une pièce jointe + nom|
|**Content-Transfer-Encoding**|Encodage (souvent **base64**)|

![[phshing analysis piece jointe analyse.png]]

### 🔄 Base64 (important)

- La pièce jointe est **encodée en base64**
- Le bloc de texte = fichier converti
- Peut être :
    - **décodé** → reconstituer le fichier original
    - analysé avec outils (ex: CyberChef)

### 🎯 Points clés pour le phishing

- Le HTML peut :
    - masquer un **lien malveillant**
    - afficher un faux contenu
- Les pièces jointes peuvent contenir :
    - malware
    - documents piégés

➡️ Toujours vérifier :

- **code HTML réel**
- **encodage des fichiers**
- **type réel vs apparent**

---

## Type of phishing

### 🎣 Types de phishing

|Type|Description|
|---|---|
|**Spam / Malspam**|Emails massifs non sollicités (malspam = version malveillante)|
|**Phishing**|Usurpation d’identité pour voler des infos|
|**Spear Phishing**|Attaque **ciblée** (personne/entreprise spécifique)|
|**Whaling**|Cible des **dirigeants** (CEO, CFO)|
|**Smishing**|Phishing via **SMS**|
|**Vishing**|Phishing via **appel vocal**|

### 🧩 Caractéristiques d’un email de phishing

|Indice|Explication|
|---|---|
|**From spoofé**|Faux expéditeur (ex: microsof.com)|
|**Urgence**|Pousse à agir vite (pression)|
|**Usurpation de marque**|Logos / design crédibles|
|**Fautes / style bizarre**|Moins fréquent avec l’IA|
|**Message générique**|“Dear Customer”|
|**Liens masqués**|URL cachée ou raccourcie|
|**Pièces jointes malveillantes**|Faux fichiers (ex: `.pdf.exe`)|

### 🎯 Objectifs des attaquants

- Vol de **credentials**
- Installation de **malware**
- Accès non autorisé

➡️ Toujours via **social engineering**

### 🔐 Safe Analysis (TRÈS important)

👉 Ne jamais cliquer directement

#### 🔧 Defanging (rendre inoffensif)

|Original|Defanged|
|---|---|
|`http://site.com`|`hxxp[://]site[.]com`|
|`user@mail.com`|`user[@]mail[.]com`|

➡️ Empêche les clics accidentels

---

## Conclusion

In this room, you explored the anatomy of an email, starting with the structure of an email address and the path a message takes from sender to recipient. You then developed the technical skills needed to investigate suspicious emails by extracting and analyzing both the header and body source code. Finally, you examined common phishing techniques and applied your knowledge to analyze real-world examples, identifying attacker intent. Nicely done!

Before moving on, it’s also important to understand Business Email Compromise (BEC), a type of attack where an adversary gains access to a legitimate internal email account and uses it to trick others into performing unauthorized or fraudulent actions.


