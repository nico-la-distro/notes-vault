# Raccourcis terminal Linux (Bash / Zsh)

> [!info] Mode par défaut Ces raccourcis s'appliquent au mode **Emacs** (activé par défaut dans Bash et Zsh). Pour basculer en mode Vi : `set -o vi`

---

## 🖱️ Déplacement du curseur

| Raccourci      | Action                                         |
| -------------- | ---------------------------------------------- |
| `Ctrl + A`     | Aller au **début** de la ligne                 |
| `Ctrl + E`     | Aller à la **fin** de la ligne                 |
| `Ctrl + F`     | Avancer d'un **caractère**                     |
| `Ctrl + B`     | Reculer d'un **caractère**                     |
| `Alt + F`      | Avancer d'un **mot**                           |
| `Alt + B`      | Reculer d'un **mot**                           |
| `Ctrl + X` `X` | Basculer entre le curseur et le début de ligne |

---

## 🗑️ Suppression

|Raccourci|Action|
|---|---|
|`Ctrl + D`|Supprimer le caractère **sous** le curseur (ou fermer le shell si ligne vide)|
|`Backspace`|Supprimer le caractère **avant** le curseur|
|`Ctrl + H`|Idem `Backspace`|
|`Ctrl + W`|Supprimer le **mot avant** le curseur|
|`Alt + D`|Supprimer le **mot après** le curseur|
|`Alt + Backspace`|Supprimer le mot avant le curseur (variante)|
|`Ctrl + K`|Supprimer du curseur jusqu'à la **fin de ligne** → kill-ring|
|`Ctrl + U`|Supprimer du curseur jusqu'au **début de ligne** → kill-ring|
|`Ctrl + X` `Backspace`|Supprimer du curseur jusqu'au début de ligne|

---

## 📋 Kill-ring (copier / coller)

> [!tip] Kill-ring Le kill-ring est un presse-papier interne à Bash. `Ctrl+K` et `Ctrl+U` y envoient le texte supprimé ; `Ctrl+Y` le recolle.

|Raccourci|Action|
|---|---|
|`Ctrl + Y`|**Coller** (yank) le dernier contenu du kill-ring|
|`Alt + Y`|Après `Ctrl+Y`, **cycler** vers les entrées précédentes du kill-ring|
|`Ctrl + Space`|Poser le **point de début de sélection** (mark)|
|`Alt + W`|Copier la région sélectionnée dans le kill-ring **sans la supprimer**|

---

## ✏️ Modifier & transposer

|Raccourci|Action|
|---|---|
|`Ctrl + T`|Transposer les deux **caractères** autour du curseur|
|`Alt + T`|Transposer les deux **mots** autour du curseur|
|`Alt + U`|Passer le mot courant en **MAJUSCULES**|
|`Alt + L`|Passer le mot courant en **minuscules**|
|`Alt + C`|Mettre en **majuscule** la première lettre du mot courant|
|`Alt + .`|Insérer le **dernier argument** de la commande précédente|
|`Alt + _`|Idem `Alt+.` (variante)|

---

## 🕓 Historique

|Raccourci|Action|
|---|---|
|`↑` / `Ctrl + P`|Commande **précédente** dans l'historique|
|`↓` / `Ctrl + N`|Commande **suivante** dans l'historique|
|`Ctrl + R`|**Recherche incrémentale** dans l'historique (arrière)|
|`Ctrl + S`|Recherche incrémentale dans l'historique (avant) — nécessite `stty -ixon`|
|`Ctrl + G`|**Quitter** la recherche dans l'historique sans exécuter|
|`Alt + <`|Aller à la **première** entrée de l'historique|
|`Alt + >`|Aller à la **dernière** entrée (ligne courante)|
|`Alt + P`|Rechercher vers l'arrière (non incrémental)|
|`Alt + N`|Rechercher vers l'avant (non incrémental)|

### Expansions bang (`!`)

|Syntaxe|Action|
|---|---|
|`!!`|Répéter la **dernière commande**|
|`!n`|Répéter la commande numéro **n**|
|`!str`|Répéter la dernière commande commençant par **str**|
|`!$`|**Dernier argument** de la commande précédente|
|`!*`|**Tous les arguments** de la commande précédente|

---

## ⏹️ Contrôle des processus

|Raccourci|Signal|Action|
|---|---|---|
|`Ctrl + C`|`SIGINT`|**Interrompre** le processus en cours|
|`Ctrl + Z`|`SIGTSTP`|**Suspendre** le processus (reprendre avec `fg` ou `bg`)|
|`Ctrl + D`|`EOF`|Fermer le shell si la ligne est vide|
|`Ctrl + \`|`SIGQUIT`|Quitter avec core dump|

---

## 🖥️ Contrôle de l'écran / terminal

|Raccourci|Action|
|---|---|
|`Ctrl + L`|**Effacer l'écran** (équivalent à `clear`)|
|`Ctrl + S`|**Suspendre** l'affichage (XOFF — freeze le flux)|
|`Ctrl + Q`|**Reprendre** l'affichage (XON — unfreeze)|
|`Shift + PgUp`|Faire défiler vers le **haut**|
|`Shift + PgDn`|Faire défiler vers le **bas**|

---

## 🔁 Complétion automatique

|Raccourci|Action|
|---|---|
|`Tab`|Compléter la commande, le fichier ou le répertoire courant|
|`Tab` `Tab`|Afficher **toutes les complétions** possibles|
|`Alt + ?`|Lister toutes les complétions (idem double `Tab`)|
|`Alt + *`|**Insérer** toutes les complétions dans la ligne|
|`Alt + /`|Tenter une complétion de nom de **fichier**|
|`Ctrl + X` `Tab`|Insérer un vrai caractère **tabulation** dans la ligne|

---

## 🔧 Édition avancée & divers

|Raccourci|Action|
|---|---|
|`Ctrl + J` / `Ctrl + M`|Valider la ligne (équivalent `Entrée`)|
|`Ctrl + O`|Valider et récupérer la commande **suivante** dans l'historique|
|`Alt + Enter`|Insérer un **saut de ligne** sans exécuter|
|`Ctrl + X` `Ctrl + E`|Ouvrir la commande en cours dans **`$EDITOR`**|
|`Ctrl + X` `Ctrl + R`|Relire le fichier `inputrc` et appliquer les liaisons|
|`Ctrl + _` / `Ctrl + X` `Ctrl + U`|**Annuler** la dernière modification (undo)|
|`Alt + R`|**Revenir** à l'état original de la ligne courante (revert)|
|`Alt + #`|**Commenter** la ligne et la mettre dans l'historique|
|`Alt + 0–9`|Passer un **argument numérique** à la prochaine action|
|`Alt + -`|Passer un argument numérique **négatif**|

---

## 🔗 Voir aussi

- `man readline` — documentation complète des liaisons clavier
- `bind -P` — lister toutes les liaisons actives dans Bash