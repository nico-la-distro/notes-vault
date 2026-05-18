https://tryhackme.com/room/phishingyl

## Anchor

If you've set up a spoof website to harvest data or distribute malware, the links to this should be disguised using the **[anchor text(opens in new tab)](https://en.wikipedia.org/wiki/Anchor_text)** and changing it either to some text which says "Click Here" or changing it to a correct looking link that reflects the business you are spoofing, for example:

`<a href="http://spoofsite.thm">Click Here</a>`

`<a href="http://spoofsite.thm">https://onlinebank.thm</a>`

---
## Phishing infrastructure

**Domain Name:**

You'll need to register either an authentic-looking domain name or one that mimics the identity of another domain. See task 5 for details on how to create the perfect domain name.

**SSL/TLS Certificates:**

Creating SSL/TLS certificates for your chosen domain name will add an extra layer of authenticity to the attack.

**Email Server/Account:**

You'll need to either set up an email server or register with an SMTP email provider. 

**DNS Records:**

Setting up DNS Records such as SPF, DKIM, DMARC will improve the deliverability of your emails and make sure they're getting into the inbox rather than the spam folder.

**Web Server:**

You'll need to set up webservers or purchase web hosting from a company to host your phishing websites. Adding SSL/TLS to the websites will give them an extra layer of authenticity. 

**Analytics:**

When a phishing campaign is part of a red team engagement, keeping analytics information is more important. You'll need something to keep track of the emails that have been sent, opened or clicked. You'll also need to combine it with information from your phishing websites for which users have supplied personal information or downloaded software. 

**Automation And Useful Software:**

Some of the above infrastructures can be quickly automated by using the below tools.

