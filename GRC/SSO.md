 **Définition** : Mécanisme d'authentification permettant à un utilisateur de se connecter **une seule fois** pour accéder à plusieurs applications/services, sans avoir à se réauthentifier pour chacun.

---

## Concepts clés

|Terme|Définition|
|---|---|
|**Identity Provider (IdP)**|Entité qui authentifie l'utilisateur et émet des tokens/assertions (ex: Okta, Azure AD, Keycloak)|
|**Service Provider (SP)**|Application qui consomme l'authentification fournie par l'IdP (ex: Salesforce, GitHub)|
|**Principal**|L'utilisateur (ou entité) qui s'authentifie|
|**Token**|Jeton signé prouvant l'identité (JWT, SAML assertion…)|
|**Session SSO**|Session maintenue côté IdP après la première authentification|
|**SLO**|Single Log-Out — déconnexion centralisée de toutes les applications|

---

## Fonctionnement général

```
Utilisateur → SP (accès refusé) → Redirect vers IdP
                                        ↓
                              Authentification (login/MFA)
                                        ↓
                              Émission d'un token/assertion
                                        ↓
                     Token envoyé au SP → Accès accordé ✅
                                        ↓
            (Accès à SP2, SP3… sans ré-authentification grâce à la session IdP)
```

---

## Protocoles SSO

### SAML 2.0 (Security Assertion Markup Language)

- **Format** : XML
- **Usage** : Entreprises, fédérations d'identités (éducation, gouvernement)
- **Flux principal** : SP-initiated ou IdP-initiated
- **Composants** :
    - **Assertion** : contient les attributs de l'utilisateur
    - **AuthnRequest** : requête d'authentification du SP vers l'IdP
    - **Response** : assertion signée de l'IdP vers le SP
- **Transport** : HTTP POST / HTTP Redirect (binding)
- **Sécurité** : signature XML (RSA), chiffrement optionnel

```
SP → AuthnRequest (base64 + redirect) → IdP
IdP → Assertion XML signée (POST) → SP (ACS URL)
```

### OAuth 2.0

> ⚠️ OAuth est un protocole d'**autorisation**, pas d'authentification. Souvent combiné à OIDC pour le SSO.

- **Rôles** :
    - Resource Owner (utilisateur)
    - Client (application)
    - Authorization Server (IdP)
    - Resource Server (API)
- **Grant types** :
    - `Authorization Code` ← le plus sécurisé (web apps)
    - `Authorization Code + PKCE` ← pour apps mobiles/SPA
    - `Client Credentials` ← machine-to-machine
    - `Implicit` ← déprécié
    - `Resource Owner Password` ← déprécié
- **Tokens** : Access Token, Refresh Token

### OpenID Connect (OIDC)

- **Couche d'identité au-dessus d'OAuth 2.0**
- Ajoute un **ID Token** (JWT) contenant des claims sur l'utilisateur
- Endpoint clé : `/.well-known/openid-configuration`
- **Claims courants** : `sub`, `email`, `name`, `iss`, `aud`, `exp`, `iat`

json

```json
// Exemple de payload ID Token (JWT)
{
  "iss": "https://accounts.google.com",
  "sub": "1234567890",
  "email": "user@example.com",
  "aud": "client_id",
  "exp": 1716508800,
  "iat": 1716505200
}
```

### Kerberos

- **Usage** : environnements Windows/Active Directory
- **Acteurs** : Client, KDC (Key Distribution Center), Service
- **Tickets** : TGT (Ticket Granting Ticket) + Service Ticket
- **Flux** :
    1. Client → KDC : `AS-REQ` (authentification)
    2. KDC → Client : `TGT` chiffré
    3. Client → KDC : `TGS-REQ` (demande de service ticket)
    4. KDC → Client : Service Ticket
    5. Client → Service : présentation du ticket

---

## Comparatif des protocoles

|                    | SAML 2.0                        | OAuth 2.0    | OIDC             | Kerberos         |
| ------------------ | ------------------------------- | ------------ | ---------------- | ---------------- |
| **Objectif**       | Authentification + Autorisation | Autorisation | Authentification | Authentification |
| **Format**         | XML                             | JSON         | JSON (JWT)       | Tickets binaires |
| **Usage typique**  | SSO entreprise                  | API access   | SSO web moderne  | AD/Windows       |
| **Transport**      | HTTP                            | HTTP         | HTTP             | UDP/TCP          |
| **Émetteur token** | IdP (assertion)                 | Auth Server  | IdP (ID Token)   | KDC              |

---

