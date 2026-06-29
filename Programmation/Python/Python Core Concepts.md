https://tryhackme.com/room/pythoncoreconcepts
## Introduction

Suite de [Python: Simple Demo](https://tryhackme.com/room/pythonsimpledemo) (variables, conditions, `while`). 

Scenario : checker 10000 usernames vs passwords par defaut -> besoin de strings/listes/dicts/operators/for loop.

Forme un duo avec [Python: Building Scripts](https://tryhackme.com/room/pythonbuildingscripts) (functions, error handling, files, libraries, Password Strength Checker).

### Learning Objectives

- Review variables, conditionals, loops
- Data types + type conversions
- F-strings + augmented assignment operators
- String methods
- Listes et dictionnaires
- Operators : comparison, logical, arithmetic, membership
- `for` loops, `while` loops, `range()`
- `break` et `continue`

---

## Quick Review: Hello World, Variables, and Conditionals

### Hello World

`print()` affiche du texte. `#` = commentaire (ignoré par Python). Texte (string) entre guillemets `"` ou `'`.

```python
# commentaire
print("Hello World")
```

Fichier : `hello.py`

### Variables and Data Types

Variables stockent des données, modifiables.

```python
food = "ice cream"   # string
money = 2000          # integer
age = 30
age = age + 1         # 31
```

|Type|Name|Example|Description|
|---|---|---|---|
|str|String|"hello"|Texte|
|int|Integer|42|Nombre entier|
|float|Float|3.14|Nombre décimal|
|bool|Boolean|True / False|Logique|
|list|List|[1, 2, 3]|Collection ordonnée|

`type()` renvoie le type d'une valeur.

```python
username = "admin"
port = 8080
print(type(username))   # <class 'str'>
print(type(port))       # <class 'int'>
```

Fichier : `types_demo.py`

### Conditional Statements and Logical Operators

```python
age = 18
if age < 17:
    print("You are NOT old enough to drive")
else:
    print("You are old enough to drive")
```

Structure : `if` + condition + `:`, bloc indenté, `else:` optionnel, `elif CONDITION:` pour checks additionnels.

Comparison operators :

|Operator|Meaning|Example|
|---|---|---|
|==|Equal to|x == 5|
|!=|Not equal to|x != 5|
|<|Less than|x < 5|
|>|Greater than|x > 5|
|<=|Less than or equal|x <= 5|
|>=|Greater than or equal|x >= 5|

Logical operators :

|Operator|Meaning|Example|
|---|---|---|
|and|True si les deux côtés vrais|x > 0 and x < 100|
|or|True si au moins un côté vrai|x == 1 or x == 10|
|not|Inverse le booléen|not is_locked|

```python
name = "bob"
hungry = True

if name == "bob" and hungry == True:
    print("Bob is hungry")
elif name == "bob" and not hungry:
    print("Bob is not hungry")
else:
    print("Not sure who this is or if they are hungry")
```

`=` assigne une valeur, `==` compare deux valeurs (erreur fréquente de débutant).

## What's New: Type Conversion and f-strings

### Type Conversion

`input()` retourne toujours un string. Conversions :

|Function|Converts To|Example|
|---|---|---|
|int()|Integer|int("42") -> 42|
|float()|Float|float("3.14") -> 3.14|
|str()|String|str(42) -> "42"|
|bool()|Boolean|bool(0) -> False|

```python
text = input("Enter a port number: ")    # "443"
print(type(text))                         # <class 'str'>

port = int(text)
print(type(port))                         # <class 'int'>
print(port + 1)                           # 444
```

Sans `int()`, `text + 1` provoque une erreur.

### Formatted Strings (f-strings)

Alternative moderne aux virgules dans `print()`. Préfixe `f` avant la quote, variables entre `{}`.

```python
username = "admin"
port = 443

# Ancienne approche
print("User", username, "is on port", port)

# f-string
print(f"User {username} is on port {port}")
```

Même output : `User admin is on port 443`. Utilisé tout au long du room.

### Augmented Assignment Operators

Raccourcis pour modifier une variable.

```python
count = 0
count += 1    # count = count + 1
count -= 1    # count = count - 1
count *= 2    # count = count * 2
count /= 4    # count = count / 4
```

Très utilisé dans les loops et counters.

Fichier : `fstrings_demo.py`

---

## Working with Strings

String = séquence de caractères entre quotes (`'`, `"`, ou `"""` pour multi-ligne). Tous traités comme même type.

### String Length

`len()` retourne le nombre de caractères.

```python
password = "Tr0ub4dor"
length = len(password)
print(f"Password length: {length}")   # 9
```

### String Indexing and Slicing

Index commence à 0.

|Index|0|1|2|3|4|5|
|---|---|---|---|---|---|---|
|Character|P|y|t|h|o|n|

```python
word = "Python"
print(word[0])     # P
print(word[5])     # n
print(word[-1])    # n  (index négatif depuis la fin)
```

Slicing : `string[start:end]`, start inclus, end exclu.

```python
word = "Python"
print(word[0:3])   # Pyt
print(word[2:])    # thon
print(word[:4])    # Pyth
```

### Useful String Methods

|Method|What It Does|Example|Result|
|---|---|---|---|
|.upper()|Convertit en majuscules|"hello".upper()|"HELLO"|
|.lower()|Convertit en minuscules|"HELLO".lower()|"hello"|
|.strip()|Retire espaces début/fin|" hi ".strip()|"hi"|
|.replace(a, b)|Remplace a par b|"cat".replace("c","b")|"bat"|
|.split(sep)|Découpe en liste selon sep|"a,b,c".split(",")|["a","b","c"]|
|.startswith(x)|Vérifie début|"http://".startswith("http")|True|
|.endswith(x)|Vérifie fin|"file.txt".endswith(".txt")|True|
|.count(x)|Compte occurrences|"banana".count("a")|3|

### Character Checks

Utile pour validation de password. Retournent un boolean.

```python
char = "A"
print(char.isupper())    # True
print(char.islower())    # False
print(char.isdigit())    # False
print(char.isalpha())    # True   (lettre ?)
print(char.isalnum())    # True   (lettre ou chiffre ?)
```

### The in Operator

Vérifie si une sous-chaîne existe dans une chaîne. Retourne True/False.

```python
url = "https://tryhackme.com/room/pythoncoreconcepts"
print("tryhackme" in url)     # True
print("hackthebox" in url)    # False
print("https" in url)         # True
```

Fonctionne aussi avec listes et dictionnaires.

Fichier : `strings_demo.py`

---

## Lists and Dictionaries

### Lists

Collection ordonnée entre crochets. Items de n'importe quel type.

```python
ports = [22, 80, 443, 8080]
usernames = ["admin", "root", "guest"]
mixed = ["server1", 443, True]
```

### Accessing and Modifying List Elements

Même indexation que les strings.

```python
ports = [22, 80, 443, 8080]
print(ports[0])      # 22
print(ports[-1])     # 8080
print(ports[1:3])    # [80, 443]

ports[0] = 2222
print(ports)          # [2222, 80, 443, 8080]
```

### Common List Methods

|Method|What It Does|Example|
|---|---|---|
|.append(x)|Ajoute x à la fin|ports.append(3306)|
|.remove(x)|Retire la 1ère occurrence de x|ports.remove(80)|
|.pop(i)|Retire et retourne l'item à l'index i|ports.pop(0)|
|.sort()|Trie en ordre croissant|ports.sort()|
|.reverse()|Inverse l'ordre|ports.reverse()|
|len(list)|Nombre d'items|len(ports)|

`in` fonctionne aussi sur les listes :

```python
common_passwords = ["123456", "password", "admin", "letmein"]
if "password" in common_passwords:
    print("This password is in the common list.")
```

### Dictionaries

Stocke des paires clé-valeur entre accolades.

```python
services = {
    22: "SSH",
    80: "HTTP",
    443: "HTTPS",
    3306: "MySQL"
}
```

### Accessing Dictionary Values

```python
print(services[22])     # SSH
print(services[443])    # HTTPS
```

### Adding, Updating, and Removing Entries

```python
services[8080] = "HTTP-Alt"   # ajout
services[22] = "OpenSSH"      # update
del services[3306]            # suppression
print(services)
# {22: 'OpenSSH', 80: 'HTTP', 443: 'HTTPS', 8080: 'HTTP-Alt'}
```

### Checking for Keys

```python
if 22 in services:
    print(f"Port 22 runs {services[22]}")
```

### Useful Dictionary Methods

|Method|Returns|
|---|---|
|.keys()|Toutes les clés|
|.values()|Toutes les valeurs|
|.items()|Paires clé-valeur (tuples)|
|.get(key, default)|Valeur de key, ou default si absente|

`.get()` évite l'erreur d'une clé inexistante (vs crochets qui plantent) :

```python
result = services.get(9999, "Unknown")
print(result)   # Unknown
```

### Looking Ahead

Dans [Python: Building Scripts](https://tryhackme.com/room/pythonbuildingscripts) : Password Strength Checker utilisant liste de passwords faibles + dictionnaire pour labels de strength.

Fichier : `lists_dicts_demo.py`

---

## Arithmetic and Membership Operators

### New Arithmetic Operators

|Operator|Name|Example|Result|
|---|---|---|---|
|**|Exponent|2 ** 8|256|
|//|Floor Division|7 // 2|3|
|%|Modulus (remainder)|7 % 2|1|

`%` : reste de la division. Utilisé pour pair/impair : `if number % 2 == 0` -> pair.

`//` : division puis arrondi vers le bas. `/` retourne toujours un float (7/2 = 3.5), `//` retourne 3.

`**` : puissance. Ex sécurité : clé 128-bit = 2 ** 128 combinaisons possibles.

### The Membership Operator: in

Fonctionne avec presque tous les types de collections.

```python
common = ["123456", "password", "qwerty", "letmein"]
user_password = "qwerty"
if user_password in common:
    print("This password is too common.")
```

Négation avec `not in` :

```python
if user_password not in common:
    print("Good. This password is not in the common list.")
```

### Combining Operators in Practice

```python
password = "Tr0ubador"
length = len(password)
has_digit = any(char.isdigit() for char in password)
if length >= 8 and has_digit:
    print("Moderate strength")
elif length >= 8 or has_digit:
    print("Weak, but has some merit")
else:
    print("Very weak")
```

`and` requiert les deux conditions vraies. `or` requiert au moins une.

Fichier : `operators_demo.py`

---

## Loops: for and while

### Quick Recap: while Loops

Tourne tant que la condition est True.

```python
attempts = 0
max_attempts = 3

while attempts < max_attempts:
    password = input("Enter password: ")
    attempts += 1
    print(f"Attempt {attempts} of {max_attempts}")
```

### The for Loop

Pour itérer sur une séquence connue.

```python
targets = ["192.168.1.1", "192.168.1.2", "192.168.1.3"]

for ip in targets:
    print(f"Scanning {ip}...")
```

`ip` prend la valeur de chaque élément, un à la fois.

### Iterating Over Strings

String = séquence -> itérable caractère par caractère.

```python
password = "S3cure!"

for char in password:
    if char.isdigit():
        print(f"Found digit: {char}")
    elif char.isupper():
        print(f"Found uppercase: {char}")
```

### The range() Function

Génère une séquence de nombres sans liste prédéfinie.

```python
# range(stop): 0 à stop-1
for i in range(5):
    print(i)       # 0,1,2,3,4

# range(start, stop): start à stop-1
for i in range(1, 6):
    print(i)       # 1,2,3,4,5

# range(start, stop, step): incrément custom
for i in range(0, 20, 5):
    print(i)       # 0,5,10,15
```

`range(5)` produit 5 nombres : 0 à 4.

### Iterating Over Dictionaries

```python
services = {22: "SSH", 80: "HTTP", 443: "HTTPS"}

for port, name in services.items():
    print(f"Port {port} = {name}")
```

### break and continue

`break` : sort immédiatement de la boucle. `continue` : passe à l'itération suivante.

```python
for port in [22, 80, 443, 8080]:
    if port == 443:
        print(f"Port {port} found. Stopping scan.")
        break
    print(f"Checked port {port}")
# 8080 jamais checké (break a stoppé avant)
```

```python
lines = ["admin", "", "root", "", "guest"]

for line in lines:
    if line == "":
        continue       # skip lignes vides
    print(f"Processing: {line}")
```

### Choosing Between for and while

`for` : nombre d'itérations connu à l'avance (liste, range, string). `while` : nombre d'itérations dépend d'une condition qui change à l'exécution.

Fichier : `loops_demo.py`

---