**GoPhish - (Open-Source Phishing Framework) - [getgophish.com(opens in new tab)](https://getgophish.com/)  

GoPhish is a web-based framework to make setting up phishing campaigns more straightforward. GoPhish allows you to store your SMTP server settings for sending emails, has a web-based tool for creating email templates using a simple WYSIWYG (What You See Is What You Get) editor. You can also schedule when emails are sent and have an analytics dashboard that shows how many emails have been sent, opened or clicked.

**SET - (Social Engineering Toolkit) - [trustedsec.com(opens in new tab)](https://www.trustedsec.com/tools/the-social-engineer-toolkit-set/)  

The Social Engineering Toolkit contains a multitude of tools, but some of the important ones for phishing are the ability to create spear-phishing attacks and deploy fake versions of common websites to trick victims into entering their credentials.

![[phishing infrastructure.png]]

---

## Using GoPhish

### 🛠️ Setup & utilisation de GoPhish

#### 1. Connexion

- Lancer la VM
- Accéder à GoPhish via URL
- Identifiants :
    - **user** : admin
    - **password** : tryhackme

#### 2. Configuration principale

**📧 Sending Profile (SMTP)**

|Champ|Valeur|
|---|---|
|Name|Local Server|
|From|noreply@redteam.thm|
|Host|127.0.0.1:25|

👉 Sert à **envoyer les emails de phishing**

![[phishing gophish sending_profiles.png]]

**🌐 Landing Page (page frauduleuse)**

- Nom : **ACME Login**
- Page HTML = faux formulaire de login
- Options à cocher :
    - ✅ Capture Submitted Data
    - ✅ Capture Passwords

👉 Sert à **récupérer les identifiants**

![[phishing gophish landing_pages.png]]

**✉️ Email Template**

- Nom : **Email 1**
- Sujet : **New Message Received**
- Contenu :
    - Email **persuasif**
    - Lien affiché :  
        `https://admin.acmeitsupport.thm`
    - Lien réel :  
        `{{.URL}}` (redirige vers la fausse page)

👉 Technique clé : **spoofing de lien**

![[phishing gophish email templates.png]]

![[phishing gophish email templates 2.png]]

**👥 Users & Groups**

- Groupe : **Targets**

|Email|
|---|
|martin@acmeitsupport.thm|
|brian@acmeitsupport.thm|
|accounts@acmeitsupport.thm|
![[phishing gophish users & groups.png]]

#### 3. Lancement de la campagne

**🎯 Paramètres Campaign**

| Champ           | Valeur                     |
| --------------- | -------------------------- |
| Name            | Campaign One               |
| Template        | Email 1                    |
| Landing Page    | ACME Login                 |
| URL             | http://10[.]130[.]160[.]41 |
| Sending Profile | Local Server               |
| Groups          | Targets                    |

👉 Astuce lab : mettre date **2 jours avant**

![[phishing gophish campaigns.png]]

#### 4. Analyse des résultats

![[phishing analyse resultats.png]]

**📊 Indicateurs suivis**

|Statut|Signification|
|---|---|
|Sent|Email envoyé|
|Opened|Email ouvert|
|Clicked|Lien cliqué|
|Submitted Data|Identifiants volés|

**🔍 Observations**

- ✅ Martin → email envoyé
- ✅ Brian → **Submitted Data (compromis)**
- ❌ Accounts → erreur (user inconnu)

👉 En cliquant sur Brian :

- accès à **username + password**

### 🧠 Points clés à retenir

- GoPhish = outil complet de campagne phishing
- Étapes principales :
    1. SMTP (envoi)
    2. Landing page (piège)
    3. Email (leurre)
    4. Targets (victimes)
    5. Campaign (exécution)
- Objectif final : **capturer les credentials**
- KPI important : **Submitted Data**

---

## Droppers

Droppers are software that phishing victims tend to be tricked into downloading and running on their system. The dropper may advertise itself as something useful or legitimate such as a codec to view a certain video or software to open a specific file.

The droppers are not usually malicious themselves, so they tend to pass antivirus checks. Once installed, the intended malware is either unpacked or downloaded from a server and installed onto the victim's computer. The malicious software usually connects back to the attacker's infrastructure. The attacker can take control of the victim's computer, which can further explore and exploit the local network.

---

## Choosing a Phishing Domain

👉 Objectif : **paraître légitime pour tromper la victime + éviter les filtres spam**

### 🧠 Techniques principales

#### 1. Expired Domains (domaines expirés)

- Acheter un domaine avec **historique**
- ✔️ Avantage :
    - Meilleure réputation
    - Moins suspect pour les filtres spam
- ❌ Domaine neuf = plus facilement bloqué

#### 2. Typosquatting (fautes visuelles)

👉 Créer un domaine **très proche visuellement** de l’original

|Technique|Exemple|
|---|---|
|Fautes d’orthographe|goggle.com → google.com|
|Ajout de point|go.ogle.com → google.com|
|Chiffres à la place des lettres|g00gle.com → google.com|
|Variation de mot|googles.com → google.com|
|Mot supplémentaire|googleresults.com → google.com|

🧠 Point clé :  
➡️ Le cerveau humain **corrige automatiquement** → tromperie efficace

#### 3. TLD Alternatives

- Changer l’extension du domaine

|Original|Phishing|
|---|---|
|tryhackme.com|tryhackme.co.uk|

👉 Exploite le fait que les utilisateurs :

- regardent surtout le nom
- ignorent souvent le TLD

#### 4. IDN Homograph Attack (spoofing Unicode)

- Utilise des caractères **visuellement identiques** d’autres alphabets

|Type|Exemple|
|---|---|
|Latin|a|
|Cyrillique|а (différent mais identique visuellement)|

👉 Permet de créer :

- un domaine **quasi impossible à distinguer à l’œil**

---

## Using MS Office in Phishing

Often during phishing campaigns, a Microsoft Office document (typically Word, Excel or PowerPoint) will be included as an attachment. Office documents can contain macros; macros do have a legitimate use but can also be used to run computer commands that can cause malware to be installed onto the victim's computer or connect back to an attacker's network and allow the attacker to take control of the victim's computer.

**Take, for example, the following scenario:**

A staff member working for Acme IT Support receives an email from human resources with an excel spreadsheet called "Staff_Salaries.xlsx" intended to go to the boss but somehow ended up in the staff members inbox instead. 

What really happened was that an attacker spoofed the human resources email address and crafted a psychologically tempting email perfectly aimed to tempt the staff member into opening the attachment.

Once the staff member opened the attachment and enabled the macros, their computer was compromised.

---

## Using Browser in Phishing

Another method of gaining control over a victim's computer could be through browser exploits; this is when there is a vulnerability against a browser itself (Internet Explorer/Edge, Firefox, Chrome, Safari, etc.), which allows the attacker to run remote commands on the victim's computer.  
  
Browser exploits aren't usually a common path to follow in a red team engagement unless you have prior knowledge of old technology being used on-site. Many browsers are kept up to date, hard to exploit due to how browsers are developed, and the exploits are often worth a lot of money if reported back to the developers.  
  
That being said, it can happen, and as previously mentioned, it could be used to target old technologies on-site because possibly the browser software cannot be updated due to incompatibility with commercial software/hardware, which can happen quite often in big institutions such as education, government and especially health care.  
  
Usually, the victim would receive an email, convincing them to visit a particular website set up by the attacker. Once the victim is on the site, the exploit works against the browser, and now the attacker can perform any commands they wish on the victim's computer.

An example of this is [CVE-2021-40444(opens in new tab)](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-40444) from September 2021, which is a vulnerability found in Microsoft systems that allowed the execution of code just from visiting a website.

---

