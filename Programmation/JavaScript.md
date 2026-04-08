## **key concepts**

| Concept             | Syntaxe JS                    | À savoir                |
| ------------------- | ----------------------------- | ----------------------- |
| Variable            | `let x = 5;`                  | Valeur modifiable       |
| Constante           | `const x = 5;`                | Valeur non modifiable   |
| Afficher            | `console.log("Hello");`       | Équivalent de `print()` |
| Condition           | `if (x > 5) {}`               | Toujours avec `{}`      |
| Sinon               | `else {}`                     | Bloc alternatif         |
| Boucle for          | `for (let i=0; i<5; i++) {}`  | Compte de 0 à 4         |
| Boucle for (propre) | `for (const x of arr) {}`     | Comme Python            |
| Fonction            | `function f(a) { return a; }` | Code réutilisable       |
| Tableau             | `const arr = [1,2,3];`        | Index commence à 0      |
| Longueur            | `arr.length`                  | Taille du tableau       |
| Aléatoire           | `Math.random()`               | Entre 0 et 1            |
| Arrondi bas         | `Math.floor(x)`               | Supprime décimales      |
| Entrée (Node)       | `require('readline')`         | Pour input utilisateur  |

## **dialogue boxes**

`alert()`
- Affiche un message
- Bouton **OK** uniquement

`alert("Hello THM");`

➡️ Informer / avertir

--- 

`prompt()`
- Demande une **entrée utilisateur**
- Retourne la valeur ou `null`

`name = prompt("What is your name?"); alert("Hello " + name);`

➡️ Récupérer une info

---

`confirm()`
- Demande une **confirmation**
- Retourne `true` ou `false`

`confirm("Do you want to proceed?");`

➡️ Valider une action

## **control flow**

Le **control flow** définit **dans quel ordre le code s’exécute** selon des conditions.

JS utilise :
- **conditions** → `if / else`, `switch`
- **boucles** → `for`, `while`, `do...while`

Objectif :  
👉 faire réagir le programme selon les situations.

Exemple :

age = prompt("What is your age");

if (age >= 18) {
  document.getElementById("message").innerHTML = "You are an adult.";
} else {
  document.getElementById("message").innerHTML = "You are a minor.";
}

**Point important (sécurité)**
- Le **contrôle se fait côté client**
- L’utilisateur peut **modifier ou contourner le JS**
- Donc **JS seul ne sécurise rien**

**Bypass d’un formulaire de login**
- Si l’authentification est faite **uniquement en JS**
- Un attaquant peut :
    - modifier le code
    - forcer la condition à `true`
    - se connecter sans identifiants valides
👉 **Les vérifications critiques doivent être faites côté serveur**


## **minification**

- Supprime : espaces, retours à la ligne, commentaires
- Raccourcit parfois les noms de variables
- Objectif : **réduire la taille** et **accélérer le chargement**
- Le code fonctionne **exactement pareil**
👉 Lisible par le navigateur, pas par l’humain.

## **obfuscation**

- Rend le code **volontairement difficile à comprendre**
- Renomme les variables (`a`, `_0x33bf`, etc.)
- Ajoute parfois du code inutile
- Objectif : **dissuader l’analyse**

## **Best practices (security)**

1️⃣ **Ne pas se fier uniquement au JS côté client**

- Le JS peut être **désactivé ou modifié**
- Toute validation importante doit aussi être faite **côté serveur**
👉 JS = confort utilisateur, **pas sécurité**

2️⃣ **Éviter les librairies non fiables**

- Toujours vérifier la **source** d’une librairie
- Des bibliothèques malveillantes peuvent imiter des vraies
👉 Une dépendance compromise = site compromis

3️⃣ **Ne jamais mettre de secrets en JS**

- API keys, tokens, mots de passe = **interdits en JS**
- Le code est **visible par tous**

`// Mauvaise pratique` 
`const apiKey = "secret";`

4️⃣ **Minifier et obfusquer le code en production**

- Réduit la taille et améliore les performances
- Rend l’analyse plus difficile pour un attaquant
- **Ce n’est pas une protection**, juste une barrière

## thm JavaScript Simple Demo (room)

https://tryhackme.com/room/javascriptsimpledemo

### Résumé (input utilisateur en Node.js)

#### Objectif

- Demander une valeur à l’utilisateur (son **guess**) dans le terminal.
- Convertir cette entrée (texte) en **nombre**.
#### Lire l’entrée

const text = await rl.question("Take a guess: ");  
guess = parseInt(text, 10);

- `rl.question(...)` renvoie **une string**
- `parseInt(..., 10)` convertit en **entier base 10**
- `await` = le programme **attend** que l’utilisateur réponde

#### Pourquoi c’est “long”

Node.js n’a pas `input()` comme Python → on utilise `readline` + `promises` pour pouvoir attendre proprement.

#### Nettoyage (important)

On ferme l’interface à la fin :

finally { rl.close(); }

`try/finally` garantit que ça se ferme même si ça plante.

---
### Version v1 (ce que fait le script)

