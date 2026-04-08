## Access Control

Le **contrôle d’accès (Access Control)** est un mécanisme de sécurité qui détermine **qui** (utilisateur/système) peut accéder à **quoi** (ressource) et **faire quoi** (actions).  
Objectif principal : **protéger les données sensibles** et limiter l’accès aux seuls utilisateurs autorisés.

### Types de contrôle d'accès

|Type|Principe|Qui décide ?|Usage typique|Image mentale|
|---|---|---|---|---|
|**DAC** (Discretionary Access Control)|Le propriétaire/admin attribue les permissions|Propriétaire de la ressource|OS, systèmes de fichiers|Le roi distribue des clés|
|**MAC** (Mandatory Access Control)|Règles strictes imposées par le système|Politique système (non négociable)|Militaire, gouvernement, environnements très sécurisés|Forteresse avec règles rigides|
|**RBAC** (Role-Based Access Control)|Accès selon le **rôle** de l’utilisateur|Organisation / rôles métier|Entreprises, SI internes|Accès selon poste (manager, sales, etc.)|
|**ABAC** (Attribute-Based Access Control)|Accès selon des **attributs** (rôle, heure, lieu, device…)|Moteur de politique basé sur attributs|Cloud, applis web modernes|Contrôle intelligent et contextuel|

**DAC**
![[DAC (Discretionary Access Control).png]]

**MAC**
![[MAC (Mandatory Access Control).png]]

**RBAC**
![[RBAC (Role-Based Access Control).png]]

**ABAC**
![[ABAC (Attribute-Based Access Control).png]]

---

## Broken Access Control

Une vulnérabilité de **Broken Access Control** apparaît quand l’application **n’applique pas correctement les restrictions d’accès**.

➡️ Résultat : un utilisateur peut accéder à des ressources / actions qu’il **ne devrait pas** pouvoir voir ou exécuter.

**Exploits courants**

|Exploit|Définition|Exemple classique|Impact|
|---|---|---|---|
|**Horizontal Privilege Escalation**|Accès aux ressources d’un **autre utilisateur de même niveau**|Changer un `userID` dans l’URL pour voir le compte d’un autre user|Violation de confidentialité / prise de compte partielle|
|**Vertical Privilege Escalation**|Accès aux ressources d’un **niveau supérieur** (ex. admin)|Manipuler un paramètre URL/champ caché pour accéder à une page admin|Compromission majeure (admin actions)|
|**Insufficient Access Control Checks**|Vérifications d’accès absentes / incohérentes|Une page sensible s’affiche sans validation de permission|Bypass de sécurité, exposition de données|
|**IDOR** (Insecure Direct Object References)|Accès direct à un objet via un identifiant prévisible/guessable|IDs séquentiels permettant d’accéder à des documents d’autres users|Exfiltration de données, accès non autorisé|
**A Retenir**

- **Ne jamais faire confiance au client** (URL, champs cachés, paramètres modifiables)

- Les contrôles d’accès doivent être :
    - **côté serveur**
    - **systématiques**
    - **cohérents** sur toutes les routes / fonctions

- Tester :
    - changement d’ID (`/profile?id=124`)
    - accès direct à endpoints admin
    - ressources sans UI mais accessibles via URL
    - IDs prédictibles (IDOR)

___

## Mitigation PHP

Pour réduire le risque de **Broken Access Control** en PHP, il faut combiner :

- **contrôle d’accès robuste (RBAC)**
- **sécurisation des requêtes SQL**
- **gestion de session correcte**
- **secure coding practices**

