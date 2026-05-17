## Discovering Command Injection

Les applications passent des entrées utilisateur directement à des fonctions système (PHP, Python, NodeJS) → si l'entrée n'est pas filtrée, un attaquant peut injecter ses propres commandes.

![[command injection php.png]]

|#|Étape|
|---|---|
|1|The application stores MP3 files in a directory contained on the operating system|
|2|The user inputs the song title they wish to search for. The application stores this input into the `$title` variable|
|3|The data within this `$title` variable is passed to the command `grep` to search a text file named `songtitle.txt` for the entry of whatever the user wishes to search for|
|4|The output of this search of `songtitle.txt` will determine whether the application informs the user that the song exists or not|

**Exemple PHP** : `$title` → `grep` dans `songtitle.txt` 

![[command injection python.png]]

|#|Étape|
|---|---|
|1|The "flask" package is used to set up a web server|
|2|A function that uses the "subprocess" package to execute a command on the device|
|3|We use a route in the webserver that will execute whatever is provided. For example, to execute `whoami`, we'd need to visit `http://flaskapp.thm/whoami`|

**Exemple Python** : `subprocess` + Flask → `http://flaskapp.thm/<commande>`

La vulnérabilité est indépendante du langage : si l'app exécute l'input, elle est exploitable.

---

## Exploiting Command Injection

Shell operators `;`, `&`, `&&` permettent de chaîner des commandes → vecteur d'injection.

|Method|Description|
|---|---|
|Blind|No direct output — behaviour must be observed to determine success|
|Verbose|Direct feedback from the application once a payload is tested|

### Detecting Blind Command Injection

Pas d'output visible. Méthodes :

- **Time delay** : `ping`, `sleep` → l'app hang pendant x secondes
- **Redirection** : `whoami > file.txt` puis `cat file.txt`
- **curl** :

```
curl http://vulnerable.app/process.php%3Fsearch%3DThe%20Beatles%3B%20whoami
```

### Detecting Verbose Command Injection

L'app retourne directement l'output des commandes exécutées (ex: `whoami`, `ping`).

### Useful payloads

|Payload|Linux|Windows|
|---|---|---|
|`whoami`|User courant|User courant|
|`ls` / `dir`|Liste répertoire|Liste répertoire|
|`ping`|Blind — hang|Blind — hang|
|`sleep` / `timeout`|Blind (si pas de ping)|Blind (si pas de ping)|
|`nc`|Reverse shell|—|

---

## Remediating Command Injection

### Vulnerable Functions

En PHP, ces fonctions exécutent directement des commandes shell — dangereuses sans validation :

- `exec`
- `passthru`
- `system`

![[command injction vulnerable fonction.png]]

1. The application will only accept a specific pattern of characters (the digits  0-9)
2. The application will then only proceed to execute this data which is all numerical.

### Input sanitisation

Restreindre le format des données acceptées (ex: chiffres uniquement, suppression des caractères spéciaux `>`, `&`, `/`).

En PHP : `filter_input` vérifie si l'input est un nombre → sinon, rejeté. [PHP function](https://www.php.net/manual/en/function.filter-input.php)

![[command injection input sanitisation.png]]

### Bypassing Filters

Les filtres peuvent être contournés en changeant le format des données. Ex : une appli qui strip les guillemets → utiliser leur valeur hexadécimale à la place → même résultat à l'exécution.

![[command injection bypassing filter.png]]

---

## Ressources

Refer to [this cheat sheet(opens in new tab)](https://github.com/payloadbox/command-injection-payload-list) if you are stuck or wish to explore some more complex payloads.