- importe `readline`
- crée `rl`
- génère `secret` (1 à 20)
- initialise `tries` et `guess`
- affiche un message
- demande un guess
- convertit en entier
- incrémente `tries`
- ferme `rl`

✅ Il lit une tentative.  
❌ Il ne dit pas encore “trop haut/trop bas/correct” (feedback manquant).

---
### Idée clé

On ajoute des **conditions (`if / else if / else`)** pour donner un retour après **1 seul guess**.

---

### Logique de feedback

Ordre important (conditions exclusives) :

1. **Hors limites** (pas entre 1 et 20)  
    → `"out of range"`
2. **Trop petit**  
    → `"Too low"`
3. **Trop grand**  
    → `"Too high"`
4. Sinon (donc égal)  
    → `"You got it ..."`

---

### Le code de décision

if (guess < 1 || guess > 20) {  
  console.log("That number is out of range. Try again.");  
} else if (guess < secret) {  
  console.log("Too low, try again.");  
} else if (guess > secret) {  
  console.log("Too high, try again.");  
} else {  
  console.log("You got it in", tries, "tries!");  
}

- `||` = **ou**
- `else if` évite de tester les autres cas une fois qu’un cas est vrai

---

### Ce que fait `guess_v2.js`

- choisit `secret` entre 1 et 20
- demande un nombre (`rl.question`)
- convertit en entier (`parseInt`)
- `tries += 1`
- compare et affiche un message

---

### Ce qui manque

👉 Le programme ne donne qu’**une seule tentative**.  
Pour un vrai jeu, il faut une **boucle** jusqu’à trouver le bon nombre.

---

### Iterations

The script we have so far efficiently provides feedback on the user’s guess; however, it does not give them a second chance. In this task, we will make the necessary changes so that it keeps prompting the user for new guesses until they figure it out.

One way to achieve this is to keep prompting the user to make new guesses as long as their guess is wrong, i.e., `while` their `guess` is **not equal** to `secret`. That’s quite easy to express in JavaScript: `while (guess !== secret)`. The `!==` means “not equal”. This is called a `while` loop and is written as shown below:

```javascript
// Repeat until the user guesses the secret number.
while (guess !== secret) {
    // Loop body: instructions to be repeated
}
```

The next part to decide is what to include in the body of this `while` loop. Based on the program logic we have built so far, we should repeat the following:

- Prompt the user to take a guess
- Convert the user’s input to a number and save it in `guess`
- Increase the number of `tries` by one
- Check `guess` with respect to the lower and upper limits
- If it is within the limits, compare it to the `secret` number
- Display feedback to the user about their choice

And we will repeat if the users didn’t make the correct guess.

The JavaScript code with the `while` loop filled is shown below.

```javascript
// Repeat until the user guesses the secret number.
while (guess !== secret) {
    const text = await rl.question("Take a guess: "); // rl.question() returns text (a string)
    guess = parseInt(text, 10); // convert the text to a number

    tries = tries + 1; // add 1 try

    // Give a hint using if / else if / else.
    if (guess < 1 || guess > 20) {
        console.log("That number is out of range. Try again.");
    } else if (guess < secret) {
        console.log("Too low, try again.");
    } else if (guess > secret) {
        console.log("Too high, try again.");
    } else {
        console.log("You got it in", tries, "tries!");
    }
}
```

#### Putting it All Together

The program we have constructed so far is listed below and is also available on the system as `guess_v3.js`.

```javascript
import * as readline from "node:readline/promises";
import { stdin as input, stdout as output } from "node:process";

const rl = readline.createInterface({ input, output });

try {
    const secret =
        Math.floor(Math.random() * (20)) + 1; // 1 <= secret <= 20
    let tries = 0;
    let guess = 0; // start with a value that cannot be the secret (since secret is 1..20)

    console.log("I'm thinking of a number between 1 and 20");

    // Repeat until the user guesses the secret number.
    while (guess !== secret) {
        const text = await rl.question("Take a guess: "); // rl.question() returns text (a string)
        guess = parseInt(text, 10); // convert the text to a number

        tries = tries + 1; // add 1 try

        // Give a hint using if / else if / else.
        if (guess < 1 || guess > 20) {
            console.log("That number is out of range. Try again.");
        } else if (guess < secret) {
            console.log("Too low, try again.");
        } else if (guess > secret) {
            console.log("Too high, try again.");
        } else {
            console.log("You got it in", tries, "tries!");
        }
    }
} finally {
    rl.close();
}
```

You can test it on the system by running `node guess_v3.js`; it can be found in `/home/ubuntu/JavaScript-Demo`. Every time you rerun the program, it should pick a new secret number for you to guess.

Terminal

```shell-session
ubuntu@tryhackme:~/JavaScript-Demo$ node guess_v3.js
I'm thinking of a number between 1 and 20
Take a guess: 10
Too low, try again.
Take a guess: 15
Too high, try again.
Take a guess: 13
Too low, try again.
Take a guess: 14
You got it in 4 tries!
```

It should be noted that the file `node guess_v4.js`, on the VM and in the `zip` file attached to Task 3, further improves this program. You are encouraged to take a look; however, it is not critical for this introductory room.