## Flux Authorization Code (OIDC/OAuth)

```
User → App → GET /authorize?response_type=code&client_id=...&redirect_uri=...
                                    ↓
                          IdP : Login page
                                    ↓
                          User s'authentifie
                                    ↓
                  Redirect → App avec ?code=AUTH_CODE
                                    ↓
          App → POST /token (code + client_secret) → IdP
                                    ↓
                  IdP → Access Token + ID Token (+ Refresh Token)
                                    ↓
               App utilise l'Access Token pour appeler l'API ✅
```

---

## Avantages / Inconvénients

### ✅ Avantages

- Réduction de la fatigue des mots de passe (moins de credentials à gérer)
- Amélioration de l'UX
- Gestion centralisée des accès (révocation immédiate)
- Réduction de la surface d'attaque (moins de mots de passe stockés)
- Facilite l'application du MFA sur un point unique

### ❌ Inconvénients / Risques

- **Single Point of Failure** : si l'IdP tombe, plus rien n'est accessible
- **Single Point of Compromise** : si l'IdP est compromis, toutes les apps le sont
- Complexité d'implémentation
- Risque de **session hijacking** (vol de token)
- **Token replay attacks** si les tokens ne sont pas correctement validés

---

## Vecteurs d'attaque SSO

|Attaque|Description|Contre-mesure|
|---|---|---|
|**Token replay**|Réutilisation d'un token volé|Durée de vie courte, `jti` claim, nonce|
|**SAML Signature Wrapping (XSW)**|Manipulation de l'assertion XML sans invalider la signature|Validation stricte du schéma XML|
|**Open Redirect**|Détournement du redirect_uri pour voler le code OAuth|Whitelist stricte des redirect_uri|
|**IdP Spoofing**|Faux IdP qui récupère les credentials|Vérification du certificat, pinning|
|**Session fixation**|Fixer une session avant authentification|Renouveler le session ID après auth|
|**Pass-the-Ticket (Kerberos)**|Vol et réutilisation d'un TGT/ST|Durée de vie courte, monitoring|
|**Golden Ticket (Kerberos)**|Forge de TGT en compromettant le krbtgt|Rotation du compte krbtgt|

---

## Bonnes pratiques de sécurité

- Utiliser **PKCE** pour tous les clients publics (mobiles, SPA)
- Valider **tous les claims** du token (`iss`, `aud`, `exp`, `nbf`)
- Durée de vie courte des tokens (Access Token : 15 min recommandé)
- Utiliser des **Refresh Tokens rotatifs**
- Implémenter le **SLO** pour propager les déconnexions
- Activer le **MFA** sur l'IdP
- Surveiller les **connexions anormales** (géolocalisation, user-agent)
- Ne jamais stocker de tokens dans `localStorage` (XSS) → préférer `httpOnly` cookies

---

## Solutions / Produits courants

|Catégorie|Exemples|
|---|---|
|**IdP cloud**|Okta, Azure AD / Entra ID, Google Workspace, Auth0|
|**IdP open source**|Keycloak, Authentik, Dex, FreeIPA|
|**Protocole entreprise**|Active Directory Federation Services (ADFS)|
|**Standards**|SAML 2.0, OIDC, OAuth 2.0, SCIM (provisioning)|

---

## JWT — JSON Web Token (rappel)

Structure : `header.payload.signature` (base64url encodé)

```
eyJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJ1c2VyMSIsImV4cCI6MTcxNjUwODgwMH0.<signature>
```

- **header** : algo de signature (`RS256`, `ES256`…)
- **payload** : claims (données de l'utilisateur)
- **signature** : vérifiée avec la clé publique de l'IdP

> ⚠️ Un JWT n'est pas chiffré par défaut (juste signé). Ne pas y stocker de données sensibles.

---

## SCIM — Provisioning automatique

- **System for Cross-domain Identity Management**
- Protocole complémentaire au SSO pour **provisionner/déprovisionner** automatiquement les utilisateurs dans les SP
- API REST + JSON
- Évite les comptes orphelins lors des départs

---

## Points clés à retenir 🎯

1. SSO = **une authentification → plusieurs services** via un IdP central
2. **SAML** = XML, entreprise legacy ; **OIDC** = JWT, moderne
3. OAuth 2.0 ≠ authentification → toujours combiner avec OIDC
4. Kerberos = SSO natif Windows/AD avec des tickets
5. Risque principal : **compromission de l'IdP = compromission totale**
6. Toujours valider les tokens côté SP (signature, expiration, audience)
7. **PKCE** obligatoire pour clients publics