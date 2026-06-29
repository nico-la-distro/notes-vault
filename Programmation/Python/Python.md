## **Operateurs mathématique**

|Opérateur|Signification|Exemple|
|--:|---|---|
|`+`|Addition|`1 + 1 = 2`|
|`-`|Soustraction|`5 - 1 = 4`|
|`*`|Multiplication|`10 * 10 = 100`|
|`/`|Division|`10 / 2 = 5`|
|`%`|Modulo (reste)|`10 % 2 = 0`|
|`**`|Exponentiation|`5**2 = 25`|

**comparaison**

|Opérateur|Signification|Exemple|
|--:|---|---|
|`>`|Supérieur à|`5 > 3 → True`|
|`<`|Inférieur à|`2 < 1 → False`|
|`==`|Égal à|`4 == 4 → True`|
|`!=`|Différent de|`4 != 5 → True`|
|`>=`|Supérieur ou égal à|`5 >= 5 → True`|
|`<=`|Inférieur ou égal à|`3 <= 2 → False`|

## **Variables**

Une **variable** = un **nom** auquel on associe une **valeur** (donnée).

Exemples :
- `food = "ice cream"` → variable `food` contient une **string** (texte)
- `money = 2000` → variable `money` contient un **nombre** (int)

**maj une variable**

On peut **modifier** la valeur d’une variable pendant le programme.
- Exemple :
    - `age = 30`
    - `age = age + 1` → on reprend la valeur actuelle de `age` et on ajoute 1
    - `print(age)` → affiche `31`

**data type**

- **String (`str`)** : texte / caractères (`"hello"`, `"ice cream"`)
- **Integer (`int`)** : nombres entiers (`2000`, `-3`)
- **Float (`float`)** : décimaux / fractions (`3.14`, `10.0`)
- **Boolean (`bool`)** : `True` ou `False`
- **List (`list`)** : collection (peut contenir plusieurs types) (`[1, "a", True]`)


## **Logical & Boolean Operators**

| Opérateur | Signification     | Exemple      |
| --------: | ----------------- | ------------ |
|      `==` | égal à            | `if x == 5:` |
|       `<` | inférieur à       | `if x < 5:`  |
|      `<=` | inférieur ou égal | `if x <= 5:` |
|       `>` | supérieur à       | `if x > 5:`  |
|      `>=` | supérieur ou égal | `if x >= 5:` |

---

|Opérateur|Vrai quand…|Exemple|
|--:|---|---|
|`and`|**toutes** les conditions sont vraies|`if x >= 5 and x <= 100:`|
|`or`|**au moins une** condition est vraie|`if x == 1 or x == 10:`|
|`not`|on **inverse** une condition/valeur|`if not hungry:`|

---

**Le reste à retenir (hors tableau)**
- Ces tests sont utilisés dans les **`if / elif / else`** et retournent **True/False**.
- En Python on écrit **`and / or / not`** (pas `AND / OR / NOT`).
- `elif` = “sinon si…”, `else` = cas par défaut si aucune condition n’est vraie.

## **Loops**

- Une **boucle** répète un bloc de code plusieurs fois.
- En Python : **`while`** (tant que…) et **`for`** (pour chaque élément / sur une plage).

**while**

|Élément|Rôle|
|---|---|
|condition du `while`|la boucle continue tant que c’est **True** (`i <= 10`)|
|mise à jour (ex: `i = i + 1`)|évite la boucle infinie, fait évoluer la condition|
|arrêt|quand la condition devient **False** (ici quand `i > 10`)|

**exemple**

```python
i = 1
while i <= 10:
    print(i)
    i = i + 1
```
---

**for**

|Élément|Rôle|
|---|---|
|liste (ex: `websites = [...]`)|collection d’éléments|
|`for site in websites`|prend chaque élément un par un|
|arrêt|quand tous les éléments ont été parcourus|

**exemple**

```python
websites = ["facebook.com", "google.com", "amazon.com"]
for site in websites:
    print(site)
```

**range()**

|Code|Ce que ça produit|
|---|---|
|`range(5)`|`0, 1, 2, 3, 4` (5 valeurs, départ à 0)|
````python
for i in range(5):
    print(i)
````

---

**à retenir**

- `while` : pense toujours à **modifier** la variable de boucle (`i += 1`) sinon boucle infinie.
- `for` : idéal pour parcourir une **liste** ou une **séquence** (plus simple/plus safe que `while` dans ce cas).

## **Functions**

- Une **fonction** = bloc de code **réutilisable** qu’on peut appeler plusieurs fois.
- Permet d’éviter le **code répétitif**, de rendre le programme plus clair et plus facile à maintenir.

**structure**

|Élément|Rôle|
|---|---|
|`def`|démarre la définition de la fonction|
|nom de fonction|identifiant pour l’appeler (ex: `sayHello`)|
|paramètres `(...)`|entrées (données) passées à la fonction (ex: `name`)|
|`:`|fin de l’en-tête, début du bloc|
|indentation|tout ce qui est indenté après `:` appartient à la fonction|
|`return` (optionnel)|renvoie une valeur au code appelant|

**exemple** (sans return)

```python
def sayHello(name):
    print("Hello " + name + "! Nice to meet you.")

sayHello("ben")
```
---

`return` : récupérer un résultat

- Une fonction peut **renvoyer** une valeur (ex: un `float`) via `return`.
- La valeur renvoyée peut ensuite être utilisée dans un calcul, stockée dans une variable, etc.

**exemple**


```python
def calcCost(item):
    if item == "sweets":
        return 3.99
    elif item == "oranges":
        return 1.99
    else:
        return 0.99

spent = 10
spent = spent + calcCost("sweets")  # +3.99
print("You have spent:" + str(spent))  # 13.99
```

---

**à retenir**

- Pour concaténer un nombre avec du texte, il faut convertir en string : `str(spent)`.
- `return` **arrête** la fonction dès qu’il est exécuté.

## **Files**

|Action|Mode `open()`|Méthodes utiles|Remarques|
|---|---|---|---|
|Lire un fichier|`"r"`|`read()` (tout), `readlines()` (liste de lignes)|Si le fichier n’est pas dans le même dossier, donner le **chemin complet**|
|Ajouter à un fichier existant|`"a"`|`write()`|**Append** = ajoute à la fin, ne supprime pas le contenu|
|Créer / écraser et écrire|`"w"`|`write()`|**Write** = crée si absent, **écrase** si présent|

- `readlines()` est pratique si tu veux **boucler ligne par ligne** (listes de sites, wordlists, etc.).
- `close()` ferme le fichier (important pour “finaliser” l’écriture).

Exemples  :

```python
f = open("file_name", "r")
print(f.read())
```

```python
f = open("demofile1.txt", "a") # Append to an existing file
f.write("The file will include more text..")
f.close()

f = open("demofile2.txt", "w") # Creating and writing to a new file
f.write("demofile2 file created, with this content in!")
f.close()
```


## **Import**

Une **librairie** = un ensemble de fonctions déjà écrites que tu peux réutiliser.

Syntaxe :
- `import nom_lib`
- Puis tu appelles avec `nom_lib.fonction()` (ou `nom_lib.objet.methode()`)

**exemple**

```python
import datetime
current_time = datetime.datetime.now()
print(current_time)
```

**pentest libraries**

| Librairie  | À quoi ça sert                            |
| ---------- | ----------------------------------------- |
| `requests` | HTTP simple (requêtes web)                |
| `scapy`    | créer/sniffer/découper des paquets réseau |
| `pwntools` | CTF + exploit dev                         |

**install librairie non incluse**

```python
pip install scapy
```



