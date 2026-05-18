- **Le maillon faible en cybersécurité = l’humain** : plus simple à “hacker” qu’un firewall.
    
- Quand un attaquant vise un humain, il cherche surtout **de l’accès** (boîte mail, M365, VPN, bases, réseau…).
    
- La room couvre : **attaques sur humains** + rôle **SOC** (détection/mitigation) + **2 scénarios pratiques**.
    

---
## Vocabulaire / concepts

|Terme|Définition courte|À retenir|
|---|---|---|
|**Social engineering**|Manipulation **psychologique** plutôt qu’exploitation technique|Repose sur **confiance** + **émotions** (urgence/peur/curiosité)|
|**Impersonation**|Se faire passer pour quelqu’un (IT, collègue, partenaire, CEO…)|Très efficace via téléphone/chat, souvent pour reset/OTP|
|**Mitigation**|Réduire la probabilité/l’impact d’une attaque|Formation + solutions anti-phishing/AV, etc.|
|**Detection**|Identifier/enquêter ce qui passe quand même|Cœur du job SOC : alertes, triage, investigation|

---
## Attaques humaines fréquentes

| Type                  | Mécanique                                                   | Le “truc” de l’attaquant                                | Indices fréquents                                                 |
| --------------------- | ----------------------------------------------------------- | ------------------------------------------------------- | ----------------------------------------------------------------- |
| **Phishing**          | Mail/lien/pièce jointe → vol d’identifiants/malware         | Se faire passer pour une entité “trusted” + urgence     | Domaine louche, PJ archive, mot de passe, demande d’action rapide |
| **Malware downloads** | Pousser à installer un “setup” via site/QR/SEO/FAKE CAPTCHA | Détourner un besoin légitime (installer X “vite”)       | Domaines “free apps”, installateurs non officiels                 |
| **Deepfakes**         | Audio/vidéo IA qui imite un proche/manager                  | Autorité + pression temporelle                          | Appels hors horaires, demandes inhabituelles (virement/reset)     |
| **Impersonation**     | Appel/chat “IT support / CEO / fournisseur”                 | Exploite la compliance + la peur de bloquer le business | Numéro masqué, pas de vérification, process contourné             |

---
## Défense : SOC = Détection + recommandations de mitigation

### 1) Mitigation (prévention/réduction)

- **Anti-phishing** (bloquer avant l’utilisateur)
    
- **Antivirus / EDR** (empêcher l’exécution)
    
- **Security awareness training** (former, simulations)
    
- Principe **“Trust but verify”** : vérifier les demandes sensibles (deepfake, reset, virement).
    

### 2) Detection (quand ça passe)

- Le SOC **triage** les alertes, **bloque/contain** si besoin, **investigue** (SIEM, logs, URLs, provenance mail), et **escalade**.
    

---
## Scénario pratique — réflexes “SOC”

|Cas|Red flags|Réponse attendue (logique)|
|---|---|---|
|Install “7-Zip” depuis un site freeware|Domaine louche + setup.exe bloqué AV|**Quarantaine** + orienter vers **source officielle/portal interne**|
|Mail “Stripe invoice” + .rar + mdp|Domaine fake, archive + mdp, montant élevé, pression|**Bloquer** + **analyser** comme phishing|
|CEO demande reset à 21h + numéro masqué|Hors horaire + contournement process + pas joignable ensuite|**Désactiver/locker compte** jusqu’à confirmation|
|Login anomalous + URLs suspectes (typo/ru)|Typosquatting + domaine .ru + parcours web louche|**Désactiver compte** + reset/IR, vérifier compromission|

---

# Here are a few great sites to follow:

- Krebs on Security - [https://krebsonsecurity.com](https://krebsonsecurity.com/)
- The Hacker News -  [https://thehackernews.com](https://thehackernews.com/)
- BleepingComputer - [https://www.bleepingcomputer.com](https://www.bleepingcomputer.com/)