|Mesure|But principal|Exemple / idée clé|À retenir|
|---|---|---|---|
|**RBAC** (Role-Based Access Control)|Limiter les actions selon le rôle|`admin`, `editor`, `user` avec permissions définies|Vérifier les permissions **avant chaque action sensible**|
|**Requêtes paramétrées**|Réduire le risque de SQL Injection|`prepare()` + `execute()` au lieu de concaténer l’entrée utilisateur|Ne jamais injecter directement `$_POST` dans SQL|
|**Bonne gestion de session**|Empêcher accès non autorisé via sessions faibles/expirées|`session_start()`, timeout, destruction de session expirée|Timeout + cookies sécurisés + validation session|
|**Secure coding practices**|Réduire les vulnérabilités applicatives|Validation/sanitization input + `password_hash()`|Éviter fonctions faibles (`md5`)|

_ressource : https://cheatsheetseries.owasp.org/cheatsheets/PHP_Configuration_Cheat_Sheet.html / https://phptherightway.com/#security / https://www.php.net/manual/en/security.php_ 

### 1) RBAC (Role-Based Access Control)

**Idée clé**

Définir des **rôles** et leurs **permissions**, puis vérifier l’autorisation avant une action.

```php
// Define roles and permissions  
$roles = [  
    'admin' => ['create', 'read', 'update', 'delete'],  
    'editor' => ['create', 'read', 'update'],  
    'user' => ['read'],  
];  
  
// Check user permissions  
function hasPermission($userRole, $requiredPermission) {  
    global $roles;  
    return in_array($requiredPermission, $roles[$userRole]);  
}  
  
// Example usage  
if (hasPermission('admin', 'delete')) {  
    // Allow delete operation  
} else {  
    // Deny delete operation  
}
```

**À retenir**

- **Centraliser** la logique d’autorisation
- Faire les checks **côté serveur**
- Vérifier **sur chaque endpoint/action**, pas seulement dans l’UI

---
### 2) Requêtes paramétrées (Prepared Statements)

**Idée clé**

Éviter la concaténation directe de l’input utilisateur dans SQL (anti-SQLi).

**Exemple vulnérable**

```php
$username = $_POST['username'];  
$password = $_POST['password'];  
$query = "SELECT * FROM users WHERE username='$username' AND password='$password'";

### Exemple sécurisé (prepared statement)

$username = $_POST['username'];  
$password = $_POST['password'];  
  
$stmt = $pdo->prepare("SELECT * FROM users WHERE username=? AND password=?");  
$stmt->execute([$username, $password]);  
$user = $stmt->fetch();
```

**À retenir**

- `prepare()` + `execute()` = base
- Ce point cible surtout **SQL Injection**, mais contribue à éviter des accès non autorisés indirects

---
### 3) Proper Session Management

**Idée clé**

Maintenir des sessions valides et expirer celles inactives.

```php
// Start session  
session_start();  
  
// Set session variables  
$_SESSION['user_id'] = $user_id;  
$_SESSION['last_activity'] = time();  
  
// Check if session is still valid  
if (isset($_SESSION['last_activity']) && (time() - $_SESSION['last_activity'] > 1800)) {  
    // Session has expired  
    session_unset();  
    session_destroy();  
}
```

**À retenir**

- **1800 s = 30 min** d’inactivité
- Vérifier la session **avant** l’accès aux ressources sensibles
- Ajouter en pratique : cookies sécurisés, regeneration d’ID de session, etc.

---
### 4) Secure Coding Practices

**Idée clé**

Valider / assainir les entrées et utiliser des fonctions sûres.

```php
// Validate user input  
$username = filter_input(INPUT_POST, 'username', FILTER_SANITIZE_STRING);  
$password = filter_input(INPUT_POST, 'password', FILTER_SANITIZE_STRING);  
  
// Avoid insecure functions  
// Example of vulnerable code using md5  
$password = md5($password);  
  
// Example of secure code using password_hash  
$password = password_hash($password, PASSWORD_DEFAULT);
```

**À retenir**

- ❌ `md5()` pour les mots de passe
- ✅ `password_hash()` (et `password_verify()` en pratique)
- La sanitization **ne remplace pas** les contrôles d’accès côté serveur

