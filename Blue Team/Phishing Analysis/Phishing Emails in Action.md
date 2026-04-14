
## Cancel your order
### 🎯 Objectif du mail

Email de phishing imitant un **reçu PayPal** pour pousser l’utilisateur à agir rapidement.

### 🧠 Techniques de phishing utilisées

|Technique|Description|But|
|---|---|---|
|Spoofed email|Fausse adresse se faisant passer pour PayPal|Gagner la confiance|
|URL shortening|Lien raccourci masquant la vraie destination|Cacher le site malveillant|
|Branded HTML|Design similaire à PayPal|Renforcer la crédibilité|

### 🔍 Premières observations (header)

|Élément|Indice suspect|
|---|---|
|Subject|Crée un sentiment d’urgence (fausse transaction)|
|From|Faux : affiche PayPal mais adresse réelle différente|
|To|Adresse inhabituelle / incohérente|

👉 **Red flag majeur : mismatch entre adresse affichée et réelle**

![[phishing email in action mismatch.png]]

### 📧 Analyse du contenu

- Email = **faux reçu d’achat (gift cards)**
- Pas de pièce jointe
- **1 seul élément interactif : bouton "Cancel the order"**

👉 Technique classique : pousser à cliquer vite

![[phishing email in action body analysis.png]]
### 🔗 Analyse du bouton

|Élément|Risque|
|---|---|
|Lien raccourci|Destination cachée|
|Redirection|Impossible de vérifier sans outil|

👉 **Règle clé :**

> Ne jamais cliquer sans vérifier la destination réelle

![[phishing email in action button investigation.png]]

### 🛠️ Outils utiles

- **[WhereGoes(opens in new tab)](https://wheregoes.com/)** : permet de voir la destination d’un lien raccourci **sans cliquer**

---

## Track Your Package

### 🎯 Objectif du mail

Email de phishing imitant un **suivi de colis** pour créer urgence et inciter au clic.

![[phishing email in action trackyourpackage.png]]
### 🧠 Techniques de phishing utilisées

|Technique|Description|But|
|---|---|---|
|Spoofed email|Faux expéditeur (centre de distribution)|Crédibilité|
|Pixel tracking|Image invisible qui se charge à l’ouverture|Savoir si l’email est lu|
|Link manipulation|Faux lien (numéro de suivi)|Rediriger vers site malveillant|

### 🔍 Premières observations

|Élément|Indice suspect|
|---|---|
|Subject|Faux numéro de tracking → urgence|
|From|Nom crédible ≠ adresse réelle (domaine suspect)|
|Lien|Correspond au tracking mais destination inconnue|

👉 **Red flag : incohérence expéditeur + lien douteux**

### 🖼️ Images bloquées (important)

- Yahoo **bloque automatiquement les images**
- Pourquoi ?
    - Contiennent souvent des **tracking pixels**
    - Permettent de savoir :
        - si l’email est ouvert
        - quand et où

👉 **Indice fort de phishing / spam**

### 🔗 Analyse du lien (via code source)

|Élément|Risque|
|---|---|
|Image cliquable (Tracking.png)|Cache un lien|
|Tracking pixel|Envoie des infos au hacker|
|Redirection|Destination réelle masquée|

👉 Impossible de vérifier juste avec un hover → analyse du code nécessaire

![[phishing email in action analyse du lien.png]]

---

## Download Document Here

### 🎯 Objectif du mail

Phishing visant à **voler des identifiants (credentials)** via une **chaîne de redirections** imitant des services connus.

### 🧠 Techniques utilisées

|Technique|Description|But|
|---|---|---|
|Artificial urgency|Lien qui expire le jour même|Forcer à agir vite|
|Brand impersonation|Imitation de Microsoft / Adobe / OneDrive|Rassurer la victime|
|Link redirection|Plusieurs redirections successives|Cacher la vraie destination|
|Credential harvesting|Faux portail de connexion|Voler login + mot de passe|

### 🔍 Premières observations

|Élément|Indice suspect|
|---|---|
|Date d’envoi / expiration|Expire le même jour → pression|
|Bouton|"Download Document Here" → incitation à cliquer|
|Contexte|Fax/document inattendu|

👉 **Combo classique : urgence + action immédiate**

![[phishing email in action fake mail.png]]
### 🔗 Chaîne d’attaque (très important)

|Étape|Description|Objectif|
|---|---|---|
|1|Email avec bouton|Piéger le clic|
|2|Faux OneDrive|Mettre en confiance|
|3|Faux Adobe|Renforcer crédibilité|
|4|Faux login (ex: Outlook)|Voler les identifiants|

![[phishing email in action fake adobe.png]]

### 🔐 Phase de vol d’identifiants

- Page de connexion **fausse**
- Peu importe les identifiants → **toujours erreur**
- Pourquoi ?
    - Les données sont **envoyées directement à l’attaquant**
    - Aucun vrai système d’authentification derrière

![[phishing email in action fake login.png]]
### ⚠️ Points clés de détection

- URL suspecte malgré branding connu
- Instructions incohérentes
- Multiples redirections
- Demande de login inattendue

👉 **Ne jamais entrer ses identifiants après un lien email**

---

## Your Account is on Hold

### 🎯 Objectif du mail

Phishing imitant un problème de **facturation Netflix** pour pousser la victime à ouvrir une **pièce jointe malveillante**.

### 🧠 Techniques utilisées

|Technique|Description|But|
|---|---|---|
|Spoofed email|Faux nom “Netllx billing”|Se faire passer pour Netflix|
|Urgency|Compte suspendu|Forcer une action rapide|
|Brand impersonation|Logos + HTML Netflix|Créer la confiance|
|Attachment-based attack|PDF piégé|Contourner les filtres|
|Poor grammar|Fautes et typos|Indice de phishing|
|Fake trusted domain|Lien “help center” détourné|Donner une illusion de légitimité|

### 🔍 Premières observations

|Élément|Indice suspect|
|---|---|
|Subject|Compte suspendu → urgence|
|From|Nom ≠ domaine réel|
|Branding|Netflix imité via HTML|
|Orthographe|“Netllx” mal écrit|

👉 **Red flags combinés = forte probabilité de phishing**

![[phishing email in action fake netflix.png]]
### 📎 Analyse de la pièce jointe

|Élément|Risque|
|---|---|
|PDF|Apparemment légitime|
|Bouton dans PDF|“Update Payment Account”|
|Lien intégré|Redirige vers domaine non officiel|

👉 Le PDF sert de **cache pour un lien malveillant**

![[phishing email in action netflix analyse.png]]
## ⚠️ Autres indices importants

- Numéro de téléphone **format inhabituel**
- Utilisation d’un domaine “help center” pour rassurer
- Mélange de contenu légitime + malveillant

👉 Technique : **hybrid trust abuse** (mélange vrai/faux)

---

## Your Recent Purchase

