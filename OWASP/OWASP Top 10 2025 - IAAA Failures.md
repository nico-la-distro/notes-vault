**OWASP** = The Open Web Application Security Project is a nonprofit foundation focused on understanding web technologies and exploitations and provides resources and tools designed to improve the security of software applications.

## IAAA

**IAAA** = mémo pour comprendre **comment une appli vérifie un utilisateur et ses actions**.  
**On ne peut pas “sauter” une étape** : si un niveau manque, les suivants ne tiennent pas.

| Élément            | Rôle                                                  | Exemples                             |
| ------------------ | ----------------------------------------------------- | ------------------------------------ |
| **Identity**       | Identifier unique qui représente une personne/service | userID, email                        |
| **Authentication** | Prouver qu’on est bien cette identité                 | mot de passe, OTP, passkey           |
| **Authorisation**  | Définir ce que l’identité a le droit de faire         | rôles, permissions                   |
| **Accountability** | Tracer et alerter sur qui fait quoi                   | logs, audit, alerting (qui/quand/où) |
**OTP** = _One-Time Password_

**Pourquoi c’est critique :** des failles IAAA peuvent permettre à un attaquant :

- d’accéder aux **données d’autres utilisateurs**
- de gagner des **privilèges** (escalade) au-delà de ce qui est permis

---

## A01 : Broken Access Control

- **Définition** : le serveur **n’applique pas correctement** les règles “qui a le droit d’accéder à quoi” **à chaque requête**.

- **Cas classique : IDOR (Insecure Direct Object Reference)**  
    Si modifier un identifiant dans l’URL / requête (ex. `?id=7` → `?id=6`) permet de **voir/éditer** les données d’un autre utilisateur → **contrôle d’accès cassé**.

| Concept                  | Description                                            | Exemple                         |
| ------------------------ | ------------------------------------------------------ | ------------------------------- |
| **IDOR**                 | Accès à un objet d’un autre user via un ID manipulable | changer `accountID` dans l’URL  |
| **Escalade horizontale** | même rôle, accès aux données d’un autre utilisateur    | user A voit le compte de user B |
| **Escalade verticale**   | accès à des actions réservées à un rôle supérieur      | user → actions admin            |

**Cause fréquente** : l’appli **fait trop confiance au client** (URL/params) au lieu de vérifier côté serveur.

**À retenir**
- Le contrôle d’accès doit être **côté serveur** et **sur chaque requête**.
- IDOR = **IDs manipulables ⇒ fuite/modif de données**.

