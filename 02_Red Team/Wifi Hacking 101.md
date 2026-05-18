https://tryhackme.com/room/wifihacking101

https://www.aircrack-ng.org/
# Intro to WPA

**Acronymes**

|Acronyme|Développement littéral|
|---|---|
|**SSID**|**Service Set Identifier**|
|**ESSID**|**Extended Service Set Identifier**|
|**BSSID**|**Basic Service Set Identifier**|
|**WPA**|**Wi-Fi Protected Access**|
|**WPA2**|**Wi-Fi Protected Access II**|
|**PSK**|**Pre-Shared Key**|
|**EAP**|**Extensible Authentication Protocol**|
|**RADIUS**|**Remote Authentication Dial-In User Service**|
|**WEP**|**Wired Equivalent Privacy**|
|**AP**|**Access Point**|
|**MAC**|**Media Access Control**|

**Termes**

|Terme|Signification|À quoi ça sert|
|---|---|---|
|**SSID**|Nom du réseau visible|Identifier le Wi-Fi dans la liste|
|**ESSID**|SSID “étendu” pouvant couvrir **plusieurs AP** (même réseau)|Dans **Aircrack**, souvent = réseau ciblé|
|**BSSID**|**MAC address** de l’**Access Point**|Identifier précisément **l’AP**|
|**WPA2-PSK**|“Personal” : **un seul mot de passe partagé**|Cas **maison** / invités / petites structures|
|**WPA2-EAP**|“Enterprise” : **login + mot de passe**|Auth via **serveur** (souvent entreprise)|
|**RADIUS**|Serveur d’authentification (pas que Wi-Fi)|Vérifie les identifiants en WPA2-EAP|
|**4-way handshake**|Échange WPA/WPA2 clé|Cœur de l’auth WPA(2)|

---

**Rappels sécurité / historique**

- **WEP** = ancien standard → **cassable** en capturant assez de paquets (attaque **statistique** pour retrouver la clé).
    
- **WPA vs WPA2** : auth **très similaire** → les **attaques** liées au handshake sont globalement les **mêmes**.

---

**Idée clé sur les clés WPA(2)-PSK**

- La clé n’est pas “juste” le mot de passe : elle est **dérivée de (ESSID + password)**.
    
- **ESSID = salt** → rend les attaques par **dictionnaire** plus coûteuses :
    
    - Le **même password** donne une **clé différente** selon le réseau (ESSID).
        
    - Donc sauf si tu as **pré-calculé** un dictionnaire **pour cet ESSID**, tu dois **tester** les mots de passe jusqu’au bon.

---
# Capturing packet to attack

- Objectif : **capturer des paquets** (notamment le **4-way handshake**) pour pouvoir attaquer un réseau **WPA** avec la suite **Aircrack-ng** (en supposant une carte Wi-Fi avec **monitor mode**).
    
- Suite **Aircrack-ng** : beaucoup d’outils, mais pour WPA on utilise surtout :
    
    - **airmon-ng** : activer/désactiver le **mode monitor**
        
    - **airodump-ng** : **sniffer / capturer** paquets + infos (BSSID, canaux, handshakes…)
        
    - **aircrack-ng** : tenter la **récupération du mot de passe** (via wordlist/attaque)
        
- Installation : présent par défaut sur **Kali**, sinon via gestionnaire de paquets ou site officiel.
    
- Entraînement conseillé : créer un **hotspot** (téléphone/tablette) avec un **mot de passe faible** (type rockyou.txt) pour reproduire toutes les étapes.
    
- Prérequis matériel :
    
    - **Monitor mode obligatoire** pour capturer le handshake (toutes les cartes ne le supportent pas).
        
    - **Injection mode** très utile : permet de **déconnecter (deauth)** un client pour le forcer à se reconnecter → le handshake se reproduit.
        
    - Sans injection : il faut **attendre** une connexion “naturelle” d’un client.
        

---
## Aircrack-ng suite (liste brute)

|Outil|Outil|
|---|---|
|aircrack-ng|airdecap-ng|
|airmon-ng|aireplay-ng|
|airodump-ng|airtun-ng|
|packetforge-ng|airbase-ng|
|airdecloak-ng|airolib-ng|
|airserv-ng|buddy-ng|
|ivstools|easside-ng|
|tkiptun-ng|wesside-ng|

---
## Outils “essentiels” pour WPA (à retenir)

|Outil|Rôle (mémo)|
|---|---|
|**airmon-ng**|activer le **monitor mode**|
|**airodump-ng**|**capture** + reconnaissance (AP/clients/handshake)|
|**aircrack-ng**|**attaque** WPA (wordlist/essais)|
|_(optionnel)_ **aireplay-ng**|**injection/deauth** pour forcer un handshake|

---
## Points clés à retenir (flash)

- **Handshake = cible** : sans lui, pas d’attaque WPA “classique”.
    
- **Monitor mode** = écouter / capturer.
    
- **Injection + deauth** = accélérer en forçant une reconnexion.