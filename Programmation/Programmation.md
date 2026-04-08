## **Conseils de pro**
=> installer les packages avec pip dans un venv -> pro, propre, pratique, ne casse pas l'OS
## **ressources**
https://www.wolframalpha.com/ math and shits
https://www.apprendslinfo.fr/ le goat de l'informatique
https://docs.python.org/fr/3/ doc python

**pentest python libraries**

| Librairie  | À quoi ça sert                            |
| ---------- | ----------------------------------------- |
| `requests` | HTTP simple (requêtes web)                |
| `scapy`    | créer/sniffer/découper des paquets réseau |
| `pwntools` | CTF + exploit dev                         |

## **lambda**

- **Fonction anonyme** en une seule ligne :

`lambda arguments: expression

- **Exemples simples** :
add = lambda a, b: a + b
print(add(2, 3))  # Affiche 5

liste = [1, 2, 3]
result = list(map(lambda x: x*2, liste))
print(result)  # Affiche [2, 4, 6]

liste = [1, 2, 3, 4]
pairs = list(filter(lambda x: x % 2 == 0, liste))
print(pairs)  # Affiche [2, 4]


- **Usage** : petites fonctions temporaires, `map`, `filter`, `sorted`
- **Limites** : une seule expression, lisibilité parfois réduite

## **README**

Les fichiers README incluent généralement des informations sur :

Ce que le projet fait
Pourquoi le projet est utile
Prise en main du projet par les utilisateurs
Où les utilisateurs peuvent obtenir de l’aide sur votre projet
Qui maintient et contribue au projet


## **scope (python)**

![[scope prog.png]]

## **interprété vs compilé**

**Langage compilé** :  
Le code est **entièrement traduit en langage machine avant l’exécution**.

**Langage interprété** :  
Le code est **exécuté directement par un interpréteur, instruction par instruction**, sans compilation préalable visible.

|Langage|Type|Description|
|---|---|---|
|C|Compilé|Compilation directe en code machine|
|C++|Compilé|Performances élevées|
|Rust|Compilé|Sécurité mémoire|
|Go|Compilé|Compilation rapide|
|Java|Hybride|Compilé en bytecode, exécuté par la JVM|
|Python|Interprété|Bytecode + interprétation|
|JavaScript|Interprété|Exécuté par le navigateur|
|PHP|Interprété|Côté serveur|
|Ruby|Interprété|Orienté simplicité|
|Bash|Interprété|Scripts système|

## **obfuscator / deobfuscator**

https://codebeautify.org/javascript-obfuscator

https://obf-io.deobfuscate.io/